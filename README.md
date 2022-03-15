# Oak Notebook 🧪

!html <div class="meta">
    <a href="https://oaklang.org/" target="_blank">↖ Back to Oak</a>
    <a href="https://github.com/thesephist/x-oak-notebook" target="_blank">See on GitHub →</a>
</div>

**Oak Notebook** is an experimental tool for creating [dynamic documents](https://thesephist.com/posts/notation/#dynamic-notation) with Markdown and [Oak](https://oaklang.org/). It's both a way of writing documents with interactive, programmable "panels" for explaining and exploring complex ideas, and a "compiler" script that transforms such Markdown documents into HTML web pages. It's a bit like [MDX](https://mdxjs.com/), if MDX was focused specifically on input widgets and interactive exploration of information.

I was inspired initially by [Streamlit](https://docs.streamlit.io/library/api-reference) and [Bret Victor's Scrubbing Calculator](http://worrydream.com/ScrubbingCalculator/) to explore ideas in this space, but what's here isn't a specific re-implementation of either of those concepts, and takes further inspirations from other products, experiments, and prior art.

For a demonstrative example, I want to introduce you to two interesting formulas for estimating the value of π. They're both derived from [identities about π that depend on properties of the arctangent function](https://en.wikipedia.org/wiki/Machin-like_formula).

$$ \\pi = 4 \\arctan(1) = 4 \\left( 1 - \\frac{1}{3} + \\frac{1}{5} - \\frac{1}{7} + \\frac{1}{9} - \\cdots \\right) $$

$$ \\pi = 6 \\arctan\\left(\\frac{1}{\\sqrt{3}}\\right) = \\frac{6}{\\sqrt{3}} \\left( 1 - \\frac{1}{3 \\times 3} + \\frac{1}{5 \\times 3^2} - \\frac{1}{7 \\times 3^3} + \\cdots \\right) $$

These formulas approach the true value of π at different rates, but it's difficult to get a sense of what those approximations look like, how close they are, or what some specific margin of error really means. One way to get a better grasp of these concepts is through this panel below. (Try tapping and dragging on the underlined number to adjust it.)

```notebook
max := nb.scrubbable(['Estimate π with ', ' terms.'], 3, 1, 12, 1)
units := nb.select('Units', ['Kilometers', 'Miles'])

fn estimatePi(coefficient, angle) {
    sequence := std.range(max.value) |> std.map(fn(n) {
        sign := if n % 2 { 0 -> 1, _ -> -1 }
        odd := 2 * n + 1
        sign * pow(angle, odd) / odd
    })
    [
        estimate := coefficient * math.sum(sequence...)
        error := math.abs(math.Pi - estimate)
        pctError := string(math.round(error / estimate * 100, 6)) + '%'
    ]
}

fn display {
    oneError := rootThreeError := ?

    nb.table([{
        [estimate, error, pctError] := estimatePi(4, 1)
        oneError <- math.abs(error / estimate)
        {
            Angle: '1'
            'Estimate for π': estimate
            Error: error
            '% Error': pctError
        }
    }, {
        [estimate, error, pctError] := estimatePi(6, 1 / math.sqrt(3))
        rootThreeError <- math.abs(error / estimate)
        {
            Angle: '1/√3'
            'Estimate for π': estimate
            Error: error
            '% Error': pctError
        }
    }])

    EarthCircumference := if units.value {
        'Kilometers' -> 40075.017
        _ -> 24901.461
    }
    nb.label(
        'If we measured the circumference of the Earth using these estimates of π,
        the estimate based on angle = 1 would be off by **{{0}}{{2}}**, while the estimate
        based on angle = 1/√3 would be off by only **{{1}}{{2}}**.' |> fmt.format(
            math.round(EarthCircumference * oneError, 3)
            math.round(EarthCircumference * rootThreeError, 3)
            if units.value {
                'Kilometers' -> 'km'
                _ -> ' miles'
            }
        )
    )
}
```

Although it's more of a niche use case, these panels can also interact with the webpage itself. If you're more of a "dark theme" person, you can pick your style here:

```notebook
colorTheme := nb.select('Color scheme', ['Light mode', 'Dark mode'])

fn display {
    if colorTheme.value {
        'Light mode' -> document.body.classList.remove('dark')
        _ -> {
            document.body.classList.add('dark')
            nb.label('_Dark mode is ... half-complete at the moment. Some syntax highlighting may look off. Apologies :)_')
        }
    }
}
```

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

```notebook
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

In this example, we specify a single text input using `nb.text()` with the label `'Your name'` and default value `'Linus'`. Every time the input changes, we compute some information about the name, and display the data in both text and table formats, or show a message if the name is empty.

```oak
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

Oak Notebook comes with a set of basic components for common input and display widgets. This section details those built-in widgets (or at least, what's available today).

### Input widgets

The **Button** is the simplest kind of input widget. It's a clickable button with customizable label text, and returns an object containing `clicks`, a list of timestamps of all past clicks.

```oak
nb.button(labelText) -> { clicks: float[] }
```

```oak
counter := nb.button('Count up!')

fn display {
    nb.label(
        len(counter.clicks)
        'clicks, last clicked'
        datetime.format(int(counter.clicks |> std.last() |> std.default(0))))
	nb.label(
        counter.clicks |>
            std.map(fn '🌈') |>
            str.join(' ')
    )
}
```

```notebook
counter := nb.button('Count up!')

fn display {
    nb.label(
        len(counter.clicks)
        'clicks, last clicked'
        datetime.format(int(counter.clicks |> std.last() |> std.default(0))))
	nb.label(
        counter.clicks |>
            std.map(fn '🌈') |>
            str.join(' ')
    )
}
```

The **Checkbox** can signal an on/off option or a boolean value. Like the button widget, it takes a label text as the only parameter, and makes available a boolean `checked` property.

```oak
nb.checkbox(labelText) -> { checked: bool }
```

```oak
light := nb.checkbox('Light switch')

fn display {
    nb.label(if light.checked {
        true -> '💡 Lights on!'
        _ -> '🔦 Light off...'
    })
}
```

```notebook
light := nb.checkbox('Light switch')

fn display {
    nb.label(if light.checked {
        true -> '💡 Lights on!'
        _ -> '🔦 Light off...'
    })
}
```

The **Number** widget is a conventional number input field wrapping the HTML `input[type="number"]`. It lets the author specify default, minimum, and maximum values, as well as valid numerical steps or increments. The returned object has a `value` property containing the number entered. Although the number widget is available, in most Oak Notebook panels, the Scrubbable widget below is preferred.

```oak
nb.number(labelText, defaultValue, min, max, step) -> { value: float }
```

```oak
first := nb.number('First number', 2, 0, 10, 1)
second := nb.number('Second number', 3, 0, 10, 1)

fn display {
    nb.label(first.value, '×', second.value, '=', first.value * second.value)
}
```

```notebook
first := nb.number('First number', 2, 0, 10, 1)
second := nb.number('Second number', 3, 0, 10, 1)

fn display {
    nb.label(first.value, '×', second.value, '=', first.value * second.value)
}
```

The **Scrubbable** is an unconventional way to input numbers within some specific range by dragging or swiping on a number within some label text. The best way to understand the scrubbable is to try it yourself below. It has a similar API to `nb.number`, except that the label text is a list containing two strings that go before and after the scrubbable number.

```oak
nb.scrubbable([labelBefore, labelAfter],
    defaultValue, min, max, step) -> { value: float }
```

```oak
rotation := nb.scrubbable(['Rotate me by ', 'º'], 30, 0, 720, 1)

fn display {
	nb.html(
		'<div style="
            transform: rotate({{0}}deg);
            height: 100px;
            width: 100px;
            display: flex;
            align-items: center;
            justify-content: center">HELLO</div>' |>
            fmt.format(rotation.value)
	)
}
```

```notebook
rotation := nb.scrubbable(['Rotate me by ', 'º'], 30, 0, 720, 1)

fn display {
	nb.html(
		'<div style="
            transform: rotate({{0}}deg);
            height: 100px;
            width: 100px;
            display: flex;
            align-items: center;
            justify-content: center">HELLO</div>' |>
            fmt.format(rotation.value)
	)
}
```

The **Text** widget is useful for entering short, single-line text, and wraps an HTML `input[type="text"]`. It surfaces a `value` property containing the entered text.

```oak
nb.text(labelText, defaultValue) -> { value: string }
```

```oak
input := nb.text('Tell me a sentence', 'The scar had not pained Harry for 19 years. All was well.')

fn display {
    words := input.value |>
        str.split(' ') |>
        std.filter(fn(s) s != '')
    nb.label('There are', len(words), 'words in this sentence!')
}
```

```notebook
input := nb.text('Tell me a sentence', 'The scar had not pained Harry for 19 years. All was well.')

fn display {
    words := input.value |>
        str.split(' ') |>
        std.filter(fn(s) s != '')
    nb.label('There are', len(words), 'words in this sentence!')
}
```

The **Prose** widget is like the Text widget, but for longer, multi-paragraph text. It renders a vertically resizable HTML `<textarea>`, and has the same API as `nb.text`.

```oak
nb.prose(labelText, defaultValue) -> { value: string }
```

```oak
paragraph := nb.prose('Story', 'All human beings are born free and equal in dignity and rights. They are endowed with reason and conscience and should act towards one another in a spirit of brotherhood.')

fn display {
    words := paragraph.value |>
        std.filter(fn(c) str.letter?(c) | c = ' ') |>
        str.lower() |>
        str.split(' ') |>
        std.filter(fn(w) w != '')
    uniqWords := words |>
        sort.sort() |>
        std.uniq()

    nb.label('Top 10 most frequently used words')
    nb.table(
        uniqWords |> std.map(fn(word) {
            Word: word
            Count: words |> std.filter(fn(w) w = word) |> len()
        }) |> sort.sort(:Count) |> std.reverse() |> std.take(10)
    )
}
```

```notebook
paragraph := nb.prose('Story', 'All human beings are born free and equal in dignity and rights. They are endowed with reason and conscience and should act towards one another in a spirit of brotherhood.')

fn display {
    words := paragraph.value |>
        std.filter(fn(c) str.letter?(c) | c = ' ') |>
        str.lower() |>
        str.split(' ') |>
        std.filter(fn(w) w != '')
    uniqWords := words |>
        sort.sort() |>
        std.uniq()

    nb.label('Top 10 most frequently used words')
    nb.table(
        uniqWords |> std.map(fn(word) {
            Word: word
            Count: words |> std.filter(fn(w) w = word) |> len()
        }) |> sort.sort(:Count) |> std.reverse() |> std.take(10)
    )
}
```

The **Select** widget lets the reader choose one among many options, using the HTML `<select>` element. The first given option is always selected by default.

```oak
nb.select(labelText, optionsList) -> { value: string }
```

```oak
color := nb.select('Color', [
    'Red'
    'Blue'
    'Green'
    'Yellow'
])

fn display {
    nb.html(
        fmt.format(
            '<div style="height: 100px;
                width: 100px;
                border-radius: 4px;
                background: {{ 0 }}"></div>'
            color.value |> str.lower()
        )
    )
}
```

```notebook
color := nb.select('Color', [
    'Red'
    'Blue'
    'Green'
    'Yellow'
])

fn display {
    nb.html(
        fmt.format(
            '<div style="height: 100px;
                width: 100px;
                border-radius: 4px;
                background: {{ 0 }}"></div>'
            color.value |> str.lower()
        )
    )
}
```

### Display widgets

The **Label** widget is a versatile text widget. It accepts Markdown-formatted strings and displays rich text.

```oak
nb.label(markdownFormattedText...)
```

```oak
fn display {
    nb.label('Hello, _Oak_ `Notebook`!')
}
```

```notebook
fn display {
    nb.label('Hello, _Oak_ `Notebook`!')
}
```

The **Table** widget is useful for displaying sequences or tables of data. It accepts either a list of values or a list of objects, and constructs an HTML `<table>` from it in the order in which the rows and columns are given.

```oak
nb.table(values...)
```

A list of simple values.

```oak
fn display {
    nb.table([1, 2, 3, 4, 5])
}
```

```notebook
fn display {
    nb.table([1, 2, 3, 4, 5])
}
```

A table of objects.

```oak
base := nb.scrubbable(['Powers of base ', ' ...'], 2, 1, 20, 1)
maxExp := nb.scrubbable(['... up to the exponent ', '.'], 5, 1, 25, 1)

fn display {
	nb.table(
		std.range(0, maxExp.value + 1, 1) |> with std.map() fn (exp) {
			{
				Exponent: exp
				Value: pow(base.value, exp)
			}
		}
	)
}
```

```notebook
base := nb.scrubbable(['Powers of base ', ' ...'], 2, 1, 20, 1)
maxExp := nb.scrubbable(['... up to the exponent ', '.'], 5, 1, 25, 1)

fn display {
	nb.table(
		std.range(0, maxExp.value + 1, 1) |> with std.map() fn (exp) {
			{
				Exponent: exp
				Value: pow(base.value, exp)
			}
		}
	)
}
```

The **HTML** widget is a catch-all widget that accepts and displays raw HTML content. It accepts a single HTML string.

```oak
nb.html(innerHTMLString)
```

```oak
r := nb.scrubbable(['Red: ', ''], 180, 0, 255, 1)
g := nb.scrubbable(['Green: ', ''], 190, 0, 255, 1)
b := nb.scrubbable(['Blue: ', ''], 216, 0, 255, 1)

fn display {
    nb.html(
        fmt.format(
            '<div style="height: 100px;
                width: 100px;
                border-radius: 4px;
                background: rgb({{0}},{{1}},{{2}})"></div>'
            r.value, g.value, b.value
        )
    )
}
```

```notebook
r := nb.scrubbable(['Red: ', ''], 180, 0, 255, 1)
g := nb.scrubbable(['Green: ', ''], 190, 0, 255, 1)
b := nb.scrubbable(['Blue: ', ''], 216, 0, 255, 1)

fn display {
    nb.html(
        fmt.format(
            '<div style="height: 100px;
                width: 100px;
                border-radius: 4px;
                background: rgb({{0}},{{1}},{{2}})"></div>'
            r.value, g.value, b.value
        )
    )
}
```

**Plot**

>Coming soon

**Color map**

>Coming soon

## Composing custom widgets

Because every Oak Notebook widget is just a simple function, they can be composed together into bigger custom widgets by just writing Oak functions. For example, let's say we wanted to create a custom widget for picking colors, inspired from the previous example. We may define an input widget called `colorPicker`:

```oak
with CustomWidget.define(:colorPicker) fn(nb, labelText, defaultRGB) {
    defaultRGB := defaultRGB |> std.default([180, 190, 216])

    // the colorPicker widget is made up of 1 label and 3 scrubbable widgets
    nb.label('**' + labelText + '**')
    r := nb.scrubbable(['Red: ', ''], defaultRGB.0, 0, 255, 1)
    g := nb.scrubbable(['Green: ', ''], defaultRGB.1, 0, 255, 1)
    b := nb.scrubbable(['Blue: ', ''], defaultRGB.2, 0, 255, 1)

    // the widget returns an RGB CSS color string
    {
        value: fn() 'rgb({{0}}, {{1}}, {{2}})' |>
            fmt.format(r.value, g.value, b.value)
    }
}
```

To more easily display colors, we could also define a `colorSquare` in terms of an `nb.html()`:

```oak
with CustomWidget.define(:colorSquare) fn(nb, labelText, color) {
    nb.html(
        fmt.format(
            '<div style="height: 100px;
                width: 100px;
                border-radius: 4px;
                float: left;
                margin-right: 1em;
                display: flex; align-items: center; justify-content: center;
                background: {{0}}">{{1}}</div>'
            color, labelText
        )
    )
}
```

With these custom widgets defined, we can use them just like any other built-in widget to define our panel.

```oak
favorite := nb.colorPicker('Favorite color')
lucky := nb.colorPicker('Lucky color', [200, 150, 150])

fn display {
    nb.colorSquare('Favorite', favorite.value())
    nb.colorSquare('Lucky', lucky.value())
}
```

```notebook
with CustomWidget.define(:colorPicker) fn(nb, labelText, defaultRGB) {
    defaultRGB := defaultRGB |> std.default([180, 190, 216])

    // the colorPicker widget is made up of 1 label and 3 scrubbable widgets
    nb.label('**' + labelText + '**')
    r := nb.scrubbable(['Red: ', ''], defaultRGB.0, 0, 255, 1)
    g := nb.scrubbable(['Green: ', ''], defaultRGB.1, 0, 255, 1)
    b := nb.scrubbable(['Blue: ', ''], defaultRGB.2, 0, 255, 1)

    // the widget returns an RGB CSS color string
    {
        value: fn() 'rgb({{0}}, {{1}}, {{2}})' |>
            fmt.format(r.value, g.value, b.value)
    }
}

with CustomWidget.define(:colorSquare) fn(nb, labelText, color) {
    nb.html(
        fmt.format(
            '<div style="height: 100px;
                width: 100px;
                border-radius: 4px;
                float: left;
                margin-right: 1em;
                display: flex; align-items: center; justify-content: center;
                background: {{0}}">{{1}}</div>'
            color, labelText
        )
    )
}

favorite := nb.colorPicker('Favorite color')
lucky := nb.colorPicker('Lucky color', [200, 150, 150])

fn display {
    nb.colorSquare('Favorite', favorite.value())
    nb.colorSquare('Lucky', lucky.value())
}
```

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

