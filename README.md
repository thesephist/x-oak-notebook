# Oak Notebook 🧪

!html <div class="meta">
    <a href="https://oaklang.org/" target="_blank">↖ Back to Oak</a>
    <a href="https://github.com/thesephist/x-oak-notebook" target="_blank">See on GitHub →</a>
</div>

**Oak Notebook** is an experimental tool for authoring [dynamic documents](https://thesephist.com/posts/notation/#dynamic-notation) with Markdown and [Oak](https://oaklang.org/).

```notebook
max := nb.scrubbable(3, 1, 10, 1, ['Estimate π with ', ' terms.'])

fn display {
    nums := std.range(max.value)
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
        {
            angle: '1'
            pi: estimate
            error: math.abs(math.Pi - estimate)
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
        {
            angle: '√3'
            pi: estimate
            error: math.abs(math.Pi - estimate)
        }
    }])
}
```

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

