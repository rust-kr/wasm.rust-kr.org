# 상호작용 추가하기

구현한 Game of Life에 상호작용을 조금 더 추가해보면서 JavaScript와 WebAssembly 인터페이스를 조금 더 탐험해보도록 하겠습니다. 이 섹션에서는 유저들이 세포를 클릭해서 생존 여부를 전환시킬수 있도록 해보고, 일시정지 기능도 구현해서 세포 패턴을 유저들이 손쉽게 그려볼 수 있도록 해볼 예정입니다.

## 일시정지 버튼

게임을 일시정지 시킬수 있도록 버튼을 하나 만들어 봅시다. `wasm-game-of-life/www/index.html` 파일 내의 `<canvas>` 바로 위에 이 버튼을 추가해주세요:

```html
<button id="play-pause"></button>
```

`wasm-game-of-life/www/index.js` JavaScript 파일에 다음 변경사항들을 적용해보겠습니다:

* 취소시키고 싶은 식별자를 `cancelAnimationFrame` 함수로 전달시킬수 있도록 `requestAnimationFrame` 함수가 제일 마지막으로 반환한 식별자를 추적합니다.

* 재생/일시정지 버튼을 눌렀을 때, 대기 중인 애니메이션 프레임을 가리키는 식별자를 가지고 있는지 확인합니다. 이 식별자를 가지고 있는 시점에서는 게임이 실행중일 것이므로 `renderLoop`가 다시 호출되지 않도록 애니메이션을 일시정지 시킵니다. 식별자를 가지고 있지 않다면 게임이 멈춰있다는 의미이므로 `requestAnimationFrame` 함수를 사용하여 게임을 다시 시작해주도록 합니다.

이 작업은 JavaScript 환경에서 진행해야 하기 때문에 따로 Rust 소스 코드를 수정할 필요가 없습니다.

`animationId` 변수를 추가하여 `requestAnimationFrame` 함수가 반환하는 식별자를 추적해봅시다. 대기 중인 애니메이션이 없을 때는 이 값을 `null`로 설정해주겠습니다.

```js
let animationId = null;

// `requestAnimationFrame`의 반환값이 `animationId`에
// 할당된다는 점 외에는 기존과 동일한 함수입니다.
const renderLoop = () => {
  drawGrid();
  drawCells();

  universe.tick();

  animationId = requestAnimationFrame(renderLoop);
};
```

언제든 `animationId`의 값을 확인해서 게임이 멈춰있는지 확인할 수 있습니다:

```js
const isPaused = () => {
  return animationId === null;
};
```

이제 재생/일시정지 버튼이 클릭됐을 때 게임의 재생 여부를 확인할 수 있게 됐습니다. 마찬가지로 `renderLoop` 함수를 통해 애니메이션을 다시 시작하거나 일시정지 시킬수도 있습니다. 추가로, 버튼을 클릭했을 때 표시되는 텍스트의 아이콘을 업데이트 해주도록 하겠습니다.

```js
const playPauseButton = document.getElementById("play-pause");

const play = () => {
  playPauseButton.textContent = "⏸";
  renderLoop();
};

const pause = () => {
  playPauseButton.textContent = "▶";
  cancelAnimationFrame(animationId);
  animationId = null;
};

playPauseButton.addEventListener("click", event => {
  if (isPaused()) {
    play();
  } else {
    pause();
  }
});
```

마지막으로, 추가한 버튼이 올바른 초기 텍스트 아이콘을 표시하도록 이전에 `requestAnimationFrame(renderLoop)`을 직접 불러서 빠르게 시작했던 부분을 `play` 함수를 대신 호출하도록 변경해보겠습니다.

```diff
// 기존에는 `requestAnimationFrame(renderLoop)`를 호출하는 줄이었습니다.
play();
```

[http://localhost:8080/](http://localhost:8080/) 페이지를 새로고침하면 일시정지/재생 버튼이 추가된 부분을 확인할 수 있습니다!

## `onclick` 이벤트로 세포 상태 전환하기

이제 게임을 일시정지 할수 있게 됐으니 세포들을 클릭해서 상태를 바꿀수 있도록 코드를 수정해보겠습니다.

세포를 클릭할 때, 클릭한 세포의 생존 여부를 전환할 수 있도록 코드를 작성해보겠습니다. `toggle` 메소드를 `wasm-game-of-life/src/lib.rs` 파일 내의 `Cell`에 추가해줍시다.

```rust
impl Cell {
    fn toggle(&mut self) {
        *self = match *self {
            Cell::Dead => Cell::Alive,
            Cell::Alive => Cell::Dead,
        };
    }
}
```

주어진 행과 열에 위치한 세포의 상태를 전환할 수 있도록, 행과 열을 벡터의 인덱스로 변환하고, 변환한 인덱스를 세포 벡터로 변환하겠습니다. 세포 벡터가 준비됐다면 변환된 인덱스에 위치하는 세포를 전환할수 있도록 메소드를 호출해봅시다:

```rust
/// Public 메소드, JavaScript로 익스포트 할수 있도록 함.
#[wasm_bindgen]
impl Universe {
    // ...

    pub fn toggle_cell(&mut self, row: u32, column: u32) {
        let idx = self.get_index(row, column);
        self.cells[idx].toggle();
    }
}
```

이 `impl` 블럭 내에 정의된 메소드에 `#[wasm_bindgen]` 속성을 추가하여 JavaScript 코드에서 호출할수 있도록 했습니다.

`wasm-game-of-life/www/index.js` 파일에서 `<canvas>` 요소의 클릭 이벤트를 수신(listening)해보겠습니다. 콜백 함수 내부에는 페이지의 상대 좌표를 캔버스 상대 좌표로 변환한 다음 `toggle_cell` 메소드를 호출하여 장면을 다시 그려볼수 있도록 코드를 작성해보겠습니다.

```js
canvas.addEventListener("click", event => {
  const boundingRect = canvas.getBoundingClientRect();

  const scaleX = canvas.width / boundingRect.width;
  const scaleY = canvas.height / boundingRect.height;

  const canvasLeft = (event.clientX - boundingRect.left) * scaleX;
  const canvasTop = (event.clientY - boundingRect.top) * scaleY;

  const row = Math.min(Math.floor(canvasTop / (CELL_SIZE + 1)), height - 1);
  const col = Math.min(Math.floor(canvasLeft / (CELL_SIZE + 1)), width - 1);

  universe.toggle_cell(row, col);

  drawGrid();
  drawCells();
});
```
`wasm-game-of-life` 경로에서 `wasm-pack build` 명령어를 실행하여 다시 빌드한 다음, [http://localhost:8080/](http://localhost:8080/) 페이지를 새로고침해보세요. 이제 세포를 직접 클릭해서 상태를 전환할 수 있게 됐습니다. 패턴을 한번 그려보세요!

## 연습해보기

* [`<input type="range">`][input-range] 위젯을 추가하여 매 프레임마다 발생하는 틱의 수를 조절할 수 있도록 코드를 작성해보세요.

* 세상을 랜덤한 초기 상태로 리셋할수 있도록 버튼을 하나 더 추가해보세요. 또 다른 버튼을 추가해서 모든 세포가 죽은 상태로 새로 시작할 수 있도록 구현해봐도 좋습니다.

* Ctrl을 누른 상태로 세포를 클릭하면 (`Ctrl + 클릭`) 클릭한 세포를 중심으로 [glider](https://en.wikipedia.org/wiki/Glider_(Conway%27s_Life)) 패턴이 추가되도록 코드를 작성해보세요. Shift를 누른 상태로 클릭할 때는 (`Shift + Click`) pulsar 패턴을 그리도록 작성해봐도 좋습니다.

[input-range]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/range
