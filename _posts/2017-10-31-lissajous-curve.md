---
title: Lissajous 曲线
---

- [Lissajous 曲线的动画演示 - Matrix67][1]
- [Lissajous Curve - Wikipedia][2]

随着参数变化，方程

$$
\begin{cases}
    x(\theta) = a\sin(\theta) \\
    y(\theta) = b\sin(n\theta+\phi)
\end{cases}
(0\le\theta\le\frac{\pi}{2},n\ge1)
$$

将会画出一系列漂亮的曲线。这种曲线叫做 Lissajous 曲线，我们常常在示波器上看到它。下面我们用 [paper.js][3] 来现场画一个: P

[1]: https://www.matrix67.com/blog/archives/6947
[2]: https://en.wikipedia.org/wiki/Lissajous_curve
[3]: http://paperjs.org

<style>input{border:1px solid black;margin-left:.25em}</style>
<table>
<tr>
    <td><label for="a">x 速度</label></td>
    <td><input type="number" id="a" value="2"></td>
</tr>
<tr>
    <td><label for="b">y 速度</label></td>
    <td><input type="number" id="b" value="3"></td>
</tr>
<tr>
    <td><label for="phi"><script type="math/tex">\phi</script> 相位</label></td>
    <td><input type="number" id="phi" value="0" step="0.1"></td>
</tr>
<tr>
    <td><label for="smooth">平滑</label></td>
    <td><input id="smooth" type="checkbox"></td>
</tr>
</table>
<canvas style="width:100%;height:100%;margin:14px 0" id="lissajous" resize></canvas>
<script src="https://cdnjs.cloudflare.com/ajax/libs/paper.js/0.11.5/paper-full.min.js"></script>
<script>
'use strict';
(function(){
    paper.install(window);
    paper.setup(document.getElementById('lissajous'));
    var radius, pCenter, circle;
    var xA = 0, a = 2, xB = 0, b = 3, step = 0.01;
    var pA, pB, cA, cB, lA, lB;
    var cC, bpath, t = 0, target = 0;

    function init() {
        radius = paper.view.viewSize.height / 2;
        pCenter = paper.view.center;
    }

    function refresh(c) {
        xA = 0;
        xB = +phi.value;
        if (circle) circle.remove();
        circle = new Path.Circle(pCenter, radius);
        circle.strokeColor = 'skyblue';
        if (!pA) pA = new Point(pCenter.x - radius, radius);
        if (!pB) pB = new Point(pCenter.x - radius, radius);
        if (!cA) cA = new Path.Circle({
            center: pA, radius: 3, fillColor: 'red'
        });
        if (!cB) cB = new Path.Circle({
            center: pB, radius: 3, fillColor: 'green'
        });
        if (!cC) cC = new Path.Circle({
            center: [100, 100], radius: 3, fillColor: 'royalblue'
        });
        if (!lA) lA = new Path.Line(
            new Point(pCenter.x - radius, 0),
            new Point(pCenter.x - radius, radius * 2)
        );
        lA.strokeColor = 'darkred';
        if (!lB) lB = new Path.Line(
            new Point(pCenter.x - radius, radius),
            new Point(pCenter.x + radius, radius)
        );
        lB.strokeColor = 'darkgreen';
        if (bpath) bpath.remove();
        bpath = new Path();
        bpath.strokeColor = 'royalblue';
        circle.strokeColor.alpha = 1;
        lA.strokeColor.alpha     = 1;
        lB.strokeColor.alpha     = 1;
        cA.fillColor.alpha       = 1;
        cB.fillColor.alpha       = 1;
        cC.fillColor.alpha       = 1;
    }

    var ia = document.getElementById('a');
    var ib = document.getElementById('b');
    var phi = document.getElementById('phi');
    var smooth = document.getElementById('smooth');
    function onChange() {
        a = +ia.value;
        b = +ib.value;
        refresh();
        target = t + Math.round(Math.PI * 100);
        view.onFrame = function(event) {
            t = event.count;
            if (event.count <= target + 20) {
                pA.set(
                    pCenter.x + radius * Math.sin(xA += a * step),
                    pCenter.y + radius * Math.cos(xA += a * step)
                );
                cA.position = pA;
                cC.position.x = lA.position.x = pA.x;
                pB.set(
                    pCenter.x + radius * Math.sin(xB += b * step),
                    pCenter.y + radius * Math.cos(xB += b * step)
                );
                cB.position = pB;
                cC.position.y = lB.position.y = pB.y;
                bpath.add(cC.position);
                if (smooth.checked)
                    bpath.smooth();
            }
            if (event.count > target) {
                circle.strokeColor.alpha -= 0.05;
                lA.strokeColor.alpha     -= 0.05;
                lB.strokeColor.alpha     -= 0.05;
                cA.fillColor.alpha       -= 0.05;
                cB.fillColor.alpha       -= 0.05;
                cC.fillColor.alpha       -= 0.05;
                if (circle.strokeColor.alpha <= 0) {
                    view.onFrame = null;
                }
            }
        };
        view.draw();
    }
    ia.addEventListener('change', onChange, false);
    ib.addEventListener('change', onChange, false);
    phi.addEventListener('change', onChange, false);
    smooth.addEventListener('change', onChange, false);
    init();
    onChange();
})();
</script>

好玩吧。
