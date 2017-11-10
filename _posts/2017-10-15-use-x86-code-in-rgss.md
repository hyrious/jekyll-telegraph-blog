---
title: RGSS 中执行机器码
---

<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/9.12.0/languages/x86asm.min.js"></script>

> 本文同时发布在 [简书][3] 和 [知乎][4] 并附有效果图，这里没有效果图。

给大家认识一下，这是我们今天的主角↓

#### [CallWindowProc][1]

```ruby
CallWindowProc = Win32API.new 'user32', 'CallWindowProc', 'pLLLL', 'i'
```

这个 API 接受 5 个参数，`p` 是机器码（字符串）的地址，四个 `L` 是可选参数，返回值由 `%eax` 带出。

```
p | *pointer     | RGSS 为 32 位运行环境，在这里是 4 字节的
L | unsigned int | pack 的时候正负数无关，例如 [-1].pack('i') 和 [-1].pack('L') 得到的结果是一样的
i | signed_int   | unpack 的时候正负数有关，[-1].pack('L').unpack('L') 会得到 2^32-1
```

#### A+B

考虑一个汇编子程，参数为两个 `int`，将求和后的结果保存到 `%eax`。

```x86asm
sum:
  push %ebp
  mov %esp,%ebp
  mov 8(%ebp),%eax  # 第一个参数
  add 12(%ebp),%eax # 第二个参数，直接加给 %eax
  leave
  ret $8 # 两个 int = 4 * 2
```

实际上堆栈框架（push ebp ~ leave）是有点浪费字节的，我们把它去掉：

```x86asm
sum:
  mov 4(%esp),%eax # 第一个参数
  add 8(%esp),%eax # 第二个参数，直接加给 %eax
  ret $8 # 两个 int = 4 * 2
```

| 指令 | 机器码 |
|--------|----------|
| mov | `0x8b ModR/M SIB Disp Imm` |
| add | `0x03 ModR/M SIB Disp Imm` | 
| ret | `0xc2 Imm16` | 

上面这段子程的机器码可以翻译为：

```ruby
[
  0x8b, 0104,0044, 4,  # [0] mov 4(%esp), %eax
  0x03, 0104,0044, 8,  # [1] add 8(%esp), %eax
  0xc2,          8,0,  #     ret $8
].pack('C*')
```

[更多机器码参考这里][2]

由于我们一开始写了四个 `L` 参数的声明，这里修正为 `ret $16`，然后我们试一下这个 API：

```ruby
p CallWindowProc.call [
  0x8b, 0104,0044, 4,  # [0] mov 4(%esp), %eax
  0x03, 0104,0044, 8,  # [1] add 8(%esp), %eax
  0xc2,         16,0,  #     ret $16
].pack('C*'), 3, 5, 0, 0
```

（RGSS3 请使用 msgbox 或者打开控制台选项来看输出）

#### Bitmap

下面我们泄露一个 Bitmap 的内存结构：

```ruby
[0,0,0,0,[0,0,[0,0,0,0,pRData]]]
```

RData 里存的是每个像素的 BGRA 信息。另外，RGSS 中 object_id * 2 是 Bitmap 对象的真实地址位置。

0 代表四字节数据并且不关心，我们要得到这个 `pRData`：

```ruby
class Bitmap
  GETADDR = [
    0x8b, 0104,0044, 4,  # mov  4(%esp), %eax
    0x8b, 0100,     16,  # mov 16(%eax), %eax
    0x8b, 0100,      8,  # mov  8(%eax), %eax
    0x8b, 0100,     16,  # mov 16(%eax), %eax
    0xc2,         16,0,  # ret $16
  ].pack('C*')
  def addr
    @_addr ||= CallWindowProc.call GETADDR, object_id * 2, 0, 0, 0
  end
end
```

有了位图数据的首地址，就可以开始搞事了：

#### Pixel

首先泄露一下位图数据的内存形式为：

```
BGRABGRABGRABGRABGRABGRABGRABGRA...
```

共 `width * height` 个 `BGRA`。

考虑一个简单的反色算法：把每个 pixel 的 `BGR` 数据都取反。

C 语言形式如下：

```cpp
for (int i = 0; i < length; ++i)
  data[i * 4] ^= 0x00FFFFFF; // AARRGGBB
```

不难写出这样的代码：

```ruby
def callproc code, a = 0, b = 0, c = 0, d = 0
  code = code.pack 'C*' if Array === code
  CallWindowProc.call code, a, b, c, d
end
class Bitmap
  INVERSE = [
    0x8b, 0104,0044, 4,  # mov  4(%esp), %eax   addr
    0x8b, 0114,0044, 8,  # mov  8(%esp), %ecx   length
    0x81, 0060,   0xff,0xff,0xff,0x00,
                         # xorl  (%eax), 0x00FFFFFF # 0+1|4&5-6^
    0x83, 0300,      4,  # add       $4, %eax
    0xe2,          -11,  # loop     -11
    0xc2,         16,0,  # ret      $16
  ].pack('C*')
  def inverse!
    callproc INVERSE, addr, width * height
    self
  end
end
```

下面我们再写一个，伪色差效果：把整个图的某个通道整体左/右移 offset 个像素。

C 代码类似下面这样：

```cpp
// phase  通道，假设一定是 0,1,2,3 中的一个
// offset 偏移像素距离，可能为负数
for (int i = 0; i < length; ++i)
  data[i + phase] = data[i + offset * 4 + phase];
// 上面一定会产生越界错误，我们修正一下
if (offset == 0) return;
length -= abs(offset);
if (offset > 0)
  for (int i = 0; i < length; ++i)
    data[i + phase] = data[i + offset * 4 + phase];
else // offset < 0
  for (int i = length; i > 0; --i)
    data[i - offset * 4 + phase] = data[i + phase];
```

出来的机器码大概是这个样子：

```ruby
class Bitmap
  SIMPLEABERRATION = [
    0x8b, 0104,0044, 4,  # [0] mov  4(%esp), %eax   addr
    0x8b, 0114,0044, 8,  # [1] mov  8(%esp), %ecx   length
    0x8b, 0164,0044,12,  # [2] mov 12(%esp), %esi   offset (can be neg)
    0x03, 0104,0044,16,  # [3] add 16(%esp), %eax   phase (% 4)
                         # ------------------------------- #
    0xbb,      4,0,0,0,  #     mov       $4, %ebx          #
    0x83, 0376,      0,  #     cmp       $0, %esi          #
    0x74,           23,  # .-- je        23                #
    0x7f,           10,  # |.- jg        10                #
    0x8d, 0104,0210,-4,  # ||  lea -4(%eax,%ecx,4),%eax    #
    0xf7, 0333,          # ||  neg    %ebx                 #
    0x01, 0361,          # ||  add    %esi , %ecx          #
    0xeb,            2,  # ||  jmp        2             -. #
    0x29, 0361,          # |'- sub    %esi , %ecx        | #
    0x8a, 0024,0260,     # |.- movb  (%eax,%esi,4),%dl  -' #
    0x88, 0020,          # ||  movb    %dl ,(%eax)         #
    0x01, 0330,          # ||  add    %ebx , %eax          #
    0xe2,           -9,  # |'- loop      -9                #
    0xc2,         16,0,  # '-- ret      $16                #
  ].pack('C*')
  def simple_aberration! offset = 5, phase = 2 # Red [B,G,R,A][phase]
    callproc SIMPLEABERRATION, addr, width * height, offset, phase % 4
    self
  end
end
```

小朋友们学会了吗 ;)

[1]: https://msdn.microsoft.com/en-us/library/windows/desktop/ms633571(v=vs.85).aspx
[2]: https://hyrious.github.io/s-/quickref.html
[3]: http://www.jianshu.com/p/a8dd4b8af0be
[4]: https://zhuanlan.zhihu.com/p/30123130
