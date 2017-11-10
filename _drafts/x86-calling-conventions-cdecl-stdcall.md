---
title: x86 调用约定
---

- [Wiki x86 调用约定](https://en.wikipedia.org/wiki/X86_calling_conventions)
- [论函数调用约定](http://blog.csdn.net/fly2k5/article/details/544112)

<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/9.12.0/languages/x86asm.min.js"></script>

写汇编/机器码的时候，一个常用套路是

```x86asm
func:
  push %ebp           # prolog
  mov  %esp,%ebp      #
  mov  8(%ebp) ,%eax
  add  12(%ebp),%eax
  leave               # epsilog
  ret                 # 
main:
  push $5             # %esp -= 4
  push $3             # %esp -= 4
  call func
  add  $8,%esp        # restore %esp
```

#### cdecl

上面这种 `push` 参数然后 `add esp` 的做法叫做 `cdecl` (C declaration)，特点是调用者自己维护/恢复堆栈，子程不会修改栈顶位置。习惯上，我们在一个子程中只操作 `ebp` 以上的部分。

关于参数的地址，`4(%ebp)` 是当前函数（`cs:eip`），`8(%ebp)` 是第一个参数，`12(%ebp)` 是第二个参数，以此类推。参数压入（`push`）时要从右向左，这样我们取的时候就是从左向右。

`printf` 这种不定长参数的函数，只要知道第一个参数，就可以算出后续参数的个数，这也是不定长参数实现的姿势之一。

#### stdcall

`stdcall` 与之相比只有一处不同，即最后一句写作

```x86asm
ret $n                # n = 参数个数 * 4
```

同时调用方不用再 `add esp` 了，栈顶位置已经被自动改回去了。

#### 局部变量

虽然大多数情况下寄存器已经够用，但是如果我要开几个内存位置当局部变量呢？

有一个指令就是干这事的，[`enter imm8,imm16`](http://www.felixcloutier.com/x86/ENTER.html)，注意写了 `enter` 之后 prolog 部分就不用再写了。

还有一个很常用的做法是，手动 `sub ebp`，其实就是不给参数的 `push`（实际上没有不给参数的写法，所以我们泄露一下）。另外，prolog 部分也就是不给参数的 `enter`。

#### 番外：口算最短机器码的 A+B

第一步：去掉 prolog，epsilog

```x86asm
mov  4(%esp),%eax
add  8(%esp),%eax
ret
```

接下来我们将其翻译为机器码

首先 `4(%esp)` 显然是一个 `[base + disp8]`（1）寻址，目标寄存器是 `%eax` 即 `0`，SIB 部分是一个 `%esp` 即 `X * esp(eiz=0) + esp` 即 `0X44`（`X` 随便填 0 1 2 3），`disp8` 部分是一个 `4`

与之对应的 `mov` Opcode 是 `8B /r`

    8B 0104 0044 04

`add` 句的 ModR/M 部分和 SIB 部分与上面相似，只要找到对应的 Opcode 是 `03 /r`

    03 0104 0044 08

`ret` 是 `c3`

现在我们有了一条

    8B 0104 0044 04 03 0104 0044 08 c3

这就是最短（仅仅 9 字节）的 A+B 了（被打死

#### 利用 union 在 C 语言中直接执行机器码

```C
union {
  char* str;
  void* (*func)();
} mc;

int main() {
  char code[] = "\x8b\104\044\x04\3\104\044\x08\xc3\0";
  mc.str = code;
  printf("%d", (int) mc.func(3, 5));
  return 0;
}
```

注意使用 gcc 编译的时候加上参数 `-m32` 来确保 32 位机器码能够正常运行。

你们自己体会一下（再次被打死

#### 利用 CallWindowProc 在 RGSS 等环境中直接执行机器码

```ruby
# encoding: ASCII-8BIT
require 'win32api'

CWP = Win32API.new('user32', 'CallWindowProc', 'piiii', 'i')

def cp code, a = 0, b = 0, c = 0, d = 0
  CWP.call code, a, b, c, d
end

p cp "\x8b\104\044\x04\3\104\044\x08\xc3\0", 3, 5
```

注意 32 位机器码仅可在 32 位 Ruby（RGSS）下运行，首行的 `# encoding: ASCII-8BIT`
是为了直接写字符串字面值，如果不能保证 encoding 请使用 `[0x8b, 0104].pack('C*')` 的形式。

CallWindowProc 的内容被抽象泄露了（逃
