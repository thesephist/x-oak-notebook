# Oak Notebook 🧪

!html <div class="meta">
    <a href="https://oaklang.org/" target="_blank">↖ Back to Oak</a>
    <a href="https://github.com/thesephist/x-oak-notebook" target="_blank">See on GitHub →</a>
</div>

**Oak Notebook** is an experimental tool for authoring [dynamic documents](https://thesephist.com/posts/notation/#dynamic-notation) with Markdown and [Oak](https://oaklang.org/). Oak Notebook is a way of writing documents with interactive, programmable "panels" for explaining and exploring complex ideas, and a "compiler" script that transforms such Markdown documents into HTML web pages. It's a bit like [MDX](https://mdxjs.com/), if MDX was focused specifically on input widgets and interactive exploration of information.

I was inspired initially by [Streamlit](https://docs.streamlit.io/library/api-reference) and [Bret Victor's Scrubbing Calculator](http://worrydream.com/ScrubbingCalculator/) to explore ideas in this space, but what's here isn't a specific re-implementation of either of those concepts, and takes further inspirations from other products, experiments, and prior art.

For a demonstrative example, I want to introduce you to two interesting formulas for estimating the value of π. They're both derived from the [approximations of π using properties of the arctangent function](https://en.wikipedia.org/wiki/Machin-like_formula).

$$ \\pi = 4 \\arctan(1) = 4 \\left( 1 - \\frac{1}{3} + \\frac{1}{5} - \\frac{1}{7} + \\frac{1}{9} - \\cdots \\right) $$

$$ \\pi = 6 \\arctan\\left(\\frac{1}{\\sqrt{3}}\\right) = \\frac{6}{\\sqrt{3}} \\left( 1 - \\frac{1}{3 \\times 3} + \\frac{1}{5 \\times 3^2} - \\frac{1}{7 \\times 3^3} + \\cdots \\right) $$

They approach the true vale of π at different rates, but it's difficult to get a sense of what those approximations look like, how close they are, or what some specific margin of error really means. One way to get a better grasp of these concepts is through this panel. (Try tapping and dragging on the underlined number to adjust it.)

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
        oneError <- error
        {
            Angle: '1'
            'Estimate for π': estimate
            Error: error
            '% Error': pctError
        }
    }, {
        [estimate, error, pctError] := estimatePi(6, 1 / math.sqrt(3))
        rootThreeError <- error
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

Oak Notebook provides a way to write these interactive panels without worrying about user interfaces or managing rich user input components.

## Guiding principles

These are some values or properties that I think a tool like Oak Notebook should probably have, but it's neither completely nor final -- simply what I've been using navigate my own thinking and prototyping. Oak Notebook today doesn't really embody all of these ideas to the extend that I'd like to, but it's just a start.

**Words first.** Dynamic documents are still fundamentally documents, and in the spirit of [literate programming](https://en.wikipedia.org/wiki/Literate_programming), I think language is the most flexible and versatile tool we have to communicate ideas. Other dynamic, interactive components should augment prose rather than dominate them on the page.

**Direct representation, direct manipulation.** When designing visualizations, we should favor representing quantities and concepts directly using shapes, graphs, animated motion, or whatever else is best suited for the idea that needs to be communicated. Likewise, when building input components, we should establish as direct a connection as possible between the user's manipulation of the controls and the data being fed into the simulation or visualization -- motion in time for time-series input, a rotating knob for angle input, a slider for size or magnitude, and so on.

**Embrace programming and programmability.** What makes programming challenging is not the notation or syntax, but the toolchain and mental models that are often required to build software. I believe that with simple build steps and a well-designed API, the ability to express dynamic ideas with the full power of a general-purpose programming language can be a gift.

**Composability.** Oak Notebook should come with a versatile set of inputs and widgets out of the box, but it should be easy and idiomatic to compose or combine these primitive building blocks together to make larger input components or reusable widgets that fit a given problem, like a color input with R/G/B controls or a reusable label for displaying dates.

## The widget system

All interactive Oak Notebook panels have two parts: the inputs and the display function. The inputs come first, and describe what variables control the panel's output. The display function runs every time one of the inputs changes, and describes what the panel should output. Here's a sample showing some of what Oak Notebook's widgets can do:

```oak
// input widgets
name := nb.text('Your name', 'Linus')

// display function
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

```notebook
// input widgets
name := nb.text('Your name', 'Linus')

// display function
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

Oak Notebook comes with a set of basic components for common input and display widgets. The rest of this section details those built-in widgets (or at least, what's available today).

### Input widgets

The **Button** is the simplest kind of input widget. It's a clickable button with customizable label text, and returns an object containing `clicks`, a list of timestamps of all clicks.

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
    with nb.label() if light.checked {
        true -> '💡 Lights on!'
        _ -> '🔦 Light off...'
    }
}
```

```notebook
light := nb.checkbox('Light switch')

fn display {
    with nb.label() if light.checked {
        true -> '💡 Lights on!'
        _ -> '🔦 Light off...'
    }
}
```

The **Number** widget is a conventional number input field wrapping the HTML `input[type="number"]`. It lets the author specify default, minimum, and maximum values, as well as valid numerical steps or increments. It makes available the `value` property containing the input number. Although the number widget is available, in most Oak Notebook panels, the Scrubbable widget below is preferred.

```oak
nb.number(labelText, defaultValue, min, max, step) -> { value: float }
```

```oak
first := nb.number('First number', 2, 0, 10, 1)
second := nb.number('Second number', 3, 0, 10, 1)

fn display {
    nb.label('{{0}} × {{1}} = {{2}}' |>
        fmt.format(first.value, second.value, first.value * second.value))
}
```

```notebook
first := nb.number('First number', 2, 0, 10, 1)
second := nb.number('Second number', 3, 0, 10, 1)

fn display {
    nb.label('{{0}} × {{1}} = {{2}}' |>
        fmt.format(first.value, second.value, first.value * second.value))
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

The **Text** widget is useful for entering short, single-line text, and wraps an HTML `input[type="text"]`. It makes available the `value` property containing the entered text.

```oak
nb.text(labelText, defaultValue) -> { value: string }
```

```oak
input := nb.text('Tell me a sentence', 'The scar had not pained Harry for 19 years. All was well.')

fn display {
    wordCount := input.value |>
        str.split(' ') |>
        std.filter(fn(s) s != '') |>
        len()
    nb.label('There are', wordCount, 'words in this sentence!')
}
```

```notebook
input := nb.text('Tell me a sentence', 'The scar had not pained Harry for 19 years. All was well.')

fn display {
    wordCount := input.value |>
        str.split(' ') |>
        std.filter(fn(s) s != '') |>
        len()
    nb.label('There are', wordCount, 'words in this sentence!')
}
```

The **Prose** widget is like the Text widget, but for longer, multi-paragraph text. It wraps a vertically resizable HTML `<textarea>`, and has the same API as `nb.text`.

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
    nb.table(
        uniqWords |> std.map(fn(word) {
            word: word
            count: words |> std.filter(fn(w) w = word) |> len()
        }) |> sort.sort(:count) |> std.reverse() |> std.take(10)
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
    nb.table(
        uniqWords |> std.map(fn(word) {
            word: word
            count: words |> std.filter(fn(w) w = word) |> len()
        }) |> sort.sort(:count) |> std.reverse() |> std.take(10)
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
            '<div style="height:100px;width:100px;border-radius:4px;background:{{ 0 }}"></div>'
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
            '<div style="height:100px;width:100px;border-radius:4px;background:{{ 0 }}"></div>'
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

A list of raw values.

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
base := nb.scrubbable(['Powers of base ', ''], 2, 1, 20, 1)
maxExp := nb.scrubbable(['... up to the exponent ', ''], 5, 1, 25, 1)

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
base := nb.scrubbable(['Powers of base ', ''], 2, 1, 20, 1)
maxExp := nb.scrubbable(['... up to the exponent ', ''], 5, 1, 25, 1)

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

## Limitations and future directions

- Global and panel-local persistent state
- Bidirectional bindings to and from inputs -- what if we want to adjust one input with another input?
- gnuplot-like syntax for describing plots and graphs easily

## Inspirations and further reading

- Bret Victor's [Scrubbing Calculator](http://worrydream.com/ScrubbingCalculator/) and [Up and Down the Ladder of Abstraction](http://worrydream.com/LadderOfAbstraction/)
- [Streamlit](https://streamlit.io/) and [Jupyter Notebook](https://jupyter.org/)
- [MDX](https://mdxjs.com/), a superset of Markdown with support for embedded JavaScript expressions and dynamic React components
- [Explorable Explanations](https://explorabl.es/), a collection of interactive explanatory essays on dynamical systems

