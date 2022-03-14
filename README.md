# Oak Notebook 🧪

!html <div class="meta">
    <a href="https://oaklang.org/" target="_blank">↖ Back to Oak</a>
    <a href="https://github.com/thesephist/x-oak-notebook" target="_blank">See on GitHub →</a>
</div>

**Oak Notebook** is an experimental tool for authoring [dynamic documents](https://thesephist.com/posts/notation/#dynamic-notation) with Markdown and [Oak](https://oaklang.org/). Oak Notebook is a way of writing documents with interactive, programmable "panels" for explaining and exploring complex ideas, and a "compiler" script that transforms such Markdown documents into HTML web pages.

I was inspired initially by [Streamlit](https://docs.streamlit.io/library/api-reference) and [Bret Victor's Scrubbing Calculator](http://worrydream.com/ScrubbingCalculator/) to explore ideas in this space, but what's here isn't a specific re-implementation of either of those concepts, and takes further inspirations from other products, experiments, and prior art.

For a demonstrative example, I want to introduce you to two interesting formulas for estimating the value of π. They're both derived from the [approximations of π using properties of the arctangent function](https://en.wikipedia.org/wiki/Machin-like_formula).

$$ \\pi = 4 \\arctan(1) = 4 \\left( 1 - \\frac{1}{3} + \\frac{1}{5} - \\frac{1}{7} + \\frac{1}{9} - \\cdots \\right) $$

$$ \\pi = 6 \\arctan\\left(\\frac{1}{\\sqrt{3}}\\right) = \\frac{6}{\\sqrt{3}} \\left( 1 - \\frac{1}{3 \\times 3} + \\frac{1}{5 \\times 3^2} - \\frac{1}{7 \\times 3^3} + \\cdots \\right) $$

They approach the true vale of π at different rates, but it's difficult to get a sense of what those approximations look like, how close they are, or what some specific margin of error really means. One way to get a better grasp of these concepts is through this panel. (Try tapping and dragging on the underlined number to adjust it.)

```notebook
max := nb.scrubbable(3, 1, 12, 1, ['Estimate π with ', ' terms.'])
units := nb.select('Units', ['Kilometers', 'Miles'])

fn display {
    nums := std.range(max.value)

    oneError := ?
    rootThreeError := ?

    nb.table([{
        sequence := nums |> with std.map() fn(n) {
            sign := if n % 2 {
                0 -> 1
                _ -> -1
            }
            odd := 2 * n + 1
            sign / odd
        }
        estimate := 4 * math.sum(sequence...)
        error := math.abs(math.Pi - estimate)
        oneError <- error
        {
            Angle: '1'
            'Estimate for π': estimate
            Error: error
            '% Error': string(math.round(error / estimate * 100, 6)) + '%'
        }
    }, {
        x := 1 / math.sqrt(3)
        sequence := nums |> with std.map() fn(n) {
            sign := if n % 2 {
                0 -> 1
                _ -> -1
            }
            odd := 2 * n + 1
            sign * pow(x, odd) / odd
        }
        estimate := 6 * math.sum(sequence...)
        error := math.abs(math.Pi - estimate)
        rootThreeError <- error
        {
            Angle: '√3'
            'Estimate for π': estimate
            Error: error
            '% Error': string(math.round(error / estimate * 100, 6)) + '%'
        }
    }])

    EarthCircumference := if units.value {
        'Kilometers' -> 40075.017
        _ -> 24901.461
    }
    nb.label(
        'If we measured the circumference of the Earth using these estimates of π,
        the estimate based on angle = 1 would be off by **{{0}}{{2}}**, while the estimate
        based on angle = √3 would be off by only **{{1}}{{2}}**.' |> fmt.format(
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

**Composability**. Authors should be able to compose and reuse customized groups of widgets by just writing Oak functions.

**End-user programmability**.

## Widget library

### Input widgets

**Button**

```oak
counter := nb.button('Count up!')

fn display {
    nb.label(counter.count, 'clicks,'
        'last clicked', datetime.format(int(counter.value)))
	nb.label(
        std.range(counter.count) |>
            std.map(fn '🌈') |>
            str.join(' ')
    )
}
```

```notebook
counter := nb.button('Count up!')

fn display {
    nb.label(counter.count, 'clicks,'
        'last clicked', datetime.format(int(counter.value)))
	nb.label(
        std.range(counter.count) |>
            std.map(fn '🌈') |>
            str.join(' ')
    )
}
```

**Checkbox**

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

**Number**

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

**Scrubbable**

```oak
rotation := nb.scrubbable(30, 0, 720, 1, ['Rotate me by ', 'º'])

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
rotation := nb.scrubbable(30, 0, 720, 1, ['Rotate me by ', 'º'])

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

**Text**

```oak
input := nb.text('Tell me a sentence', 'The quick brown fox jumped over the lazy dog')

fn display {
    wordCount := input.value |>
        str.split(' ') |>
        std.filter(fn(s) s != '') |>
        len()
    nb.label('There are', wordCount, 'words in this sentence!')
}
```

```notebook
input := nb.text('Tell me a sentence', 'The quick brown fox jumped over the lazy dog')

fn display {
    wordCount := input.value |>
        str.split(' ') |>
        std.filter(fn(s) s != '') |>
        len()
    nb.label('There are', wordCount, 'words in this sentence!')
}
```

**Prose**

```oak
paragraph := nb.prose('Story', 'All human beings are born free and equal in dignity and rights. They are endowed with reason and conscience and should act towards one another in a spirit of brotherhood.')

fn display {
    words := paragraph.value |>
        std.filter(fn(c) str.letter?(c) | c = ' ') |>
        str.lower() |>
        str.split(' ') |>
        std.compact()
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
        std.compact()
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

**Select**

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

- Graph or plot
- 2D color map
	- Path or surface over 2D canvas
- LaTeX formula with KaTeX, perhaps using the KaTeX-language code fence

**Label**

```oak
fn display {
    nb.label('Hello, World!')
}
```

```notebook
fn display {
    nb.label('Hello, World!')
}
```

**Table**

```notebook
fn display {
    nb.table([1, 2, 3, 4, 5])
}
```

```notebook
base := nb.scrubbable(2, 1, 20, 1, ['Base: ', ''])
maxExp := nb.scrubbable(5, 1, 25, 1, ['Show up to: ', ''])

fn display {
	nb.table(
		std.range(0, maxExp.value + 1, 1) |> with std.map() fn (exp) {
			{
				power: exp
				value: pow(base.value, exp)
			}
		}
	)
}
```

**HTML**

```oak
r := nb.scrubbable(180, 0, 255, 1, ['Red: ', ''])
g := nb.scrubbable(190, 0, 255, 1, ['Green: ', ''])
b := nb.scrubbable(216, 0, 255, 1, ['Blue: ', ''])

fn display {
    nb.html(
        fmt.format(
            '<div style="height:100px;width:100px;border-radius:4px;background:rgb({{0}},{{1}},{{2}})"></div>'
            r.value, g.value, b.value
        )
    )
}
```

```notebook
r := nb.scrubbable(180, 0, 255, 1, ['Red: ', ''])
g := nb.scrubbable(190, 0, 255, 1, ['Green: ', ''])
b := nb.scrubbable(216, 0, 255, 1, ['Blue: ', ''])

fn display {
    nb.html(
        fmt.format(
            '<div style="height:100px;width:100px;border-radius:4px;background:rgb({{0}},{{1}},{{2}})"></div>'
            r.value, g.value, b.value
        )
    )
}
```

## Limitations and future directions

- Global and panel-local persistent state
- Bidirectional bindings to and from inputs -- what if we want to adjust one input with another input?
- gnuplot-like syntax for describing plots and graphs easily:

```
f(x) = exp(-x**2 / 2)
plot [t=-4:4] f(t), t**2 / 16

plot [t=-4:4] f(t) title "Bell Curve", t**2 / 16 title "Parabola"
```

## Inspirations and further reading

- Bret Victor's [Scrubbing Calculator](http://worrydream.com/ScrubbingCalculator/) and [Up and Down the Ladder of Abstraction](http://worrydream.com/LadderOfAbstraction/)
- [Streamlit](https://streamlit.io/) and [Jupyter Notebook](https://jupyter.org/)
- [MDX](https://mdxjs.com/), a superset of Markdown with support for embedded React components
- [Explorable Explanations](https://explorabl.es/), a collection of interactive explanatory essays on dynamical systems

