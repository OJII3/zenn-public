---
title: "C++で並列処理を試す"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [c++, ros]
published: false
---

# C++で並列処理をしたくなった

`std::thread`か`std::async`を使えばいいっぽい。

## 例えばこんなクラス

```cpp loop.hop
class Loop {
 public:
  void StartLoop();
  void StopLoop();
 private:
  bool is_running_ = false;
};
```
```cpp loop.cpp
#include "loop.hpp"

Loop::StartLoop() {
  is_running = true
  while(is_running) {
    std::cout << "Hello" << std::endl;
  }
}

Loop::StopLoop() {
  is_running = false;
}
```

このループを他から呼びつつ、別の処理をしたい

## ふつうに書く

```cpp main.cpp
#include "loop.hpp"

int main() {
  Loop loop;
  loop.StartLoop(); // ブロッキング発生！無限ループとなり、以降の処理は実行されない
  std::this_thread::sleepfor(std::chrono::seconds(5));
  loop.StopLoop();
}
```

これじゃ困るのでどうにかする。

## 並列処理

```cpp main.cpp
#include "loop.hpp"

int main() {
  auto loop = std::make_shared<Loop>();
  std::thread([&]() {
   loop->StartLoop();
  }).detach(); // fire and forget 的に非同期処理を実行
  std::this_thread::sleep_for(std::chrono::seconds(5));
  loop->StopLoop();
}
```

とりあえず目標は達成。

## 例外処理

別スレッドでエラーが起きた時に、メインスレッドでその処理をしたい。








