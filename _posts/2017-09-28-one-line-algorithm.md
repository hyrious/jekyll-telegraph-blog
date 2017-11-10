---
title: One line algorithm
---

```ruby
Node = Struct.new :x, :l, :r
#     +
#    / \
#   *   9
#  / \
# 5   4
x = Node.new(:+, Node.new(:*, Node.new(5), Node.new(4)), Node.new(9))
```

- [Depth First Search](https://en.wikipedia.org/wiki/Depth-first_search)

```ruby
def dfs(start, &f)
  yield start, -> a { dfs(a, &f) }
end

p dfs(x) { |x, y| x ? x.l ? [x.x, y.(x.l), y.(x.r)] : x.x : '' }
#=> [:+, [:*, 5, 4], 9]
```

- [Breadth First Search](https://en.wikipedia.org/wiki/Breadth-first_search)

```ruby
def bfs(start = [])
  yield start.pop, start.method(:unshift) until start.empty?
end

p [].instance_exec {
  bfs([x]) { |x, u| x ? x.l ? (u.(x.l); u.(x.r); push(x.x)) : push(x.x) : () }
  self
}
#=> [:+, :*, 9, 5, 4]
```

- [Operator Precedence Grammar](https://en.wikipedia.org/wiki/Operator-precedence_grammar)

```ruby
def parse(str, start = [], &f)
  start.instance_exec(str.method(:slice!), &f)
end

p parse('5 * 4 + 9') { |s|
  r = -> { a, b, c = *pop(3); push([b, a, c]) }
  begin
    a = nil
    case
    when a = s[/^\d+/  ] then push a.to_i
    when a = s[/^\+|\-/] then r.(); push(a.to_sym)
    when a = s[/^\*|\//] then r.() while %i[+ -].include?(a[-2]); push(a.to_sym)
    when a = s[/^\s+/  ]
    end
  end while a
  r.() while size > 1
  self[0]
}
#=> [:+, [:*, 5, 4], 9]
```
