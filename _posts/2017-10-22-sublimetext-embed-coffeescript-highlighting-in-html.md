---
title: Sublime Text 添加 HTML 内嵌 CoffeeScript 高亮
---

1. Install package `Better CoffeeScript`.
2. Preferences > Browse Packages.  
   e.g. `C:\Users\<yourname>\AppData\Roaming\Sublime Text 3\Packages`
3. Goto [sublimehq/Packages][1], get *HTML.sublime-syntax*.
4. Put it in `Packages\HTML`.  
   e.g. `%AppData%\Sublime Text 3\Packages\HTML\HTML.sublime-syntax`
5. Edit it, add something about coffeescript.  
   Above line 129 (just below embeded js part), insert:

<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/9.12.0/languages/yaml.min.js"></script>

```yml
- match: '(<)((?i:script))\b(?![^>]*/>)(?![^>]*(?i:type.?=.?text/((?!coffeescript).*)))'
  captures:
    0: meta.tag.script.begin.html
    1: punctuation.definition.tag.begin.html
    2: entity.name.tag.script.html
  push:
    - match: (?i)(-->)?\s*(</)(script)(>)
      captures:
        0: meta.tag.script.end.html
        1: comment.block.html punctuation.definition.comment.html
        2: punctuation.definition.tag.begin.html
        3: entity.name.tag.script.html
        4: punctuation.definition.tag.end.html
      pop: true
    - match: '(>)\s*(<!--)?'
      captures:
        1: meta.tag.script.begin.html punctuation.definition.tag.end.html
        2: comment.block.html punctuation.definition.comment.html
      push:
        - meta_content_scope: source.coffee.embedded.html
        - include: 'scope:source.coffee'
      with_prototype:
         - match: (?i)(?=(-->)?\s*</script)
           pop: true
    - match: ''
      push:
        - meta_scope: meta.tag.script.begin.html
        - match: '(?=>)'
          pop: true
        - include: tag-stuff
```

#### Use CoffeeScript in browser

It's not recommended, though.

```html
<script type="text/coffeescript">
  console.log 'it works'
</script>
<script src="http://coffeescript.org/browser-compiler/coffeescript.js"></script>
```

[1]: https://raw.githubusercontent.com/sublimehq/Packages/master/HTML/HTML.sublime-syntax
