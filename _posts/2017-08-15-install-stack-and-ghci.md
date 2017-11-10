---
title: Install Stack and GHCi
---
    OS: Windows 10
    stack --version
    Version 1.4.0, Git revision e714f1dd3fade19496d91bd6a017e435a96a6bcd x86_64 hpack-0.17.0
    stack ghci
    GHCi, version 8.0.2

### Steps

1. Grab [stack.exe][1], put it in your PATH.
2. Edit `%STACK_ROOT%\config.yaml`, add [Tsinghua’s package-indices][2].
   If `%STACK_ROOT%` not exists, this file is placed at `%AppData%\stack\config.yaml`.
3. Run `stack setup` **without administrative privilege**, or you could try setup again :/
4. `stack ghci`.

#### Tsinghua’s package-indices

<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/9.12.0/languages/yaml.min.js"></script>

```yaml
package-indices:
  - name: Tsinghua
    download-prefix: https://mirrors.tuna.tsinghua.edu.cn/hackage/package/
    http: https://mirrors.tuna.tsinghua.edu.cn/hackage/00-index.tar.gz
setup-info: "http://mirrors.tuna.tsinghua.edu.cn/stackage/stack-setup.yaml"
urls:
  latest-snapshot: http://mirrors.tuna.tsinghua.edu.cn/stackage/snapshots.json
  lts-build-plans: http://mirrors.tuna.tsinghua.edu.cn/stackage/lts-haskell/
  nightly-build-plans: http://mirrors.tuna.tsinghua.edu.cn/stackage/stackage-nightly/
```

[1]: https://docs.haskellstack.org/en/stable/install_and_upgrade/#manual-download
[2]: https://zhuanlan.zhihu.com/p/25005809