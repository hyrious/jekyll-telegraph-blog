---
title: caret.css
---
```css
@keyframes blink { from { opacity: 1 } to { opacity: 0; filter: blur(1px); } }
.caret::after {
  content: "";
  border-right: 2px solid #777;
  animation: .6s blink infinite alternate cubic-bezier(0.5,-0.34, 0.15, 0.76)
}
```

<style>
  @keyframes blink { from { opacity: 1 } to { opacity: 0; filter: blur(1px); } }
  .caret::after {
    content: "";
    border-right: 2px solid #777;
    animation: .6s blink infinite alternate cubic-bezier(0.5,-0.34, 0.15, 0.76)
  }
</style>

右边是一个在线的例子 <span class="caret"></span>