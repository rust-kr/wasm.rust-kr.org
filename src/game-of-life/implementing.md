# Conway's Game of Life 구현하기

## 설계

시작하기 전에, 어떤 방식으로 Game of Lifef를 설계 할지 살펴봅시다.

### 무한한 세상

Game of Life는 무한한 세상에서 시작됩니다. 하지만 보통은 우리가 무한한 메모리와 컴퓨터 파워를 가지고 있지 않기 때문에 다음 세 가지 방법 중 한 방법을 통해 이 귀찮은 제한을 우회하게 됩니다:

1. 세상의 어떤 부분이 많은 컴퓨터 자원을 필요로 하는지 추적하고 이러한 부분을 필요할 때 확장합니다. 최악의 경우에는, 이 확장이 제한 없이 진행되고 코드가 계속해서 느려지면서 결국에는 메모리를 다 차지하게 됩니다.

2. 모서리에 위치한 세포들이 가운데에 위치한 세포들과 비교해서 더 적은 이웃을 가지게 되는 사이즈가 정해져 있는 세상을 만듭니다. 이 방법에는 [gliders](https://conwaylife.com/wiki/Glider)와 같은 무한한 패턴이 모서리에서 끝나버리게 된다는 단점이 있습니다.

3. 사이즈가 정해졌지만 계속해서 연결되는 우주를 만듭니다. 세상의 끝을 반대쪽 세상의 끝으로 연결시켜 세포들이 계속해서 이웃을 가질수 있게 합니다. 이렇게 gliders 패턴이 계속 움직일수 있게 됩니다.

그러면 세 번째 방법으로 구현을 해보겠습니다.

### Rust와 JavaScript 코드끼리 연결하기

> ⚡ 다음 내용은 이 튜토리얼에서 다루는 내용 중에서도 아주 중요한 내용입니다. 이 내용을 이해하면서 얻어갈수 있는 부분이 아주 많습니다!

JavaScript는 `Object`, `Array` 그리고 [DOM 노드(node)](https://developer.mozilla.org/ko/docs/Glossary/Node/DOM)들이 할당되는 가비지 콜렉터가 관리하는 힙을 사용하지만, 작성하게 될 Rust 코드의 선형 메모리는 별개의 공간을 사용하게 됩니다. WebAssembly는 현재로써는 가비지 콜렉터가 관리하는 힙에 직접 접근할수 없습니다. (2018년 4월 기준으로, ["인터페이스 타입" 제안][interface-types] 과 함께 변경될 전망이긴 합니다.) 반면에 JavaScript는 [`ArrayBuffer`][array-buf]나 스칼라 값 (scalar values / `u8`, `i32`, `f64`, 등...) 만으로라도 이 선형 메모리를 읽고 쓸수 있습니다. 이런 내용을 기반으로 모든 WebAssembly와 JavaScript 사이의 커뮤니케이션이 구성되게 됩니다.

[interface-types]: https://github.com/WebAssembly/interface-types/blob/master/proposals/interface-types/Explainer.md
[array-buf]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer

`wasm_bindgen`는 이 경계를 사이로 어떻게 구조체 (compound structure) 들을 주고받아야 하는지 정해주는 역할을 합니다. 이러한 작업은 Rust 구조체를 박싱(boxing)하고, 쉽게 사용하기 위해 JavaScript 클래스에 포인터를 랩핑(wrapping)하고, Rust 코드에서 JavaScript 객체 테이블을 인덱싱(indexing)하는 과정을 포함합니다. `wasm_bindgen`은 매우 간편하지만, 데이터 표현 설계를 모두 대신 해주진 않습니다. 원하는 방식으로 인터페이스 설계를 구현할 수 있도록 도와주는 도구 정도로 생각하면 좋습니다.

WebAssembly와 JavaScript 사이의 인터페이스를 설계할 때, 다음 내용들을 최적화 작업 시 고려해야 합니다:

1. **JavaScript와 WebAssembly 선형 메모리 사이를 오가는 복사(copy) 최소화하기.**
   불필요한 복사는 불필요한 오버헤드를 발생시킵니다.

2. **직렬화 (serializing)와 역직렬화(deserializing) 최소화하기.** 복사와 마찬가지로, 직렬화와 역직렬화도 오버헤드를 발생시킬수 있고, 이러한 작업이 복사도 자주 발생시키게 됩니다. 한 곳에서 모든 직렬화 작업을 하는 대신 일반적으로 WebAssembly 선형 메모리의 알려진 위치로 [opaque handle](https://en.wikipedia.org/wiki/Opaque_data_type)들을 넘기는 방식으로 많은 오버헤드를 줄일수 있게 됩니다. 그리고 `wasm_bindgen`을 통해 JavaScript의 `Object`나 박싱된 Rust `struct`를 가리키는 opaque handle들을 더 쉽게 정의하고 사용할 수 있습니다.

대부분의 경우에는, JavaScript와 WebAssembly를 오갈 때 사이즈가 크고 오래 살아있어야 하는 자료 구조를 WebAssembly 선형 메모리에 두고, 이러한 값들을 JavaScript에서 opaque handle로써 노출 시키는 것이 좋은 인터페이스 설계입니다. JavaScript가 이러한 opaque handle를 통해 WebAssembly 함수를 호출하고, 데이터를 변형시키고, 무거운 컴퓨팅 작업을 하고, 값을 검색하고, 최종적으로 작은 사이즈의 복사할수 있는 값을 반환하게 됩니다. 작은 값만 반환하게 되면 JavaScript 가비지 콜렉터가 관리하는 힙과 WebAssembly 선형 메모리 사이의 모든 값들을 앞뒤로 복사하고 직렬화할 필요가 없어지게 됩니다.

### Rust와 JavaScript를 구현하는 프로그램에서 조작하기

위험한 사례를 살펴보는 것으로 시작해봅시다. 매 틱마다 세상을 WebAssembly에서 불러오거나 가져오기 위해 복사하지 않아야 하고, 세포 하나씩 객체를 모두 할당하거나 경계를 오가면서 읽고 쓰는 것도 좋지 않습니다.

그렇다면 어떻게 구현하는게 좋을까요? 죽은 세포를 `0`로 나타내고 살아있는 세포를 `1`로 나타내는 식으로 세포들을 각각 1 byte 값으로 나타내볼수도 있는데, WebAssembly 선형 메모리에 1차원 배열로 나타내봅시다.

4 x 4 사이즈의 우주를 메모리 이미지로 표현해보겠습니다:

![4 x 4 사이즈 세상의 스크린샷](../images/game-of-life/universe.png)

이 공식을 사용해서 주어진 열과 행에 해당하는 배열 인덱스를 찾을 수 있습니다:

```text
index(row, column, universe) = row * width(universe) + column
```

세포들을 JavaScript에 노출시킬때 여러가지 방법을 사용해볼수 있는데, 우선은 Rust `String` 타입의 값으로 세포들을 문자로 표시할수 있도록 `Universe` 타입에 `std::fmt::Display` 트레이트(trait)를 구현해주도록 합시다. 이 트레이트를 통해 Rust `String` 타입의 값을 WebAssembly 선형 메모리에서 JavaScript 가비지 콜렉터가 관리하는 힙으로 복사할 수 있게 됩니다. 그 다음, 복사된 값을 HTML `textConent`에 표시해보도록 하겠습니다. 이 챕터 후반에서는 이 구현에 덧붙여서 세포들을 힙에 복사하지 않도록 해보고 세포들을 `<canvas>`에 표시해볼 예정입니다.

*하나 더 대신 해볼법한 설계가 있는데, 세상 전체를 노출시키지 않고 매 틱마다 상태가 바뀌게 되는 세포들을 목록으로 만들어서 Rust 코드에서 JavaScript로 반환해볼수도 있습니다. 이 방법으로, JavaScript 코드에서 세상 전체를 순회할 필요 없이 일부만 순회할 수 있게 됩니다. 단점으로는, 이 델타 기반 (delta-based)의 설계는 구현하기가 조금 더 어렵습니다.*

## Rust 코드 구현하기

직전 챕터에서 초기 프로젝트 템플릿을 클론했는데, 이 템플릿을 한번 수정해보도록 합시다.

`alert`를 임포트하는 줄과 `greet` 함수를 `wasm-game-of-life/src/lib.rs` 파일에서 지워보고, 세포의 타입 정의를 대신 추가해 주는 것으로 시작해보겠습니다:

```rust
#[wasm_bindgen]
#[repr(u8)]
#[derive(Clone, Copy, Debug, PartialEq, Eq)]
pub enum Cell {
    Dead = 0,
    Alive = 1,
}
```

각 세포가 1 byte 사이즈로 표현돼야 하므로 `#[repr(u8)]`을 잊지 않고 붙여주도록 하고, 세포 주변에 살아있는 이웃들을 쉽게 셀수 있도록, `0`를 `Dead`로, `1`을 `Alive`로 정해주는 것도 중요합니다.

그 다음, 세상을 정의해봅시다. 세상을 나타내는 구조체는 너비와 높이, 세포들을 나타내는 `width * height` 크기의 벡터(vector)를 필드로 가지게 됩니다.

```rust
#[wasm_bindgen]
pub struct Universe {
    width: u32,
    height: u32,
    cells: Vec<Cell>,
}
```

이전에 설명한 내용대로 주어진 행과 열을 세포 벡터의 인덱스로 변환하여 사용할 수 있습니다:

```rust
impl Universe {
    fn get_index(&self, row: u32, column: u32) -> usize {
        (row * self.width + column) as usize
    }

    // ...
}
```

세포의 다음 상태를 계산하려면 몇 개의 이웃이 살아있는지 확인해야 합니다. 이 내용을 토대로 `live_neighbor_count` 메소드를 작성해봅시다!


```rust
impl Universe {
    // ...

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
}
```

`live_neighbor_count` 메소드는 델타값과 나머지 값을 각각 확인해서 세상의 가장자리에서 발생할 수 있는 예외를 `if` 문으로 처리해줍니다. `-1`의 델타를 적용할 때, `self.height - 1`을 추가해서 `1`을 빼는 대신 나머지 값을 계속 처리하고, `row`와 `column`은 각각 `0`이 될수 있습니다. 이 `row`와 `column` 값에서 `1`을 빼려고 시도할 때, unsigned integer underflow 가 발생하게 됩니다.

이제 현재 세대를 기반으로 다음 세대를 처리하는데 필요한 준비가 완료됐습니다! `match` 문을 사용해서 게임의 규칙을 보기 명확하게 나타내봅시다. 추가로 틱이 일어날 때 JavaScript가 컨트롤하도록 할 예정이기 때문에, `#[wasm_bindgen]` 블럭을 추가해서 이 메소드를 JavaScript 코드에 노출시켜보도록 하겠습니다.

```rust
/// Public 메소드, JavaScript로 익스포트 할 수 있게 함.
#[wasm_bindgen]
impl Universe {
    pub fn tick(&mut self) {
        let mut next = self.cells.clone();

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
                    // Rule 3: 과잉 인구로 3개 초과의 이웃을 가진 모든 세포는 죽게 됩니다.
                    (Cell::Alive, x) if x > 3 => Cell::Dead,
                    // 규칙 4: 세포 증식으로 정확히 3개의 이웃을 가진 세포는 살아나게 됩니다.
                    (Cell::Dead, 3) => Cell::Alive,
                    // 규칙이 적용되지 않는 세포들의 상태는 그대로 유지되게 됩니다.
                    (otherwise, _) => otherwise,
                };

                next[idx] = next_cell;
            }
        }

        self.cells = next;
    }

    // ...
}
```

지금까지는 세상의 상태를 세포들의 벡터로 나타냈습니다. 조금 더 사람이 읽기 쉽도록 텍스트 렌더러 (text renderer) 를 구현해보도록 합시다. 세상을 한줄 한줄씩 텍스트로 표현을 해보도록 하는데, 살아있는 세포들을 유니코드 문자 `◼` ("black medium square") 로 나타내고 죽은 세포들을 `◻` ("white medium square") 로 표현해보겠습니다.

또한, Rust 스탠다드 라이브러리의 [`Display`] 트레이트를 구현해서 사람이 읽기 쉬운 방식으로 포맷할수 있도록 메소드를 추가해보겠습니다. 이 트레이트를 구현하면 자동적으로 Universe의 인스턴스(instance)들이 [`to_string`] 메소드를 사용할수 있게 됩니다.

[`Display`]: https://doc.rust-lang.org/1.25.0/std/fmt/trait.Display.html
[`to_string`]: https://doc.rust-lang.org/1.25.0/std/string/trait.ToString.html

```rust
use std::fmt;

impl fmt::Display for Universe {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        for line in self.cells.as_slice().chunks(self.width as usize) {
            for &cell in line {
                let symbol = if cell == Cell::Dead { '◻' } else { '◼' };
                write!(f, "{}", symbol)?;
            }
            write!(f, "\n")?;
        }

        Ok(())
    }
}
```

마지막으로 `render` 메소드와 함께 생성자를 정의해서 세포들이 살아나고 죽어가는 신기한 세상을 생성할 수 있도록 해보겠습니다.

```rust
/// public 메소드, JavaScript로 익스포트 할수 있게 함.
#[wasm_bindgen]
impl Universe {
    // ...

    pub fn new() -> Universe {
        let width = 64;
        let height = 64;

        let cells = (0..width * height)
            .map(|i| {
                if i % 2 == 0 || i % 7 == 0 {
                    Cell::Alive
                } else {
                    Cell::Dead
                }
            })
            .collect();

        Universe {
            width,
            height,
            cells,
        }
    }

    pub fn render(&self) -> String {
        self.to_string()
    }
}
```

드디어 Game of Life의 Rust 코드 구현이 끝났습니다!

이제 `wasm-game-of-life` 디렉토리에서 `wasm-pack build`를 실행해서 WebAssembly 파일을 다시 컴파일해주세요.

## JavaScript로 페이지 렌더링하기

`wasm-game-of-life/www/index.html`에 `<pre>` 요소를 추가해서 세상을 렌더링 해봅시다. `<pre>` 요소를 `<script>` 태그 바로 위에 추가해주세요:

```html
<body>
  <pre id="game-of-life-canvas"></pre>
  <script src="./bootstrap.js"></script>
</body>
```

추가로, `<pre>` 가 웹사이트 중간에 표시될수 있도록 CSS flex box를 사용해봅시다. `wasm-game-of-life/www/index.html` 파일을 열고 `<head>` 내에 `<style>` 태그를 추가해주세요:

```html
<style>
  body {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }
</style>
```

이제 `wasm-game-of-life/www/index.js` 파일 최상단 위치한 기존 `greet` 함수를 지우고 `Universe`를 임포트하는 줄을 추가해주세요.

```js
import { Universe } from "wasm-game-of-life";
```

`<pre>` 요소를 `pre` 상수에 담은 다음 새 우주를 시작해봅시다.

```js
const pre = document.getElementById("game-of-life-canvas");
const universe = Universe.new();
```

이 JavaScript 함수는 [`requestAnimationFrame` 루프][requestAnimationFrame]로 실행해서 매 반복마다 업데이트된 세상을 `<pre>`에 반영하고 `Universe::tick`을 호출합니다.

[requestAnimationFrame]: https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame

```js
const renderLoop = () => {
  pre.textContent = universe.render();
  universe.tick();

  requestAnimationFrame(renderLoop);
};
```

렌더링 처리를 시작하려면 다음 코드를 `renderLoop` 함수 밖에 추가해서 렌더링 루프를 시작해주세요:

```js
requestAnimationFrame(renderLoop);
```

다시 한번 (`npm run`을 `wasm-game-of-life/www` 디렉토리에서 실행한) 개발 서버가 아직 구동중인지 확인해주세요. [http://localhost:8080/](http://localhost:8080/) 페이지를 열면 다음 내용을 확인할수 있게 됩니다:

[![텍스트를 렌더하는 Game of Life 구현 스크린샷](../images/game-of-life/initial-game-of-life-pre.png)](../images/game-of-life/initial-game-of-life-pre.png)

## 메모리에서 바로 캔버스로 렌더링하기

Rust 코드에서 `String`을 생성 (및 할당) 하고 `wasm-bindgen`로 이 생성한 값을 유효한 JavaScript 문자열로 변환하면 세포들을 불필요하게 복사하게 됩니다. 우리가 JavaScript 코드에서 세상의 너비와 높이를 이미 알고 있고, 세포를 만드는 처리가 이루어지는 WebAssembly 선형 메모리를 읽을 수 있기 때문에, `render` 메소드를 수정하여 `cells` 배열의 시작을 가리키는 포인터를 대신 반환해보도록 합시다.

그리고 유니코드 문자를 렌더링하지 않고 [Canvas API] 를 대신 사용해봅시다. 이 API를 이 부분 이후부터 계속 사용하겠습니다.

[Canvas API]: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API

`wasm-game-of-life/www/index.html` 파일에서 이전에 추가한 `<pre>`를 지우고 렌더링에 사용할 `<canvas>`를 추가해주세요. (`<body>` 태그 내에서 JavaScript 코드를 시작시키는 `<script>` 태그 위에 추가돼야 합니다.)

```html
<body>
  <canvas id="game-of-life-canvas"></canvas>
  <script src='./bootstrap.js'></script>
</body>
```

Rust로 구현한 코드에서 필요한 정보를 얻어올 수 있도록 세상의 넓이, 너비, 세포 배열을 가리키는 포인터를 반환하는 getter 함수들을 조금 더 작성해보겠습니다. 이 함수들도 JavaScript 코드로 노출시켜야 하니 잘 확인해주고, `wasm-game-of-life/src/lib.rs` 파일에 다음 코드를 추가해주세요:

```rust
/// Public 메소드, JavaScript로 익스포트 할수 있게 함.
#[wasm_bindgen]
impl Universe {
    // ...

    pub fn width(&self) -> u32 {
        self.width
    }

    pub fn height(&self) -> u32 {
        self.height
    }

    pub fn cells(&self) -> *const Cell {
        self.cells.as_ptr()
    }
}
```

그 다음, `wasm-game-of-life/www/index.js` 파일의 `wasm-game-of-life` 모듈에서 `Cell`을 임포트하는 줄을 추가해준 다음 캔버스를 렌더링할때 사용할 상수 몇가지를 정의해주세요:

```js
import { Universe, Cell } from "wasm-game-of-life";

const CELL_SIZE = 5; // px
const GRID_COLOR = "#CCCCCC";
const DEAD_COLOR = "#FFFFFF";
const ALIVE_COLOR = "#000000";
```

이제 `<pre>` 태그의 `textContent` 대신에 `<canvas>`를 업데이트 할수 있도록 JavaScript 나머지 코드를 다시 작성해주세요:

```js
// Universe를 생성하고 너비와 높이를 반환받습니다.
const universe = Universe.new();
const width = universe.width();
const height = universe.height();

// 캔버스에 세포들을 표시할 공간을 만들어주고, 각 세포들이 1px 두께의 테두리를 가질수 있도록 해줍니다.
const canvas = document.getElementById("game-of-life-canvas");
canvas.height = (CELL_SIZE + 1) * height + 1;
canvas.width = (CELL_SIZE + 1) * width + 1;

const ctx = canvas.getContext('2d');

const renderLoop = () => {
  universe.tick();

  drawGrid();
  drawCells();

  requestAnimationFrame(renderLoop);
};
```
세포들 사이에 격자를 그리려면 일정한 간격으로 나란히 놓인 수평선과 수직선을 그려줍니다. 이러한 선들은 교차하여 격자를 형성하게 됩니다.

```js
const drawGrid = () => {
  ctx.beginPath();
  ctx.strokeStyle = GRID_COLOR;

  // 수직줄
  for (let i = 0; i <= width; i++) {
    ctx.moveTo(i * (CELL_SIZE + 1) + 1, 0);
    ctx.lineTo(i * (CELL_SIZE + 1) + 1, (CELL_SIZE + 1) * height + 1);
  }

  // 수평줄
  for (let j = 0; j <= height; j++) {
    ctx.moveTo(0,                           j * (CELL_SIZE + 1) + 1);
    ctx.lineTo((CELL_SIZE + 1) * width + 1, j * (CELL_SIZE + 1) + 1);
  }

  ctx.stroke();
};
```

raw wasm 모듈인 `wasm_game_of_life_bg`에 정의된 `memory`를 임포트해서 WebAssembly의 선형 메모리에 직접 접근할 수 있습니다. 세포를 그리려면 우선 세상에 있는 세포들을 가리키는 포인터를 반환받고 세포 버퍼(buffer)를 오버레이(overlay)하는 `Uint8Array` 객체를 생성해야 합니다. 그 다음 각 세포를 순회하여 생존 여부에 따라 흰색 또는 검은색 사각형을 그립니다. 포인터를 오버레이하게 되면서, 모든 틱마다 세포들을 복사하지 않도록 최적화 작업을 해줄수 있습니다.

```js
// WebAssembly 메모리를 파일 최상단에 임포트해줍니다.
import { memory } from "wasm-game-of-life/wasm_game_of_life_bg";

// ...

const getIndex = (row, column) => {
  return row * width + column;
};

const drawCells = () => {
  const cellsPtr = universe.cells();
  const cells = new Uint8Array(memory.buffer, cellsPtr, width * height);

  ctx.beginPath();

  for (let row = 0; row < height; row++) {
    for (let col = 0; col < width; col++) {
      const idx = getIndex(row, col);

      ctx.fillStyle = cells[idx] === Cell.Dead
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

  ctx.stroke();
};
```

이전에 보여드린 코드를 사용해서 렌더링 처리를 시작해보겠습니다:

```js
drawGrid();
drawCells();
requestAnimationFrame(renderLoop);
```

`drawGrid()`와 `drawCells()` 두 함수를 `requestAnimationFrame()` 함수 호출 _이전에_ 호출해야 한다는 점을 꼭 기억해주세요. _초기_ 상태의 세상이 그려진 이후에 수정사항을 적용해야 합니다. `requestAnimationFrame(renderLoop)` 함수만 호출하게 된다면 `universe.tick()`이 호출된 _이후_ 시점의 두번째 틱이 대신 그려지게 됩니다.

## 다 됐어요!

최상단 `wasm-game-of-life` 디렉토리에서 다음 명령어를 실행하여 WebAssembly와 바인딩 파일 (bindings glue) 들을 다시 빌드해줍시다:

```
wasm-pack build
```

다시 한번 개발 서버가 아직 구동중인지 확인해주세요. 구동중이지 않다면 `wasm-game-of-life/www` 디렉토리에서 다시 시작해주세요:

```
npm run start
```

[http://localhost:8080/](http://localhost:8080/) 페이지를 웹 브라우저에서 새로고침하면 구현된 흥미진진한 Game of Life가 시작되게 됩니다.

[![Game of Life 구현 스크린샷](../images/game-of-life/initial-game-of-life.png)](../images/game-of-life/initial-game-of-life.png)

추가로 관심이 있다면, [hashlife](https://en.wikipedia.org/wiki/Hashlife) 라는 엄청 멋진 Game of Life 알고리즘 구현도 있으니 한번 확인해보세요. 이 알고리즘은 공격적인 메모이제이션 (aggressive memoizing) 기법을 사용하는데, 덕분에 코드가 더 오래 구동되는 만큼 미래 세대들을 *기하급수적으로 더 빠르게* 계산할수 있게 해줍니다. hashlife를 이 튜토리얼에서 구현해보면 정말 재밌겠지만, 이 책은 Rust와 WebAssembly 사용에 중점을 두고 있으므로 다루지 않도록 하겠습니다. 하지만 hashlife에 대해 따로 배워보길 적극적으로 권장합니다.

## 연습해보기

* [space ship](https://conwaylife.com/wiki/Spaceship) 패턴 하나를 표시하는 세상을 만들어보세요.

* 초기 세상을 하드코딩 하는 대신, 각 세포가 50% 확률로 살아있거나 죽어있는 상태로 랜덤하게 생성될수 있도록 해보세요.

  *힌트: [the `js-sys` crate](https://crates.io/crates/js-sys) 크레이트를 사용하여 [`Math.random` JavaScript 함수](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random)를 임포트해보세요.*

  <details>
    <summary>정답</summary>
    
    *먼저, `wasm-game-of-life/Cargo.toml` 파일을 열고 `js-sys`를 종속성으로 추가해주세요:*

    ```toml
    # ...
    [dependencies]
    js-sys = "0.3"
    # ...
    ```

    *그 다음 `js_sys::Math::random` 함수를 사용해서 50% 확률로 값을 결정해주세요:*

    ```rust
    extern crate js_sys;

    // ...

    if js_sys::Math::random() < 0.5 {
        // 세포가 살아있게 함
    } else {
        // 세포가 죽어있게 함
    }
    ```
  </details>

* 각 세포를 byte 값으로 표현하면서 순회를 쉽게 할수 있지만, 메모리 자원을 낭비한다는 단점이 있습니다. 1 byte는 8 bit인데, 실제로 세포 생존 여부를 표시할 때는 1 bit만 사용하고 있습니다. 이 데이터 표현을을 리팩토링하여 각 세포가 1 bit의 사이즈만 사용할수 있도록 코드를 작성해보세요.

  <details>
    <summary>정답</summary>

    Rust 언어에서 `Vec<Cell>` 타입 대신 (추가 기능을 제공하는 라이브러리인) [`fixedbitset` 크레이트의 `FixedBitSet` 타입](https://crates.io/crates/fixedbitset)을 사용하여 세포들을 나타내볼수도 있습니다.

    ```rust
    // Cargo.toml에 종속성을 추가했는지 확인해주세요!
    extern crate fixedbitset;
    use fixedbitset::FixedBitSet;

    // ...

    #[wasm_bindgen]
    pub struct Universe {
        width: u32,
        height: u32,
        cells: FixedBitSet,
    }
    ```

    Universe의 생성자를 다음과 같이 수정해보겠습니다:

    ```rust
    pub fn new() -> Universe {
        let width = 64;
        let height = 64;

        let size = (width * height) as usize;
        let mut cells = FixedBitSet::with_capacity(size);

        for i in 0..size {
            cells.set(i, i % 2 == 0 || i % 7 == 0);
        }

        Universe {
            width,
            height,
            cells,
        }
    }
    ```

    다음 틱에서 세포를 업데이트할수 있도록 `FixedBitSet` 타입의 `set` 메소드를 사용해봅시다:

    ```rust
    next.set(idx, match (cell, live_neighbors) {
        (true, x) if x < 2 => false,
        (true, 2) | (true, 3) => true,
        (true, x) if x > 3 => false,
        (false, 3) => true,
        (otherwise, _) => otherwise
    });
    ```

    JavaScript에서 시작하는 bit를 가리키는 포인터를 반환해야 하므로, `FixedBitSet` 타입을 슬라이스(slice)로 변환한 다음 변환된 슬라이스를 포인터로 다시 변환해주세요:

    ```rust
    #[wasm_bindgen]
    impl Universe {
        // ...

        pub fn cells(&self) -> *const u32 {
            self.cells.as_slice().as_ptr()
        }
    }
    ```

    JavaScript의 wasm 메모리에서 `Uint8Array`를 생성해오는 방법은 이전과 동일합니다. 하지만 이번에는 각 세포를 나타내기 위해 byte 대신 bit을 사용하므로 배열의 길이가 더이상 `width * height`가 아니고 `width * height / 8`이 되어야 하니 다시 잘 확인해주세요:

    ```js
    const cells = new Uint8Array(memory.buffer, cellsPtr, width * height / 8);
    ```

    다음 함수를 사용하여 인덱스와 `Uint8Array`가 주어질 때 *n*번째 bit의 값이 0인지 1인지 확인할 수 있습니다:

    ```js
    const bitIsSet = (n, arr) => {
      const byte = Math.floor(n / 8);
      const mask = 1 << (n % 8);
      return (arr[byte] & mask) === mask;
    };
    ```
    
    이제 준비가 됐으니 `drawCells` 함수를 다음과 같이 업데이트 해줍시다:

    ```js
    const drawCells = () => {
      const cellsPtr = universe.cells();

      // 수정된 부분입니다!
      const cells = new Uint8Array(memory.buffer, cellsPtr, width * height / 8);

      ctx.beginPath();

      for (let row = 0; row < height; row++) {
        for (let col = 0; col < width; col++) {
          const idx = getIndex(row, col);

          // 수정된 부분입니다!
          ctx.fillStyle = bitIsSet(idx, cells)
            ? ALIVE_COLOR
            : DEAD_COLOR;

          ctx.fillRect(
            col * (CELL_SIZE + 1) + 1,
            row * (CELL_SIZE + 1) + 1,
            CELL_SIZE,
            CELL_SIZE
          );
        }
      }

      ctx.stroke();
    };
    ```

  </details>
