---
title: PEG and Syntax Highlighting (1)
---

Syntax Highlighting is often implemented by Regular Expressions (like
[highlight.js][6] this blog uses), but it's common to cause
[mis-highlighting][2] especially when it comes to languages like Ruby (where
the parentheses can be omitted).

[PEG][1] (Parsing Expression Grammar) is a set of rules describing a formal
language without ambiguousness. In fact, Some IDEs (like Jet Brains') have
been using [formal grammar][3] on syntax highlighting and code analysis for
a long time.

It's quite easy to implement one. Let's do it. (will use JavaScript)

- - -

If you don't know PEG, here the [Basics][4].

If you are not familiar with ES6 syntaxes, Google it, or keep reading. I only
use a tiny subset of it.

Okay, here we go.

#### Preparations

To make things easier, we need a data structure to present PEG expression
units. How about `{ type: 'OneOrMore', value: [SubExpressions] }` ?

```js
const Dot   = ()        => ({ type: '.' })
const Id    = str       => ({ type: 'i', value: str })
const Ques  = exp       => ({ type: '?', value: exp })
const Star  = exp       => ({ type: '*', value: exp })
const Plus  = exp       => ({ type: '+', value: exp })
const Seq   = (...exp)  => ({ type: 's', value: exp })
const Or    = (...exp)  => ({ type: '/', value: exp })
const Not   = exp       => ({ type: '!', value: exp })
const And   = exp       => ({ type: '&', value: exp })
```

The `value` is like `params` or `child`, whatever. We still missed one thing
called `Range` which is like `[a-zA-Z_]`, I'm so lazy that I replaced it by
`Or` :

```js
const Range = str       => {
  let exp = []
  for (let i = 0, ch; ch = str.charAt(i); ++i) {
    if (ch === '-')
      for (let last = str.charCodeAt(i - 1) + 1, next = str.charCodeAt(++i);
        last <= next; ++last)
        exp.push(String.fromCharCode(last))
    else
      exp.push(ch)
  }
  return Or(...exp)
}
```

"Oh No", you may ask, "What's the **literals** and **eps** ?"

We treat any string as literal, and we all know that `eps = !.` (Don't use
`""` (empty string) because we use strings to match real characters).

- - -

Seems enough! Lets translate <a name="PEG">PEG.peg</a> from the [paper][5] into data:

```js
const PEG = {
  Grammar:    Seq(Id('Spacing'), Plus(Id('Definition')), Id('EndOfFile')),
  Definition: Seq(Id('Identifier'), Id('LEFTARROW'), Id('Expression')),
  Expression: Seq(Id('Sequence'), Star(Seq(Id('SLASH'), Id('Sequence')))),
  Sequence:   Star(Id('Prefix')),
  Prefix:     Seq(Ques(Or(Id('AND'), Id('NOT'))), Id('Suffix')),
  Suffix:     Seq(Id('Primary'), Ques(Or(Id('QUESTION'), Id('STAR'),
    Id('PLUS')))),
  Primary:    Or(
    Seq(Id('Identifier'), Not(Id('LEFTARROW'))),
    Seq(Id('OPEN'), Id('Expression'), Id('CLOSE')),
    Id('Literal'), Id('Class'), Id('DOT')),
  Identifier: Seq(Id('IdentStart'), Star(Id('IdentCont')), Id('Spacing')),
  IdentStart: Range('a-zA-Z_'),
  IdentCont:  Or(Id('IdentStart'), Range('0-9')),
  Literal:    Or(
    Seq("'", Star(Seq(Not("'"), Id('Char'))), "'", Id('Spacing')),
    Seq('"', Star(Seq(Not('"'), Id('Char'))), '"', Id('Spacing'))),
  Class:      Seq('[', Star(Seq(Not(']'), Id('Range'))), ']', Id('Spacing')),
  Range:      Or(Seq(Id('Char'), '-', Id('Char')), Id('Char')),
  Char:       Or(
    Seq('\\', Range('nrt\'"\[\]\\')), 
    Seq('\\', Range('0-2'), Range('0-7'), Range('0-7')),
    Seq('\\', Range('0-7'), Ques(Range('0-7'))), Seq(Not('\\'), Dot())),
  LEFTARROW:  Seq('<-', Id('Spacing')),
  SLASH:      Seq('/', Id('Spacing')),
  AND:        Seq('&', Id('Spacing')),
  NOT:        Seq('!', Id('Spacing')),
  QUESTION:   Seq('?', Id('Spacing')),
  STAR:       Seq('*', Id('Spacing')),
  PLUS:       Seq('+', Id('Spacing')),
  OPEN:       Seq('(', Id('Spacing')),
  CLOSE:      Seq(')', Id('Spacing')),
  DOT:        Seq('.', Id('Spacing')),
  Spacing:    Star(Or(Id('Space'), Id('Comment'))),
  Comment:    Seq('#', Star(Seq(Not(Id('EndOfLine')), Dot())),
    Id('EndOfLine')),
  Space:      Or(' ', '\t', Id('EndOfLine')),
  EndOfLine:  Or('\r\n', '\n', '\r'),
  EndOfFile:  Not(Dot())
}
```

Quite a hard work if you really code it by hand =)

- - -

#### Lexer

Now we should write a function named `lex(str)`, which takes codes waiting
for highlighting, and then returns the codes with highlight marks. It is
more commonly called *lex* or *tokenize* .

What's more, have we looked for methods to match a substring? It's easy,
but **do not use regexp** 'cuz regexp may do skip. Just `String#startsWith()`
is enough.

To simplify works, now we just **lex** with the PEG.peg defined above.

Here we are:

```js
const lex = (str, ptn = PEG.Grammar, pos = 0, id = 'Grammar') => {
  if (typeof ptn === 'string' && str.startsWith(ptn, pos))
    return { id: id, child: ptn, length: ptn.length }
}
```

If we call `lex('abc', 'ab', 0, 'A')` , we will get
`{ id: 'A', child: 'ab', length: 2 }` .

Let's work on different patterns, and add them to `lex()` .

The first one is **Dot** :

```js
  if (ptn.type === '.' && str[pos])
    return { id: id, child: str[pos], length: 1 }
```

**Id** :

```js
  if (ptn.type === 'i')
    return lex(str, PEG[ptn.value], pos, ptn.value)
```

**Seq** is like "and":

```js
  if (ptn.type === 's') {
    let ret = { id: id, child: [], length: 0 }
    for (let _ptn of ptn.value) {
      let _ret = lex(str, _ptn, pos + ret.length, id)
      if (!_ret) return null
      ret.child.push(_ret)
      ret.length += _ret.length
    }
    if (ret.length) return ret
  }
```

**Or** is "or", hummm..:

```js
  if (ptn.type === '/') {
    for (let _ptn of ptn.value) {
      let ret = lex(str, _ptn, pos, id)
      if (ret) return ret
    }
  }
```

When **Ques** fails, it returns "eps". (we define "eps" as any node with zero length.) :

```js
  if (ptn.type === '?')
    return lex(str, ptn.value, pos, id) || { id: id, child: [], length: 0 }
```

**Star** and **Plus**, there's only one different line between them:

```js
  if (ptn.type === '*') {
    let ret = { id: id, child: [], length: 0 }, _ret
    while (_ret = lex(str, ptn.value, pos + ret.length, id)) {
      ret.child.push(_ret)
      ret.length += _ret.length
      if (!_ret.length) break
    }
    return ret
  }
```

```js
  if (ptn.type === '+') {
    let ret = { id: id, child: [], length: 0 }, _ret
    while (_ret = lex(str, ptn.value, pos + ret.length, id)) {
      ret.child.push(_ret)
      ret.length += _ret.length
      if (!_ret.length) break
    }
    if (ret.length) return ret
  }
```

**Not** returns "eps" if failed, **And** does opposite:

```js
  if (ptn.type === '!' && !lex(str, ptn.value, pos, id))
    return { id: id, child: [], length: 0 }
```

```js
  if (ptn.type === '&' && lex(str, ptn.value, pos, id))
    return { id: id, child: [], length: 0 }
```

The last thing:

```js
  return null
```

- - -

Now call `lex('String <- .+')`, we get:

```json
{"id":"Grammar","child":[{"id":"Spacing","child":[{"id":"EndOfLine","child":"\n","length":1},{"id":"Space","child":" ","length":1},{"id":"Space","child":" ","length":1}],"length":3},{"id":"Grammar","child":[{"id":"Definition","child":[{"id":"Identifier","child":[{"id":"IdentStart","child":"S","length":1},{"id":"Identifier","child":[{"id":"IdentStart","child":"t","length":1},{"id":"IdentStart","child":"r","length":1},{"id":"IdentStart","child":"i","length":1},{"id":"IdentStart","child":"n","length":1},{"id":"IdentStart","child":"g","length":1}],"length":5},{"id":"Spacing","child":[{"id":"Space","child":" ","length":1}],"length":1}],"length":7},{"id":"LEFTARROW","child":[{"id":"LEFTARROW","child":"<-","length":2},{"id":"Spacing","child":[{"id":"Space","child":" ","length":1}],"length":1}],"length":3},{"id":"Expression","child":[{"id":"Sequence","child":[{"id":"Prefix","child":[{"id":"Prefix","child":[],"length":0},{"id":"Suffix","child":[{"id":"DOT","child":[{"id":"DOT","child":".","length":1},{"id":"Spacing","child":[],"length":0}],"length":1},{"id":"PLUS","child":[{"id":"PLUS","child":"+","length":1},{"id":"Spacing","child":[{"id":"EndOfLine","child":"\n","length":1},{"id":"Space","child":" ","length":1},{"id":"Space","child":" ","length":1}],"length":3}],"length":4}],"length":5}],"length":5}],"length":5},{"id":"Expression","child":[],"length":0}],"length":5}],"length":15}],"length":15},{"id":"EndOfFile","child":[],"length":0}],"length":18}
```

#### Pretty

Oops, lets **cut** down some nodes with zero length, and then **merge** nodes
with the same *id* .

```js
const cut = (node) => {
  if (typeof node.child === 'string') return node
  node.child = node.child.filter(x => x.length).map(x => cut(x))
  if (node.child[0].id === node.id && node.child[0].length === node.length)
    return node.child[0]
  return node
}

const merge = (node) => {
  if (typeof node.child === 'string') return node
  let _child = [], _cur, _id
  for (let _node of node.child) {
    if (_node.id === _id && typeof _cur.child === typeof _node.child) {
      _cur.child = _cur.child.concat(_node.child)
      _cur.length += _node.length
    } else {
      _child.push(_cur = _node)
      _id = _node.id
    }
  }
  node.child = _child.map(x => merge(x))
  return node
}
```

We can also pretty the node:

```js
const pretty = (node, id = []) => {
  let _ret = [] // [{ token: '# \n', id: [ 'Grammar', 'Spacing', 'Comment' ] }
  const _pretty = (_node, _id) => {
    if (typeof _node.child === 'string')
      return _ret.push({ token: _node.child, id: _id })
    for (let __node of _node.child)
      if (_id[_id.length - 1] !== _node.id)
        _pretty(__node, _id.concat([_node.id]))
      else
        _pretty(__node, _id)
  }
  _pretty(node, id)
  let ret = [], _cur, _id // merge items of _ret with the same [id]
  for (let x of _ret) {
    if (_id && x.id.toString() === _id.toString())
      _cur.token += x.token
    else {
      ret.push(_cur = x)
      _id = x.id
    }
  }
  return ret
}
```

Fine, have a try:

```js
let tokens = pretty(merge(cut(lex(`
# test
A <- B (C+ D)? !B
`))))
for (let { token, id } of tokens)
  console.log(JSON.stringify(token).padStart(10), id.join('.'))
```

Returns:

```
      "\n" Grammar.Spacing
"# test\n" Grammar.Spacing.Comment
       "A" Grammar.Definition.Identifier
       " " Grammar.Definition.Identifier.Spacing
      "<-" Grammar.Definition.LEFTARROW
       " " Grammar.Definition.LEFTARROW.Spacing
       "B" Grammar.Definition.Expression.Sequence.Prefix.Suffix.Primary.Identifier
       " " Grammar.Definition.Expression.Sequence.Prefix.Suffix.Primary.Identifier.Spacing
       "(" Grammar.Definition.Expression.Sequence.Prefix.Suffix.Primary
       "C" Grammar.Definition.Expression.Sequence.Prefix.Suffix.Primary.Expression.Sequence.Prefix.Suffix.Primary.Identifier
       "+" Grammar.Definition.Expression.Sequence.Prefix.Suffix.Primary.Expression.Sequence.Prefix.Suffix.PLUS
       " " Grammar.Definition.Expression.Sequence.Prefix.Suffix.Primary.Expression.Sequence.Prefix.Suffix.PLUS.Spacing
       "D" Grammar.Definition.Expression.Sequence.Prefix.Suffix.Primary.Expression.Sequence.Prefix.Suffix.Primary.Identifier
       ")" Grammar.Definition.Expression.Sequence.Prefix.Suffix.Primary
       "?" Grammar.Definition.Expression.Sequence.Prefix.Suffix.QUESTION
       " " Grammar.Definition.Expression.Sequence.Prefix.Suffix.QUESTION.Spacing
       "!" Grammar.Definition.Expression.Sequence.Prefix
       "B" Grammar.Definition.Expression.Sequence.Prefix.Suffix.Primary.Identifier
      "\n" Grammar.Definition.Expression.Sequence.Prefix.Suffix.Primary.Identifier.Spacing
```

Oh yes, it's easy to output things above in form of
`<span class="id"> token </span>` , which means it is enough for syntax
highlighting.

<pre><code class="nohighlight"><style scoped>
  .Class { color: darkgreen }
  .Range { color: green }
  .Literal { color: purple }
  .Identifier { color: black }
  .Comment { color: darkgrey }
  .LEFTARROW { color: darkblue }
  .PLUS,.STAR,.QUESTION { color: darkcyan }
  .AND,.NOT { color: darkorange }
  .DOT { color: red }
  .SLASH { color: silver }
</style><script>
'use strict'

const Dot   = ()        => ({ type: '.' })
const Id    = str       => ({ type: 'i', value: str })
const Ques  = exp       => ({ type: '?', value: exp })
const Star  = exp       => ({ type: '*', value: exp })
const Plus  = exp       => ({ type: '+', value: exp })
const Seq   = (...exp)  => ({ type: 's', value: exp })
const Or    = (...exp)  => ({ type: '/', value: exp })
const Range = str       => {
  let exp = []
  for (let i = 0, ch; ch = str.charAt(i); ++i) {
    if (ch === '-')
      for (let last = str.charCodeAt(i - 1) + 1, next = str.charCodeAt(++i);
        last <= next; ++last)
        exp.push(String.fromCharCode(last))
    else
      exp.push(ch)
  }
  return Or(...exp)
}
const Not   = exp       => ({ type: '!', value: exp })
const And   = exp       => ({ type: '&', value: exp })

const PEG = {
  Grammar:    Seq(Id('Spacing'), Plus(Id('Definition')), Id('EndOfFile')),
  Definition: Seq(Id('Identifier'), Id('LEFTARROW'), Id('Expression')),
  Expression: Seq(Id('Sequence'), Star(Seq(Id('SLASH'), Id('Sequence')))),
  Sequence:   Star(Id('Prefix')),
  Prefix:     Seq(Ques(Or(Id('AND'), Id('NOT'))), Id('Suffix')),
  Suffix:     Seq(Id('Primary'), Ques(Or(Id('QUESTION'), Id('STAR'),
    Id('PLUS')))),
  Primary:    Or(
    Seq(Id('Identifier'), Not(Id('LEFTARROW'))),
    Seq(Id('OPEN'), Id('Expression'), Id('CLOSE')),
    Id('Literal'), Id('Class'), Id('DOT')),
  Identifier: Seq(Id('IdentStart'), Star(Id('IdentCont')), Id('Spacing')),
  IdentStart: Range('a-zA-Z_'),
  IdentCont:  Or(Id('IdentStart'), Range('0-9')),
  Literal:    Or(
    Seq("'", Star(Seq(Not("'"), Id('Char'))), "'", Id('Spacing')),
    Seq('"', Star(Seq(Not('"'), Id('Char'))), '"', Id('Spacing'))),
  Class:      Seq('[', Star(Seq(Not(']'), Id('Range'))), ']', Id('Spacing')),
  Range:      Or(Seq(Id('Char'), '-', Id('Char')), Id('Char')),
  Char:       Or(
    Seq('\\', Range('nrt\'"\[\]\\')), 
    Seq('\\', Range('0-2'), Range('0-7'), Range('0-7')),
    Seq('\\', Range('0-7'), Ques(Range('0-7'))), Seq(Not('\\'), Dot())),
  LEFTARROW:  Seq('<-', Id('Spacing')),
  SLASH:      Seq('/', Id('Spacing')),
  AND:        Seq('&', Id('Spacing')),
  NOT:        Seq('!', Id('Spacing')),
  QUESTION:   Seq('?', Id('Spacing')),
  STAR:       Seq('*', Id('Spacing')),
  PLUS:       Seq('+', Id('Spacing')),
  OPEN:       Seq('(', Id('Spacing')),
  CLOSE:      Seq(')', Id('Spacing')),
  DOT:        Seq('.', Id('Spacing')),
  Spacing:    Star(Or(Id('Space'), Id('Comment'))),
  Comment:    Seq('#', Star(Seq(Not(Id('EndOfLine')), Dot())),
    Id('EndOfLine')),
  Space:      Or(' ', '\t', Id('EndOfLine')),
  EndOfLine:  Or('\r\n', '\n', '\r'),
  EndOfFile:  Not(Dot())
}

const lex = (str, ptn = PEG.Grammar, pos = 0, id = 'Grammar', indent = 0) => {
  if (typeof ptn === 'string' && str.startsWith(ptn, pos))
    return { id: id, child: ptn, length: ptn.length }
  // if (ptn.type)
  //   document.write('&nbsp;'.repeat(indent) +
  //     `Matching (${ptn.type}) ${pos} ${JSON.stringify(ptn.value)}<br>`)
  if (ptn.type === '.' && str[pos])
    return { id: id, child: str[pos], length: 1 }
  if (ptn.type === 'i')
    return lex(str, PEG[ptn.value], pos, ptn.value, indent + 1)
  if (ptn.type === 's') {
    let ret = { id: id, child: [], length: 0 }
    for (let _ptn of ptn.value) {
      let _ret = lex(str, _ptn, pos + ret.length, id, indent + 1)
      if (!_ret) return null
      ret.child.push(_ret)
      ret.length += _ret.length
    }
    if (ret.length) return ret
  }
  if (ptn.type === '/') {
    for (let _ptn of ptn.value) {
      let ret = lex(str, _ptn, pos, id, indent + 1)
      if (ret) return ret
    }
  }
  if (ptn.type === '?')
    return lex(str, ptn.value, pos, id, indent + 1) || { id: id, child: [], length: 0 }
  if (ptn.type === '*') {
    let ret = { id: id, child: [], length: 0 }, _ret
    while (_ret = lex(str, ptn.value, pos + ret.length, id, indent + 1)) {
      ret.child.push(_ret)
      ret.length += _ret.length
      if (!_ret.length) break
    }
    return ret
  }
  if (ptn.type === '+') {
    let ret = { id: id, child: [], length: 0 }, _ret
    while (_ret = lex(str, ptn.value, pos + ret.length, id, indent + 1)) {
      ret.child.push(_ret)
      ret.length += _ret.length
      if (!_ret.length) break
    }
    if (ret.length) return ret
  }
  if (ptn.type === '!' && !lex(str, ptn.value, pos, id, indent + 1))
    return { id: id, child: [], length: 0 }
  if (ptn.type === '&' && lex(str, ptn.value, pos, id, indent + 1))
    return { id: id, child: [], length: 0 }
  return null
}

const cut = (node) => {
  if (typeof node.child === 'string') return node
  node.child = node.child.filter(x => x.length).map(x => cut(x))
  if (node.child[0].id === node.id && node.child[0].length === node.length)
    return node.child[0]
  return node
}

const merge = (node) => {
  if (typeof node.child === 'string') return node
  let _child = [], _cur, _id
  for (let _node of node.child) {
    if (_node.id === _id && typeof _cur.child === typeof _node.child) {
      _cur.child = _cur.child.concat(_node.child)
      _cur.length += _node.length
    } else {
      _child.push(_cur = _node)
      _id = _node.id
    }
  }
  node.child = _child.map(x => merge(x))
  return node
}

const pretty = (node, id = []) => {
  let _ret = [] // [{ token: '# \n', id: [ 'Grammar', 'Spacing', 'Comment' ] }
  const _pretty = (_node, _id) => {
    if (typeof _node.child === 'string')
      return _ret.push({ token: _node.child, id: _id })
    for (let __node of _node.child)
      if (_id[_id.length - 1] !== _node.id)
        _pretty(__node, _id.concat([_node.id]))
      else
        _pretty(__node, _id)
  }
  _pretty(node, id)
  let ret = [], _cur, _id // merge items of _ret with the same [id]
  for (let x of _ret) {
    if (_id && x.id.toString() === _id.toString())
      _cur.token += x.token
    else {
      ret.push(_cur = x)
      _id = x.id
    }
  }
  return ret
}

const make = (node) => {
  if (typeof node.child === 'string')
    return `<span class="${node.id}">${node.child}</span>`
  let ret = `<span class="${node.id}">`
  for (let _node of node.child)
    ret += make(_node)
  return ret + '</span>'
}

let nodes = merge(cut(lex(`# Hierarchical syntax
Grammar    <- Spacing Definition+ EndOfFile
Definition <- Identifier LEFTARROW Expression
Expression <- Sequence (SLASH Sequence)*
Sequence   <- Prefix*
Prefix     <- (AND / NOT)? Suffix
Suffix     <- Primary (QUESTION / STAR / PLUS)?
Primary    <- Identifier !LEFTARROW
            / OPEN Expression CLOSE
            / Literal / Class / DOT
# Lexical syntax
Identifier <- IdentStart IdentCont* Spacing
IdentStart <- [a-zA-Z_]
IdentCont  <- IdentStart / [0-9]
Literal    <- ['] (!['] Char)* ['] Spacing
            / ["] (!["] Char)* ["] Spacing
Class      <- '[' (!']' Range)* ']' Spacing
Range      <- Char '-' Char / Char
Char       <- '\\\\' [nrt'"\\[\\]\\\\]
            / '\\\\' [0-2][0-7][0-7]
            / '\\\\' [0-7][0-7]?
            / !'\\\\' .
LEFTARROW  <- '<-' Spacing
SLASH      <- '/' Spacing
AND        <- '&' Spacing
NOT        <- '!' Spacing
QUESTION   <- '?' Spacing
STAR       <- '*' Spacing
PLUS       <- '+' Spacing
OPEN       <- '(' Spacing
CLOSE      <- ')' Spacing
DOT        <- '.' Spacing
Spacing    <- (Space / Comment)*
Comment    <- '#' (!EndOfLine .)* EndOfLine
Space      <- ' ' / '\\t' / EndOfLine
EndOfLine  <- '\\r\\n' / '\\n' / '\\r'
EndOfFile  <- !.`)))

document.write(make(nodes))
// for (let { token, id } of pretty(nodes))
//   console.log(JSON.stringify(token).padStart(10), id.join('.'))
</script></code></pre>

(Should be colorful code, if you see nothing, use the latest chrome.)

- - -

That's all? No! Currently the `lex()` function returns what we want only if
the whole syntax is right, but highlighting needs it always return something
to make world more colorful (ry.

BTW, we want it handles not only the PEG grammar but also other languages,
how to do that? How about a translator that turns plain PEG rules to something
like [`const PEG`](#PEG) above?

To be continued.

[1]: https://en.wikipedia.org/wiki/Parsing_expression_grammar
[2]: https://github.com/sublimehq/Packages/issues?utf8=%E2%9C%93&q=is%3Aissue%20highlight
[3]: https://en.wikipedia.org/wiki/Formal_grammar
[4]: https://github.com/PhilippeSigaud/Pegged/wiki/PEG-Basics
[5]: https://pdos.csail.mit.edu/~baford/packrat/popl04/
[6]: https://highlightjs.org