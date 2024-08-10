# Game of Life 테스팅하기

브라우저 JavaScript 환경에서 실행할 수 있도록 Game of Life를 구현했으니 이제 Rust 코드에서 WebAssembly 함수를 테스팅하는 방법에 대해 알아봅시다.

`tick` 함수로 예상값과 일치하는 올바른 값을 불러올 수 있는지 테스팅 해보겠습니다.

우선 테스팅을 작성하기 전에 `wasm_game_of_life/src/lib.rs` 파일에 작성한 `impl Universe` 블럭 내부에 setter 함수와 getter 함수들을 조금 더 작성해 보겠습니다. `set_width`와 `set_height` 함수를 작성해서 다른 사이즈의 `Universe`들을 만들어볼 수 있도록 해봅시다.

```rust
#[wasm_bindgen]
impl Universe { 
    // ...

    /// 세상의 너비를 설정합니다.
    ///
    /// 모든 세포를 죽은 상태로 리셋합니다.
    pub fn set_width(&mut self, width: u32) {
        self.width = width;
        self.cells = (0..width * self.height).map(|_i| Cell::Dead).collect();
    }

    /// 세상의 넓이를 설정합니다.
    ///
    /// 모든 세포를 죽은 상태로 리셋합니다.
    pub fn set_height(&mut self, height: u32) {
        self.height = height;
        self.cells = (0..self.width * height).map(|_i| Cell::Dead).collect();
    }

}
```

`wasm_game_of_life/src/lib.rs` 파일에 `#[wasm_bindgen]` 속성 없이 `impl Universe` 블럭을 하나 더 만들어보겠습니다. 추가로 테스팅에 사용하는 데 필요한 함수가 몇 개 있는데, 이 함수들은 JavaScript로 노출시키지 않아야 합니다. Rust로 생성한 WebAssembly 함수는 대여한 참조를 반환하지 못하는데, 이 함수들 위에 `#[wasm_bindgen]` 속성을 추가해 보고 어떤 에러를 확인할 수 있게 되는지 살펴봅시다.

`get_cells` 함수를 구현해서 `Universe`의 `cells` 필드 값을 가져와 보겠습니다. `set_cells` 함수도 작성해서 주어진 행과 열에 위치한 `Universe`의 세포를 `Alive` 상태로 업데이트할 수 있도록 해보겠습니다.

```rust
impl Universe {
    /// 세상에 존재하는 모든 죽어있는 세포와 살아있는 세포를 반환합니다.
    pub fn get_cells(&self) -> &[Cell] {
        &self.cells
    }

    /// 배열로 주어진 행과 열들을 확인하고 세포들을 살아있는 상태로 업데이트합니다.
    pub fn set_cells(&mut self, cells: &[(u32, u32)]) {
        for (row, col) in cells.iter().cloned() {
            let idx = self.get_index(row, col);
            self.cells[idx] = Cell::Alive;
        }
    }

}
```

이제 `wasm_game_of_life/tests/web.rs` 파일에 테스팅 코드를 작성해 보도록 하겠습니다.

진행하기 전에, 이미 완성된 테스팅 코드가 있으니 한번 살펴봅시다. `wasm-game-of-life` 경로에서 `wasm-pack test --chrome --headless` 명령어를 실행하여 Rust로 생성한 WebAssembly 테스팅 코드가 잘 작동하는지 확인할 수 있습니다. `--firefox`, `--safari`, `--node` 옵션을 사용하여 특정 브라우저 환경에서 코드를 테스트할 수도 있습니다.

우선 `wasm_game_of_life/tests/web.rs` 파일에서, `wasm_game_of_life` 크레이트와 `Universe`를 익스포트 해줍시다.

```rust
extern crate wasm_game_of_life;
use wasm_game_of_life::Universe;
```

`wasm_game_of_life/tests/web.rs` 파일에서 spaceship 예시 패턴을 만드는 함수들을 작성해 봅시다.

`tick` 함수를 호출할 때 사용할 패턴과, 한 틱 이후의 결괏값을 비교할 때 사용할 예상값이 필요한데, 우선은 `input_spaceship` 함수에서 어떤 세포들을 `Alive` 상태로 생성할지 정해줍시다. `expected_spaceship` 함수를 테스트해 보는데, `input_spaceship` 호출 이후 시점 틱의 spaceship 패턴을 직접 계산해서 값을 채워보도록 하겠습니다.

```rust
#[cfg(test)]
pub fn input_spaceship() -> Universe {
    let mut universe = Universe::new();
    universe.set_width(6);
    universe.set_height(6);
    universe.set_cells(&[(1,2), (2,3), (3,1), (3,2), (3,3)]);
    universe
}

#[cfg(test)]
pub fn expected_spaceship() -> Universe {
    let mut universe = Universe::new();
    universe.set_width(6);
    universe.set_height(6);
    universe.set_cells(&[(2,1), (2,3), (3,2), (3,3), (4,2)]);
    universe
}
```
마지막으로 `test_tick` 함수를 구현해 주도록 하겠습니다. 먼저 `input_spaceship()`와 `expected_spaceship()`를 호출해서 인스턴스들을 만들어줍시다. 그다음에 `input_universe` 인스턴스의 `tick` 함수를 호출해 주도록 합시다. 추가로, `assert_eq!` 매크로를 사용해서 `get_cells()`를 호출한 다음 `input_universe`와 `expected_universe` 가 동일한 `Cell` 타입의 배열을 값으로 가지고 있는지 확인해 보겠습니다. 마무리로는 이 코드 블럭 위에 `#[wasm_bindgen_test]` 속성을 추가해서 `wasm-pack test` 명령어로 Rust로 생성한 WebAssembly 코드를 테스트할 수 있도록 해줍시다.

```rust
#[wasm_bindgen_test]
pub fn test_tick() {
    // 작은 사이즈의 Universe과 spaceship 패턴을 생성해 봅시다.
    let mut input_universe = input_spaceship();

    // `input_universe` 인스턴스의 다음 틱 결괏값과 비교하게 될 spaceship 패턴입니다.
    let expected_universe = expected_spaceship();

    // `tick` 함수를 실행하고 두 `Universe`의 세포들이 동일한지 확인합니다.
    input_universe.tick();
    assert_eq!(&input_universe.get_cells(), &expected_universe.get_cells());
}
```

이제 `wasm-game-of-life` 경로에서 `wasm-pack test --firefox --headless` 명령어를 실행하여 테스팅 코드를 실행해 주면 됩니다.