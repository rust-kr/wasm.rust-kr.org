# 디버깅

코드를 더 작성하기 전에, 디버깅 툴들을 조금 살펴보고 장전해 볼까요? 관심이 있다면 [러스트로 웹어셈블리 바이너리를 생성하는 데 사용해 볼 수 있는 접근 방법들과 툴들에 대해 다루는 참조 페이지][reference-debugging]도 확인해 보세요!

[reference-debugging]: ../reference/debugging.html

## 패닉 로그 활성화하기

[코드가 패닉 할 때 개발자 콘솔에 도움이 되는 에러 메세지를 표시하면 디버깅이 더 쉬워집니다.](../reference/debugging.html#logging-panics)

필수로 사용할 필요는 없지만 `wasm-pack-template`은 `wasm-game-of-life/src/utils.rs` 파일에 기본으로 활성화돼 있는 종속성인 [the `console_error_panic_hook` 크레이트][panic-hook]를 포함합니다. 이 훅(hook)을 생성자에 포함시켜주면 이 기능을 사용할 수 있게 됩니다. `wasm-game-of-life/src/lib.rs` 파일 내의 `Universe::new` 생성자에서 불러보도록 하겠습니다.

```rust
pub fn new() -> Universe {
    utils::set_panic_hook();

    // ...
}
```

[panic-hook]: https://github.com/rustwasm/console_error_panic_hook

## 구현한 Game of Life에 로그 기능 추가하기

[`web-sys` 크레이트를 통해 `console.log` 함수를 사용해 보고, 매 세포들의 정보를 로그][logging] 할 수 있도록 기능을 추가해 보겠습니다.

우선, `wasm-game-of-life/Cargo.toml` 파일을 열고 `web-sys`를 종속성으로 추가한 다음 `"console"` 기능을 활성화해 주세요:

```toml
[dependencies]

# ...

[dependencies.web-sys]
version = "0.3"
features = [
  "console",
]
```

더 나은 개발자 경험을 위해 `console.log` 함수를 `println!` 스타일의 매크로로 감싸보겠습니다:

[logging]: ../reference/debugging.html#logging-with-the-console-apis

```rust
extern crate web_sys;

// `println!(...)` 스타일의 문법을 `console.log`에 사용할 수 있게 해주는 매크로.
macro_rules! log {
    ( $( $t:tt )* ) => {
        web_sys::console::log_1(&format!( $( $t )* ).into());
    }
}
```

이제 러스트 코드에서 `log` 매크로를 호출해서 콘솔에 메세지를 로그 해봅시다. 예를 들어서, 세포의 상태, 이웃 수, 다음 틱 상태를 로그 할 수 있도록 `wasm-game-of-life/src/lib.rs` 파일을 수정해 주세요:

```diff
diff --git a/src/lib.rs b/src/lib.rs
index f757641..a30e107 100755
--- a/src/lib.rs
+++ b/src/lib.rs
@@ -123,6 +122,14 @@ impl Universe {
                 let cell = self.cells[idx];
                 let live_neighbors = self.live_neighbor_count(row, col);

+                log!(
+                    "cell[{}, {}] is initially {:?} and has {} live neighbors",
+                    row,
+                    col,
+                    cell,
+                    live_neighbors
+                );
+
                 let next_cell = match (cell, live_neighbors) {
                     // 규칙 1: 인구 부족으로 2개 미만의 이웃을 가진 세포는 죽게 됩니다. 
@@ -140,6 +147,8 @@ impl Universe {
                     (otherwise, _) => otherwise,
                 };

+                log!("    it becomes {:?}", next_cell);
+
                 next[idx] = next_cell;
             }
         }
```

## 디버거를 사용하여 매 틱마다 일시정지 시키기

[브라우저의 스탭핑 디버거 (stepping debugger) 는 Rust로 작성한 웹어셈블리와 상호작용하는 자바스크립트 코드를 살펴볼 때 유용합니다.](../reference/debugging.html#using-a-debugger)

예를 들어서, `universe.tick()` 호출 이전에 [자바스크립트 코드에 `debugger;` 줄][dbg-stmt]을 추가하면 디버깅 툴을 사용하여 `renderLoop`의 매 순회마다 코드 실행을 일시정지시킬수 있게 됩니다.

```js
const renderLoop = () => {
  debugger;
  universe.tick();

  drawGrid();
  drawCells();

  requestAnimationFrame(renderLoop);
};
```

이제 로그 메세지를 쉽게 살펴볼 수 있도록 간편한 체크포인트(checkpoint) 기능을 사용할 수 있게 됐습니다. 이 기능을 통해 현재 렌더된 프레임과 이전 프레임을 비교할 수 있습니다.

[dbg-stmt]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/debugger

[![Game of Life 디버깅 화면 스크린샷](../images/game-of-life/debugging.png)](../images/game-of-life/debugging.png)

## 연습해 보기

* 죽게 되거나 살아나게 되는 식으로 상태가 전환되는 각각 세포의 행과 열을 기록할 수 있도록 `tick` 함수를 로그 하는 코드를 추가해 보세요.

* `Universe::new` 메소드에 `panic!()` 매크로를 추가해서 웹 브라우저의 자바스크립트 디버깅 툴에서 패닉한 코드를 [퇴각검색(backtrace)](https://ko.wikipedia.org/wiki/퇴각검색) 할 수 있도록 수정해 보세요. 추가로 debug 심볼을 비활성화한 다음 `console_error_panic_hook` 선택적 종속성을 다시 빌드했을 때 스택 추적도 다시 확인해 보세요. 그렇게 유용하진 않은 것 같은데, 안 그런가요?