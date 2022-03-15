# Oak Notebook 🧪

**Oak Notebook** is an experimental tool for creating [dynamic documents](https://thesephist.com/posts/notation/#dynamic-notation) with Markdown and [Oak](https://oaklang.org/). It's both a way of writing documents with interactive, programmable "panels" for explaining and exploring complex ideas, and a "compiler" script that transforms such Markdown documents into HTML web pages. It's a bit like [MDX](https://mdxjs.com/), if MDX was focused specifically on input widgets and interactive exploration of information.

![A demo of Oak Notebook, experimenting with input widgets](/img/oak-notebook-demo.gif)

I was inspired initially by [Streamlit](https://docs.streamlit.io/library/api-reference) and [Bret Victor's Scrubbing Calculator](http://worrydream.com/ScrubbingCalculator/) to explore ideas in this space, but what's here isn't a specific re-implementation of either of those concepts, and takes further inspirations from other products, experiments, and prior art.

Oak Notebook provides a way to embed these interactive panels into documents written in Markdown without worrying about styling user interfaces or managing rich user input components.

## Guiding principles

These are some values or properties that I think a tool like Oak Notebook should probably embody, but it's neither a complete nor a final list -- simply what I've been using navigate my own thinking and prototyping. Oak Notebook today doesn't really embody all of these ideas to the extent that I'd like to, but it's just a start.

**Words first.** Dynamic documents are still fundamentally documents, and in the spirit of [literate programming](https://en.wikipedia.org/wiki/Literate_programming), I think language is the most flexible and versatile tool we have to communicate ideas. Other dynamic, interactive components should augment prose rather than dominate them on the page.

**Direct representation, direct manipulation.** When designing visualizations, we should favor representing quantities and concepts directly using shapes, graphs, animated motion, or whatever else is best suited for the idea that needs to be communicated. Likewise, when building input components, we should establish as direct a connection as possible between the user's manipulation of the controls and the data being fed into the simulation or visualization -- motion in time for time-series input, a rotating knob for angle input, a slider for size or magnitude, and so on.

**Embrace programming and programmability.** What makes programming challenging is not the notation or syntax, but the toolchain and mental models that are often required to build software. I believe that with simple build steps and a well-designed API, the ability to express dynamic ideas with the full power of a general-purpose programming language can be a gift.

**Composability.** Oak Notebook should come with a versatile set of inputs and widgets out of the box, but it should be easy and idiomatic to compose or combine these primitive building blocks together to make larger input components or reusable widgets that fit a given problem, like a color input with R/G/B controls or a reusable label for displaying dates.

## How it works

>⚠️ **NOTE**: None of the APIs here are really final by any definition of that word, and more importantly, my current implementation of these widgets are still pretty fragile. For example, if you get the type of parameters wrong, things will probably blow up without a clear error message.
>
>I'm ... sorry. This is currently more of a proof-of-concept than a thing for people to pick up and use. _¯\\\_(ツ)\_/¯_

Oak Notebook documents are a superset of Markdown documents. For example, you can see the Markdown document for this exact Oak Notebook page [here on GitHub](https://github.com/thesephist/x-oak-notebook/blob/main/demo.md). What makes Oak Notebook documents special are the code blocks tagged "notebook", with `\`\`\`notebook ... \`\`\``. When the Oak Notebook compiler sees any code blocks tagged this way, it transforms them into interactive panels like the ones you see on this page. For example, a very minimal Oak Notebook document may be

```md
# Hello, world!

Here's a Notebook panel:

`​``notebook
nb.label('Some sample text')
`​``
```

All interactive Oak Notebook panels have two parts: the inputs and the display function. The inputs come first, and describe what variables control the panel's output. The display function runs every time one of the inputs changes, and describes what the panel should display as output. Here's a sample showing some of what Oak Notebook's widgets can do:

In this example, we specify a single text input using `nb.text()` with the label `'Your name'` and default value `'Linus'`. Every time the input changes, we compute some information about the name, and display the data in both text and table formats, or show a message if the name is empty.

```js
// input widgets
name := nb.text('Your name', 'Linus')

// display function -- run this every time the input changes
fn display {
    trimmedName := name.value |> str.trim()
    uniqLetters := trimmedName |>
        std.filter(fn(c) !str.space?(c)) |>
        str.upper() |>
        sort.sort() |>
        std.uniq()

    // output widgets
    if trimmedName {
        '' -> nb.label('_Type in your name to see something cool!_')
        _ -> {
            nb.label(
                fmt.format(
                    'Your name is _{{0}}_. It contains **{{1}}** unique letters: {{2}}.'
                    trimmedName
                    len(uniqLetters)
                    uniqLetters |> str.join(', ')
                )
            )
            nb.label('Unicode codepoints in your name:')
            nb.table(
                trimmedName |>
                    str.split() |>
                    std.map(fn(c) { Character: c, Codepoint: codepoint(c) })
            )
        }
    }
}
```

Every Oak Notebook document can contain any number of interactive panels, and they should run and render independently of each other (unless the panel code does something strange like assigning properties to the global `window` object). The panels all run entirely in the browser independent of any server.

## The widget system

Oak Notebook comes with a set of basic components for common input and display widgets. They include buttons, checkboxes and selection drop-downs, various kinds of text and number inputs, labels, tables, and other display formats. For a full and detailed account of built-in widgets, check out the [demo page](https://thesephist.github.io/x-oak-notebook/).
## Limitations and future directions

There are a few things I definitely like about this implementation you've been reading about. The first is **clear, obvious affordances for inputs**. When an interactive piece of software is primarily meant to be a document, I don't think we should expect the reader to play around with every little panel and figure out which things they can drag or click or hover on to make interactive components do special things. It seems better to have clear, consistent input methods that are obvious and easy to use, even if they don't blend in as satisfyingly to specific visualizations. In providing clear, accessible, reusable inputs, I think this current direction feels right.

I also like that **you write an Oak Notebook by writing a document, not writing a program**. On platforms like [Streamlit](https://streamlit.io/), a document is a program (for example, a Python source file) which contains library function calls to render prose. Jupyter Notebooks are somewhere in between with prose interleaving program snippets. I think [MDX](https://mdxjs.com/docs/using-mdx/) gets it right here -- we should try to make writing interactive documents more like writing documents, and less like composing programs.

I think the current way that widgets can be composed together with functions is _okay_, but not amazing. It makes me feel like I'm writing software, rather than writing a document. MDX's style of supporting inline JSX may ease this problem somewhat, because JSX components compose very well notationally, but I fear that adding the full power of JSX to a document might lead to interactive panels that grow very complex, to become little mini-apps themselves.

By far the two worst things about Oak Notebook as it stands today are the [quality of notation](https://thesephist.com/posts/notation/) and the lack of an [in-place toolchain](https://www.inkandswitch.com/end-user-programming/#in-place-toolchain). The `nb.scrubbable('Label', 42, 10, 100)` style API isn't great. It's learnable, but I'd rather just write `scrubbable number from 10 to 100, default 42`, or even better, just drag-and-drop a component in as I'm writing the document. I don't like that I have to write `fn display {}` for output widgets, either. It's good that there's clear separation between input and output, but I think this, too, can be handled in a clearer way. Perhaps all this points to a DSL as the solution? But designing a DSL is itself a monumental task. On a related note, modifying any of these "panels" requires that I break out a code editor and run a compilation script, rather than make edits "in place" of the original panel. I want to extend the idea of direct manipulation all the way through to the process of writing documents, beyond simply reading them.

There are also things that I've left out so far that aren't fundamentally incompatible with Oak Notebook's current design, but absent simply because I haven't gotten to them yet. Of those ideas, here are a few that I think are worth exploring.

- **Persistent state that's local to a panel or shared between panels.** Right now, I can't take user input from one panel at the top of a document and carry that through to other panels farther down the page. This is also not possible in other "document" oriented tools like MDX, and a cause for confusing state management bugs in more capable tools like Jupyter. I also think there may be ways to address use cases that need shared state between panels without explicitly letting panels "share state" directly, perhaps using something like document-wide data bindings or events.
- **Two-way data binding between inputs.** I may want to have two input fields tied to each other so that changing one changes the other in some specific way. For example, in a demonstration about pendulum physics, I may want to first hold the pendulum period constant and change the length, then reverse the order and hold the length constant while changing the oscillation period. Right now, these require two redundant panels. (Maybe that's for the best?)
- **A more human-friendly way to plot data.** Tools like matplotlib and gnuplot are capable of great graphical wonders, but they work off of crazy DSLs and very wide API surfaces. I want plotting APIs that are capable of just accepting whatever data I give it and doing the "right thing" most of the time, with opportunities for progressive, intuitive customization later.

## Inspirations and further reading

This is not a complete compendium of prior art in this space by any means, but a growing collection of some places I've looked to for inspiration so far.

- Bret Victor's [Scrubbing Calculator](http://worrydream.com/ScrubbingCalculator/) and [Up and Down the Ladder of Abstraction](http://worrydream.com/LadderOfAbstraction/)
- [Streamlit](https://streamlit.io/) and [Jupyter Notebook](https://jupyter.org/)
- [MDX](https://mdxjs.com/), a superset of Markdown with support for embedded JavaScript expressions and dynamic React components
- [Explorable Explanations](https://explorabl.es/), a collection of interactive explanatory essays on dynamical systems
