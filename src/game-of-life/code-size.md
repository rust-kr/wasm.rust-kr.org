# `.wasm` 파일 사이즈 줄이기

구현했던 Game of Life 프로그램과 같이 `.wasm` 형식으로 컴파일된 바이너리를 유저들에게 네트워크로 전송할 때 코드 사이즈를 신경 쓰는 편이 좋습니다. 대부분의 경우에는 `.wasm` 파일의 사이즈가 작을수록 페이지가 빨리 로드되고 유저 경험이 더 나아집니다.

## 빌드 설정으로 `.wasm` 바이너리 사이즈를 얼마나 줄일 수 있나요?

[`wasm` 바이너리 사이즈를 줄일 때 어떤 빌드 설정을 수정해 볼 수 있는지 한번 살펴보세요.](../reference/code-size.html#optimizing-builds-for-code-size)


(debug 심볼이 빠져있는) 기본 빌드 설정으로 Game of Life를 빌드하면 29,410 bytes 사이즈의 바이너리가 출력되게 됩니다:

```
$ wc -c pkg/wasm_game_of_life_bg.wasm
29410 pkg/wasm_game_of_life_bg.wasm
```

LTO를 활성화한 다음 `opt-level = "z"`를 설정하고 `wasm-opt -Oz` 명령어를 실행해서 빌드하면 출력되는 `.wasm` 바이너리의 사이즈가 17,317 bytes 정도로 줄어드는 부분을 확인할 수 있습니다.

```
$ wc -c pkg/wasm_game_of_life_bg.wasm
17317 pkg/wasm_game_of_life_bg.wasm
```

그다음에는 출력된 바이너리를 (거의 모든 HTTP 서버가 하는 대로) `gzip`를 사용하여 압축해 보는데, 정말 귀엽게도 바이너리 파일의 사이즈를 9,045 bytes까지 줄일 수 있게 됩니다!

```
$ gzip -9 < pkg/wasm_game_of_life_bg.wasm | wc -c
9045
```

## 연습해 보기

* [`wasm-snip` 툴](../reference/code-size.html#use-the-wasm-snip-tool)을 사용해서 구현했던 Game of Life의 `.wasm` 바이너리에서 패닉 디버깅 함수를 제외시켜보세요. 파일의 사이즈가 얼마나 줄어드나요?

* [`wee_alloc` 를 전역 할당자 (global allocator)](https://github.com/rustwasm/wee_alloc)를 사용해 보고 파일의 사이즈를 이전과 비교해 보세요. 프로젝트를 시작할 때 클론 했던 `rustwasm/wasm-pack-template` 템플릿은 "wee_alloc" 이라는 cargo 기능을 포함하고 있는데, 이 기능은 `wasm-game-of-life/Cargo.toml` 파일에 있는 `[features]` 섹션의 `default` 필드에 추가해서 활성화할 수 있습니다.

  ```toml
  [features]
  default = ["wee_alloc"]
  ```

  `wee_alloc`을 활성화하면 `.wasm` 바이너리 사이즈가 얼마나 줄어드나요?

* 튜토리얼을 진행하는 내내 한 `Universe`만 페이지에 포함시켰는데, 생성자를 사용하는 대신 단 한 개의 `static mut` 전역 인스턴스 (global instance) 만 수정하는 코드를 익스포트 해볼 수도 있습니다. 이전 챕터에서 다룬 이중 버퍼링 기법 (double buffering technique) 을 사용하고 싶다면, 이러한 버퍼도 `static mut` 키워드를 사용하여 전역적으로 만들어볼 수 있습니다. 이런 방식으로, 구현한 게임에서 모든 동적 할당 (dynamic allocation) 을 없앨 수 있고, `#![no_std]` 속성을 추가하여 할당기(allocator)가 없는 크레이트를 만들어볼 수도 있습니다. 동적 할당에 필요한 종속성을 다 없애면 `.wasm` 파일의 사이즈가 얼마나 줄어드나요?