# 타임 프로파일링 하기

이 챕터에서는 타임 프로파일링 작업을 해보면서 전에 구현한 Game of Life의 성능을 개선시켜보겠습니다.

시작하기 전에, [Rust와 WebAssembly 코드에 사용해볼수 있는 타임 프로파일링 툴들](../reference/time-profiling.md)을 살펴보고 익숙해져 보세요.

## `window.performance.now` 함수를 사용하여 초당 프레임 (FPS, Frames Per Second) 타이머 만들기
이 FPS 타이머는 구현한 게임의 렌더링 속도를 어떻게 개선하는지 살펴볼 때 매우 유용하게 사용될 예정입니다.

`wasm-game-of-life/www/index.js` 파일에 `fps` 객체를 추가하는 것으로 시작해봅시다:

```js
const fps = new class {
  constructor() {
    this.fps = document.getElementById("fps");
    this.frames = [];
    this.lastFrameTimeStamp = performance.now();
  }

  render() {
    // 마지막 프레임 렌더부터의 델타 시간을 fps 단위로 변환합니다.
    const now = performance.now();
    const delta = now - this.lastFrameTimeStamp;
    this.lastFrameTimeStamp = now;
    const fps = 1 / delta * 1000;

    // 마지막 100개의 타이밍만 저장합니다.
    this.frames.push(fps);
    if (this.frames.length > 100) {
      this.frames.shift();
    }

    // 최대, 최소 타이밍과 마지막 100개 타이밍의 평균을 찾습니다.
    let min = Infinity;
    let max = -Infinity;
    let sum = 0;
    for (let i = 0; i < this.frames.length; i++) {
      sum += this.frames[i];
      min = Math.min(this.frames[i], min);
      max = Math.max(this.frames[i], max);
    }
    let mean = sum / this.frames.length;

    // 통계를 렌더합니다.
    this.fps.textContent = `
Frames per Second:
         latest = ${Math.round(fps)}
avg of last 100 = ${Math.round(mean)}
min of last 100 = ${Math.round(min)}
max of last 100 = ${Math.round(max)}
`.trim();
  }
};
```

`fps` 객체의 `render` 함수를 매 `renderLoop` 반복마다 불러보겠습니다:

```js
const renderLoop = () => {
    fps.render(); //new

    universe.tick();
    drawGrid();
    drawCells();

    animationId = requestAnimationFrame(renderLoop);
};
```

마지막으로, `fps` 요소를 잊지 말고 `wasm-game-of-life/www/index.html` 파일에 추가해줍시다. `<canvas>` 바로 위에 추가해주세요:

```html
<div id="fps"></div>
```

그리고 CSS 프로퍼티를 추가해서 깔끔하게 포맷해주겠습니다:

```css
#fps {
  white-space: pre;
  font-family: monospace;
}
```

짜잔! 이제 [http://localhost:8080](http://localhost:8080) 페이지를 새로고침 하면 FPS 카운터를 확인할 수 있게 됐습니다!

[perf-now]: https://developer.mozilla.org/en-US/docs/Web/API/Performance/now

### `Universe::tick`과 `console.time`, `console.timeEnd` 시간 측정하기

각 `Universe::tick`이 호출되는데 시간이 얼마나 걸리는지 확인해볼려면 `web-sys` 크레이트의 `console.time`, `console.timeend` 를 활용해볼수 있습니다.

먼저, `wasm-game-of-life/Cargo.toml` 파일을 열고 `web-sys` 를 종속성으로 추가해주세요:

```toml
[dependencies.web-sys]
version = "0.3"
features = [
  "console",
]
```

`console.time`를 호출할 때 마다 `console.timeEnd`도 같이 호출될 예정이기 때문에, [RAII][] 타입으로 묶어서 간편하게 사용할수도 있습니다:

```rust
extern crate web_sys;
use web_sys::console;

pub struct Timer<'a> {
    name: &'a str,
}

impl<'a> Timer<'a> {
    pub fn new(name: &'a str) -> Timer<'a> {
        console::time_with_label(name);
        Timer { name }
    }
}

impl<'a> Drop for Timer<'a> {
    fn drop(&mut self) {
        console::time_end_with_label(self.name);
    }
}
```

그 다음에, 메소드 최상단에 이 코드를 추가해서 얼마나 각 `Universe::tick` 호출이 오래 걸리는지 측정해볼수 있습니다:

```rust
let _timer = Timer::new("Universe::tick");
```

콘솔에 `Universe::tick`를 호출하는데 얼마나 오래 걸렸는지 로그를 표시합니다:

[![console.time 로그 스크린샷](../images/game-of-life/console-time.png)](../images/game-of-life/console-time.png)

추가로, 브라우저 프로파일러(profiler)의 timeline 혹은 waterfall 뷰에서 `console.time`과 `console.timeEnd`가 같이 실행된 부분을 확인할 수 있습니다:

[![console.time 로그 스크린샷](../images/game-of-life/console-time-in-profiler.png)](../images/game-of-life/console-time-in-profiler.png)

[RAII]: https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization

## Game of Life 세상의 사이즈 키워보기

> ⚠️ 이 섹션은 FireFox의 스크린샷을 예시로 보여줍니다. 거의 모든 모던 브라우저가 비슷한 기능들을 가지고 있지만, 개발자 도구에 브라우저마다 약간의 차이가 있을 수 있습니다. 추출하게 될 프로파일 정보는 기본적으로 같지만, 보게 될 뷰나 도구 이름 등이 다를수 있는 점을 확인해주세요.

구현한 Game of Life 세상을 더 크게 만들어보면 어떨까요? 64 x 64 사이즈의 세상을 128 x 128 사이즈로 늘려봅시다. (`wasm-game-of-life/src/lib.rs` 파일에서 `Universe::new`를 수정해주세요.) 제 컴퓨터에서는 사이즈를 이렇게 늘리게 되면 부드럽게 작동하던 60fps 화면이 버벅이면서 40fps 처럼 보이는 것 같습니다.

프로파일을 기록하고 waterfall 뷰를 확인하면, 각 애니메이션이 20 밀리초 이상 걸리는 것을 확인할 수 있습니다. 60fps로 작동했을 때 프레임 전체를 렌더하는데 16 밀리초가 걸린 것을 떠올려보면 확실히 차이가 있습니다. 참고로, JavaScript와 WebAssembly외에도 페이지를 그리는 등 브라우저가 수행하는 작업의 영향도 있습니다.

[![페이지 렌더링 처리의 waterfall 뷰 스크린샷](../images/game-of-life/drawCells-before-waterfall.png)](../images/game-of-life/drawCells-before-waterfall.png)

한 애니메이션 프레임동안 어떤 일이 일어나는지 확인해보면, `CanvasRenderingContext2D.fillStyle` setter가 성능 차원에서 많은 비용을 요구하는 것을 확인할 수 있습니다.

> ⚠️ FireFox 브라우저에서 위에서 언급된 `CanvasRenderingContext2D.fillStyle` 대신에 "DOM"이 표시된다면, 성능 개발자 도구 (performance developer tools) 에서 "Gecko 플랫폼 데이터 표시하기 (Show Gecko Platform Data)" 옵션을 활성화해줘야 합니다:
> 
> [!["Gecko 플랫폼 데이터 표시하기 (Show Gecko Platform Data)" 옵션 활성화하기](../images/game-of-life/profiler-firefox-show-gecko-platform.png)](../images/game-of-life/profiler-firefox-show-gecko-platform.png)

[![페이지 렌더링 처리의 flamegraph 뷰 스크린샷](../images/game-of-life/drawCells-before-flamegraph.png)](../images/game-of-life/drawCells-before-flamegraph.png)

많이 표시되는 호출 트리의 집계 (call tree's aggregation) 를 살펴보면 이게 전혀 이상한 동작이 아님을 확인할 수 있습니다:

[![페이지 렌더링 처리의 flamegraph 뷰 스크린샷](../images/game-of-life/drawCells-before-calltree.png)](../images/game-of-life/drawCells-before-calltree.png)

거의 40% 분량을 이 setter에 사용해버렸네요!

> ⚡ `tick` 메소드가 성능 병목을 일으키는데 특별한 이유가 있을 것 같았지만, 사실 그렇지 않은 부분을 확인했습니다. 작업을 하다 보면 예상치 못한 부분에서 시간을 많이 쓰게 될수도 있으니, 항상 **정말 중요한** 프로파일링 도구를 먼저 살펴보도록 합시다.

`wasm-game-of-life/www/index.js` 파일 내의 `drawCells` 함수에서 `fillStyle` 프로퍼티가 한번 정해지고 세상 내의 모든 세포와 모든 애니메이션에 이 프로퍼티가 사용되게 됩니다.

```js
for (let row = 0; row < height; row++) {
  for (let col = 0; col < width; col++) {
    const idx = getIndex(row, col);

    ctx.fillStyle = cells[idx] === DEAD
      ? DEAD_COLOR
      : ALIVE_COLOR;

    ctx.fillRect(
      col * (CELL_SIZE + 1) + 1,
      row * (CELL_SIZE + 1) + 1,
      CELL_SIZE,
      CELL_SIZE
    );
  }
}
```

`fillStyle`가 많은 성능을 요구하는 부분을 확인했는데, 대신에 어떤 식으로 코드를 작성해야 할까요? 세포의 생존 여부에 따라 `fillStyle`를 사용하도록 바꿔봅시다. `fillStyle = ALIVE_COLOR`를 적어줘서 살아있는 세포만 그리게 하고 `fillStyle = DEAD_COLOR`를 적어줘서 죽은 세포도 동일한 방식으로 처리를 해준다면, 세포들을 모두 한번에 그리지 않고 `fillStyle`을 두번만 설정할수 있게 됩니다.

```js
// 살아있는 세포 처리.
ctx.fillStyle = ALIVE_COLOR;
for (let row = 0; row < height; row++) {
  for (let col = 0; col < width; col++) {
    const idx = getIndex(row, col);
    if (cells[idx] !== Cell.Alive) {
      continue;
    }

    ctx.fillRect(
      col * (CELL_SIZE + 1) + 1,
      row * (CELL_SIZE + 1) + 1,
      CELL_SIZE,
      CELL_SIZE
    );
  }
}

// 죽어있는 세포 처리.
ctx.fillStyle = DEAD_COLOR;
for (let row = 0; row < height; row++) {
  for (let col = 0; col < width; col++) {
    const idx = getIndex(row, col);
    if (cells[idx] !== Cell.Dead) {
      continue;
    }

    ctx.fillRect(
      col * (CELL_SIZE + 1) + 1,
      row * (CELL_SIZE + 1) + 1,
      CELL_SIZE,
      CELL_SIZE
    );
  }
}
```

코드 파일을 저장하고 [http://localhost:8080/](http://localhost:8080/) 페이지를 새로고침 해주면, 웹사이트가 60fps로 다시 부드럽게 렌더링을 하는 것을 볼수 있습니다.

프로파일을 다시 확인해보면, 각 애니메이션 프레임마다 오직 10 밀리초만 걸리는 부분을 확인할 수 있습니다.

[![drawCells 함수를 업데이트 한 이후 페이지 렌더링 처리의 waterfall 뷰 스크린샷](../images/game-of-life/drawCells-after-waterfall.png)](../images/game-of-life/drawCells-after-waterfall.png)

한 프레임을 다시 분석해보면, `fillStyle` 이 더이상 성능을 많이 사용하지 않고, `fillRect`가 각 세포 사각형을 그리는데 대부분의 시간을 사용하는 것을 확인할 수 있습니다.

[![drawCells 함수를 업데이트 한 이후 페이지 렌더링 처리의 flamegraph 뷰 스크린샷](../images/game-of-life/drawCells-after-flamegraph.png)](../images/game-of-life/drawCells-after-flamegraph.png)

## Game of Life가 더 빠르게 흘러가도록 만들어보기

어떤 사람들은 빨리 빨리 하는걸 더 좋아해서, 매 프레임마다 1틱이 아니라 9틱씩 넘어가는걸 선호하기도 합니다. `wasm-game-of-life/www/index.js` 파일의 `renderLoop` 함수를 수정해서 생각외로 쉽게 할수 있긴 합니다:

```js
for (let i = 0; i < 9; i++) {
  universe.tick();
}
```

제 컴퓨터에서는 이 코드가 35fps 속도로 다시 느려지는것 같습니다. 좋지 않은 현상이니 다시 60fps 로 만들어보겠습니다!

`Universe::tick` 에서 시간이 많이 소요되는 것으로 보이므로, `console.time`과 `console.timeEnd`를 호출해서 래핑(wrapping)할수 있도록 `Timer`를 추가해보고 어떻게 되는지 살펴보겠습니다. 제 가설대로라면 매 틱마다 세포의 새 벡터를 할당하고 기존 벡터를 해제(freeing)하는 작업은 성능을 많이 사용하고 시간도 많이 소비하게 됩니다.

```rust
pub fn tick(&mut self) {
    let _timer = Timer::new("Universe::tick");

    let mut next = {
        let _timer = Timer::new("다음 세포들을 할당합니다.");
        self.cells.clone()
    };

    {
        let _timer = Timer::new("다음 세대를 처리합니다.");
        for row in 0..self.height {
            for col in 0..self.width {
                let idx = self.get_index(row, col);
                let cell = self.cells[idx];
                let live_neighbors = self.live_neighbor_count(row, col);

                let next_cell = match (cell, live_neighbors) {
                    // 규칙 1: 인구 부족으로 2개 미만의 이웃을 가진 세포는 죽게 됩니다.
                    (Cell::Alive, x) if x < 2 => Cell::Dead,
                    // 규칙 2: 2개 혹은 3개의 이웃을 가진 세포는 다음 세대에서 계속 살아있습니다.
                    (Cell::Alive, 2) | (Cell::Alive, 3) => Cell::Alive,
                    // 규칙 3: 과잉 인구로 3개 초과의 이웃을 가진 모든 세포는 죽게 됩니다.
                    (Cell::Alive, x) if x > 3 => Cell::Dead,
                    // 규칙 4: 세포 증식으로 정확히 3개의 이웃을 가진 세포는 살아나게 됩니다.
                    (Cell::Dead, 3) => Cell::Alive,
                    // 규칙이 적용되지 않는 세포들의 상태는 그대로 유지되게 됩니다.
                    (otherwise, _) => otherwise,
                };

                next[idx] = next_cell;
            }
        }
    }

    let _timer = Timer::new("기존 세포들을 해제합니다.");
    self.cells = next;
}
```

브라우저 개발자 도구에서 타이밍(timing)을 확인해보면 이 가설이 사실은 명백하게 틀린 것을 알수 있습니다. 실제로는, 대부분의 시간이 다음 세대 세포들을 계산하는데 사용되게 됩니다. 그리고 매 틱마다 벡터 값을 할당하고 해제하는 작업이 놀랍게도 그렇게 성능이 많이 들지 않습니다. 다시 한번 **정말 중요한** 프로파일링을 해보겠습니다!

[![Universe::tick 타이머 결과 값의 스크린샷](../images/game-of-life/console-time-in-universe-tick.png)](../images/game-of-life/console-time-in-universe-tick.png)

다음 섹션에서는 `nightly` 컴파일러가 필요합니다. 필요한 [테스트 기능 게이트(test feature gate)](https://doc.rust-lang.org/unstable-book/library-features/test.html)가 `nightly` 버전에 포함돼 있어서 그런데, 벤치마킹에 이 기능을 사용해보겠습니다. [cargo benchcmp][benchcmp]이라는 툴도 필요하니 설치해주도록 합시다. `cargo bench`로 생성한 마이크로 벤치마크들(micro-benchmarks) 을 비교하는데 사용하는 작은 사이즈의 유틸리티입니다.

[benchcmp]: https://github.com/BurntSushi/cargo-benchcmp

WebAssembly랑 동일한 작업을 수행하는 네이티브 코드인 `#[bench]`를 작성해봅시다. 이렇게 작성하는 대신에 더 많은 기능을 사용할수 있게 됩니다. 이제 새롭게 작성된 `wasm-game-of-life/benches/bench.rs`을 확인해주세요:

```rust
#![feature(test)]

extern crate test;
extern crate wasm_game_of_life;

#[bench]
fn universe_ticks(b: &mut test::Bencher) {
    let mut universe = wasm_game_of_life::Universe::new();

    b.iter(|| {
        universe.tick();
    });
}
```

그리고 `#[wasm_bindgen]` 속성을 모두 주석처리 해주도록 하고, `Cargo.toml` 파일의 `"cdylib"`도 주석처리 해주겠습니다. 이렇게 하지 않으면 빌드가 실패하고 링크 오류(link errors)가 발생하게 됩니다.

준비가 다 됐다면, `cargo bench | tee before.txt` 명령어를 실행해서 코드를 컴파일하고 벤치마크를 실행해봅시다. `| tee before.txt`를 포함하면서 `cargo bench` 명령어의 출력값을 가져와서 `before.txt` 파일에 저장하게 되니 이 부분도 확인해주세요.

```
$ cargo bench | tee before.txt
    Finished release [optimized + debuginfo] target(s) in 0.0 secs
     Running target/release/deps/wasm_game_of_life-91574dfbe2b5a124

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/release/deps/bench-8474091a05cfa2d9

running 1 test
test universe_ticks ... bench:     664,421 ns/iter (+/- 51,926)

test result: ok. 0 passed; 0 failed; 0 ignored; 1 measured; 0 filtered out
```

바이너리 파일의 위치도 같이 표시되는 부분을 확인할 수 있습니다. 이번에는 사용하는 운영체제(operating system)의 프로파일러를 사용하여 벤치마킹을 다시 해보겠습니다. 제 컴퓨터는 리눅스(Linux)를 사용하고 있으니 [`perf`][perf]를 예제로 사용해보겠습니다:

[perf]: https://perf.wiki.kernel.org/index.php/Main_Page

```
$ perf record -g target/release/deps/bench-8474091a05cfa2d9 --bench
running 1 test
test universe_ticks ... bench:     635,061 ns/iter (+/- 38,764)

test result: ok. 0 passed; 0 failed; 0 ignored; 1 measured; 0 filtered out

[ perf record: Woken up 1 times to write data ]
[ perf record: Captured and wrote 0.178 MB perf.data (2349 samples) ]
```

`perf report` 명령어로 프로파일을 로드하면, 예상한 대로 대부분의 시간이 `Universe::tick`를 실행하는데 소비되는 것을 볼수 있습니다:

[![perf report 스크린샷](../images/game-of-life/bench-perf-report.png)](../images/game-of-life/bench-perf-report.png)

`perf` will annotate which instructions in a function time is being spent at if
you press `a`:
`a` 키를 누르면 함수를 실행하는데 어떤 어셈블리(Assembly) 명령어(instruction)를 실행하고 있는지 표시되는 것을 볼 
수 있습니다:

[![perf 화면에서 어셈블리 명령어를 표시하는 화면의 스크린샷](../images/game-of-life/bench-perf-annotate.png)](../images/game-of-life/bench-perf-annotate.png)

위 내용을 확인하면 26.67%의 시간이 이웃 세포들을 만들어내는 데 사용되고, 23.41%를 이웃들의 열 인덱스, 그리고 나머지 15.42%를 행 인덱스를 구하는데 사용되는 것을 알수 있습니다. 제일 많은 성능을 사용하는 세 명령어들 중, 두번째와 세번째가 비용이 많이 드는 `div` 명령어인 부분도 볼수 있습니다. 이 `div`들이 `Universe::live_neighbor_count` 함수에서 나머지 연산자로 인덱싱 하도록 구현했던 부분입니다. (modulo indexing logic)

`wasm-game-of-life/src/lib.rs` 파일에서 `live_neighbor_count`를 정의했던 내용을 다시 떠올려봅시다:

```rust
fn live_neighbor_count(&self, row: u32, column: u32) -> u8 {
    let mut count = 0;
    for delta_row in [self.height - 1, 0, 1].iter().cloned() {
        for delta_col in [self.width - 1, 0, 1].iter().cloned() {
            if delta_row == 0 && delta_col == 0 {
                continue;
            }

            let neighbor_row = (row + delta_row) % self.height;
            let neighbor_col = (column + delta_col) % self.width;
            let idx = self.get_index(neighbor_row, neighbor_col);
            count += self.cells[idx] as u8;
        }
    }
    count
}
```

코드를 확인해보면 `if` 블럭들로 첫째와 마지막 행과 열의 엣지 케이스(edge case) 를 처리하는데 코드를 불필요하게 길게 쓰지 않도록 나머지 연산자 (modulo operator) 를 사용하고 있는 부분을 확인할 수 있습니다. `row`과 `column` 둘 다 세상의 끝에 있지 않고 나머지 연산자를 쓸 필요가 없는 일반적인 케이스를 처리하는데도 `div` 명령어를 사용해서 많은 성능을 사용하고 있습니다. 반복문을 사용하는 대신에 `if` 문으로 이 엣지 케이스들을 처리해서 여러 분기들을 CPU의 분기 예측기(branch predictor)가 예측하기 쉽도록 만들어보겠습니다.

`live_neighbor_count`를 다음과 같이 다시 작성해봅시다:

```rust
fn live_neighbor_count(&self, row: u32, column: u32) -> u8 {
    let mut count = 0;

    let north = if row == 0 {
        self.height - 1
    } else {
        row - 1
    };

    let south = if row == self.height - 1 {
        0
    } else {
        row + 1
    };

    let west = if column == 0 {
        self.width - 1
    } else {
        column - 1
    };

    let east = if column == self.width - 1 {
        0
    } else {
        column + 1
    };

    let nw = self.get_index(north, west);
    count += self.cells[nw] as u8;

    let n = self.get_index(north, column);
    count += self.cells[n] as u8;

    let ne = self.get_index(north, east);
    count += self.cells[ne] as u8;

    let w = self.get_index(row, west);
    count += self.cells[w] as u8;

    let e = self.get_index(row, east);
    count += self.cells[e] as u8;

    let sw = self.get_index(south, west);
    count += self.cells[sw] as u8;

    let s = self.get_index(south, column);
    count += self.cells[s] as u8;

    let se = self.get_index(south, east);
    count += self.cells[se] as u8;

    count
}
```

벤치마킹을 다시 실행해봅시다! 이번에는 출력되는 메세지들을 `after.txt`로 출력해보겠습니다.

```
$ cargo bench | tee after.txt
   Compiling wasm_game_of_life v0.1.0 (file:///home/fitzgen/wasm_game_of_life)
    Finished release [optimized + debuginfo] target(s) in 0.82 secs
     Running target/release/deps/wasm_game_of_life-91574dfbe2b5a124

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/release/deps/bench-8474091a05cfa2d9

running 1 test
test universe_ticks ... bench:      87,258 ns/iter (+/- 14,632)

test result: ok. 0 passed; 0 failed; 0 ignored; 1 measured; 0 filtered out
```

훨씬 나은것 같습니다! `benchcmp` 툴과 방금 생성한 두 텍스트 파일을 확인해보면 얼마나 나이졌는지 볼수 있습니다:

```
$ cargo benchcmp before.txt after.txt
 name            before.txt ns/iter  after.txt ns/iter  diff ns/iter   diff %  speedup
 universe_ticks  664,421             87,258                 -577,163  -86.87%   x 7.61
```

우와! 7.61 배나 빨라졌네요!

WebAssembly는 일반적인 하드웨어 아키텍쳐(architecture)와 밀접하게 매핑(mapping)돼 있지만, 이러한 네이티브 코드의 속도 향상이 WebAssembly 성능 향상으로도 전환될 수 있는지 확인해봐야 합니다.

`wasm-pack build` 명령어를 실행해서 `.wasm` 파일을 다시 빌드하고 [http://localhost:8080/](http://localhost:8080/) 페이지를 새로고침 해주세요. 제 컴퓨터에서는 60 fps의 속도로 다시 작동하고 있는것 같습니다. 브라우저의 프로파일러를 다시 사용해보니 각 애니메이션 프레임이 10 밀리초씩 걸리는 것으로 보입니다.

성공적으로 잘 마무리한것 같습니다!

[![나머지 연산자를 if문 분기들로 바꾼 이후 페이지 렌더링 처리의 waterfall 뷰 스크린샷](../images/game-of-life/drawCells-after-flamegraph.png)](../images/game-of-life/drawCells-after-flamegraph.png)

## 연습해보기

* 현재로는 할당과 해제를 처리하는 코드를 지우는 게 `Universe::tick` 속도를 향상시키는 가장 쉬운 방법입니다. `Universe`가 두 벡터만 관리하도록 하고, 두 벡터를 코드가 실행되는 내내 해제하거나 `tick` 함수에서 새 버퍼를 할당하지 않도록 세포들을 이중 버퍼링 (double buffering) 으로 구현해보세요.

* "Game of Life 구현하기" 챕터에서 델타 기반으로 설계한 내용을 Rust 코드가 상태가 바뀐 세포들의 목록을 반환하도록 다시 설계해보세요. `<canvas>` 가 더 빨리 렌더되는 게 보이시나요? 매 틱마다 새 델타 목록을 할당하지 않으면서 구현하실수 있는 것 같으신가요?

* 프로파일링 툴로 확인할 수 있듯, 2D `<canvas>` 렌더링이 특별히 빠르지는 않습니다. 2D 캔버스 렌더러 대신 [WebGL][webgl] 렌더러를 사용해서 구현해보세요. WebGL로 구현한 버전은 얼마나 더 빠른가요? 답답했던 WebGL가 비교하면 얼마나 세상을 더 크게 만들수 있게 됐나요?

[webgl]: https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API
