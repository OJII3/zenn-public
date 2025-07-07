---
title: "Macの競プロ環境構築バトル with flake.nix"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [nix, macos, atcoder]
published: true
---

# はじめに

よくある、MacOS で `#include <bits/stdc++.h>` を使う方法 (ただし flake.nix で) となります。

nix-darwin で管理された Mac という若干変わった環境なので、今流行りのAIくんには荷が重かったようです...

:::message
Nix への風評被害防止のために書いておくと、環境の再現性を求めないのであれば、こんなことにこだわる必要はありません。つまり、こんなことに苦しんでいる(ように見える)根本的な理由は単なるこだわりであって、Nix はその解決手段なのです。
:::

## GCC を使おうとしてみるが、Clangd がダメ

以下のような `flake.nix` を書けば `#include <bits/stdc++.h>` が含まれた C++ ファイルをコンパイルできるシェルができます。

```nix
{
  description = "A procon flake.nix";

  inputs = {
    nixpkgs.url = "github:nixos/nixpkgs?ref=nixpkgs-unstable";
    flake-parts.url = "github:hercules-ci/flake-parts";
    systems.url = "github:nix-systems/default";
  };

  outputs =
    inputs@{ self, systems, nixpkgs, flake-parts, ... }:
    flake-parts.lib.mkFlake { inherit inputs; } {
      systems = import inputs.systems;
      perSystem = { config, pkgs, system, ... }:
        {
          devShells.default = pkgs.mkShell {
            packages = with pkgs; [
              gcc
              clang-tools # clangd用
            ];
          };
        };
    };
}

```

しかしながら、`clangd` がこれを認識してくれないため、エディタ(Neovim)上でエラーが出てしまいました。(この時点では `<iostream>` 等すら認識しませんでした。)

## Clangd に IncludePath を設定してみる

`.clangd` ファイル等で IncludePath を設定してみたりもしましたが、`--enable-config` オプションを付けて起動しても `.clangd` ファイルを読んでくれていないようでした。

これは未だ原因不明で、LSPの設定の問題なのかClangdの設定の問題なのか三月なのか...

## Clang の IncludePath に `bits/stdc++.h` を追加する

`clang-tools` (`clang`) が認識している場所の中に `bits/stdc++.h` を配置してあげる方針です。

「兢プロ Mac bits」などと調べて出てくる方法のうち、手動で `/usr/local/include/bits` に `stdc++.h` を作る方法と似たようなことを `flake.nix` に記述します。

[clang-tools のパッケージの定義](https://github.com/NixOS/nixpkgs/blob/nixos-unstable/pkgs/development/compilers/llvm/common/clang-tools/default.nix#L56)を見に行くと、

```nix
  # enableLibcxx will use the c++ headers from clang instead of gcc.
  # This shouldn't have any effect on platforms that use clang as the default compiler already.
```

と書かれていますので、C++ の std を使うにはそもそも `enableLibcxx = true` を設定する必要があったようです。これを踏まえて、以下のような packageOverride を `flake.nix` に追加します。

```nix
              (clang-tools.overrideAttrs (old: {
                enableLibcxx = true;
                postPatch = ''
                  mkdir -p $out/include/bits/stdc++.h.gch
                  cat > $out/include/bits/stdc++.h << 'EOF'
                  #pragma once

                  #include <algorithm>
                  #include <array>
                  #include <bitset>
                  #include <cassert>
                  #include <cctype>
                  #include <cerrno>
                  #include <cfenv>
                  #include <cfloat>
                  #include <chrono>
                  #include <cinttypes>
                  #include <climits>
                  #include <clocale>
                  #include <cmath>
                  #include <complex>
                  #include <csetjmp>
                  #include <csignal>
                  #include <cstdarg>
                  #include <cstddef>
                  #include <cstdint>
                  #include <cstdio>
                  #include <cstdlib>
                  #include <cstring>
                  #include <ctime>
                  #include <cwchar>
                  #include <cwctype>
                  #include <deque>
                  #include <exception>
                  #include <fstream>
                  #include <functional>
                  #include <iomanip>
                  #include <ios>
                  #include <iosfwd>
                  #include <iostream>
                  #include <istream>
                  #include <iterator>
                  #include <limits>
                  #include <list>
                  #include <locale>
                  #include <map>
                  #include <memory>
                  #include <new>
                  #include <numeric>
                  #include <ostream>
                  #include <queue>
                  #include <random>
                  #include <regex>
                  #include <set>
                  #include <sstream>
                  #include <stack>
                  #include <stdexcept>
                  #include <streambuf>
                  #include <string>
                  #include <tuple>
                  #include <type_traits>
                  #include <typeinfo>
                  #include <unordered_map>
                  #include <unordered_set>
                  #include <utility>
                  #include <valarray>
                  #include <vector>

                  using namespace std;
                  EOF
                '';
              }))
```

ちゃんと認識されました。

`flake.nix` の全文は [こちら](https://github.com/OJII3/procon/blob/main/flake.nix) にあります。兢プロは全然やってなくて恥ずかしいので中は見ないでね。

## おわりに

若干泥臭い気もしますが、グローバルな設定やパスに依存しない、このリポジトリだけで完結する環境が完成しました。

Nix まだまだ理解できてないので、ご指摘お待ちしております。兢プロはぼちぼち頑張っていこうと思います。
