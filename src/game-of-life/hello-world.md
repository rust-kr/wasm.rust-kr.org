# Hello, World!

이 섹션에서는 러스트와 웹어셈블리 프로그램을 처음 실행하는 과정을 다룹니다. "Hello, World!" 메세지를 보여주는 웹페이지를 만들어봅시다.

시작하기 전에 [셋업 가이드](setup.html)를 읽고 잘 따라왔는지 한 번 더 확인해 주세요.

## 프로젝트 템플릿 클론하기

이 프로젝트 템플릿은 합리적인 기본 설정으로 구성돼 있습니다. 이 템플릿을 활용해서 웹으로 코드를 빠르게 개발, 통합 및 패키징할 수 있습니다.

이 명령어로 프로젝트를 클론해보세요:

```text
cargo generate --git https://github.com/rustwasm/wasm-pack-template
```

새 프로젝트 이름을 어떻게 지을지 입력하게 됩니다. **"wasm-game-of-life"** 이라는 이름을 사용합시다.

```text
wasm-game-of-life
```

## 어떻게 구성돼 있나요?

새로 생성한 `wasm-game-of-life` 프로젝트를 열어보도록 합시다.

```
cd wasm-game-of-life
```

그다음 안에 어떤 파일이 담겨있는지 확인해 봅시다:

```text
wasm-game-of-life/
├── Cargo.toml
├── LICENSE_APACHE
├── LICENSE_MIT
├── README.md
└── src
    ├── lib.rs
    └── utils.rs
```

몇 가지 파일을 더 자세히 살펴볼까요?

### `wasm-game-of-life/Cargo.toml`

`Cargo.toml` 파일은 러스트의 패키지 매니저이자 빌드 툴인 `cargo`와 함께 사용되는데, 의존성과 메타데이터를 지정하는 역할을 합니다. 이 파일은 `wasm-bindgen` 의존성과 함께 `wasm` 라이브러리 생성에 사용될 수 있도록 올바르게 설정된 `crate-type`이 함께 사전에 미리 포함돼 있습니다. 몇 가지 필수가 아닌 의존성은 나중에 살펴보겠습니다.

### `wasm-game-of-life/src/lib.rs`

`src/lib.rs` 파일은 웹어셈블리로 컴파일하는 러스트 크레이트의 핵심 코드입니다. `wasm-bindgen`을 자바스크립트를 조작하기 위해 사용하고, `window.alert` 자바스크립트 함수를 불러온 다음에 `greet` 러스트 함수를 자바스크립트로 보냅니다. 이렇게 "Hello World" alert 메세지를 표시시킬 수 있게 됩니다.

```rust
mod utils;

use wasm_bindgen::prelude::*;

// `wee_alloc` 기능이 활성화돼 있으면, `wee-alloc`를 전역 할당자로 사용합니다.
#[cfg(feature = "wee_alloc")]
#[global_allocator]
static ALLOC: wee_alloc::WeeAlloc = wee_alloc::WeeAlloc::INIT;

#[wasm_bindgen]
extern {
    fn alert(s: &str);
}

#[wasm_bindgen]
pub fn greet() {
    alert("Hello, wasm-game-of-life!");
}
```

### `wasm-game-of-life/src/utils.rs`

`src/utils.rs` 모듈은 자주 사용되는 유틸리티를 제공하는데, 이 유틸리티가 웹어셈블리로 컴파일된 러스트 코드 작업을 더 쉽게 할 수 있도록 도와줍니다. [wasm 코드 디버깅하기](debugging.html)와 같은 튜토리얼 후반 섹션에서 이러한 유틸리티들을 더 자세히 살펴보겠습니다. 현재 시점에서는 이 파일을 무시해도 괜찮습니다.

## 프로젝트 빌드하기

`wasm-pack`을 사용하여 다음 빌드 과정을 자동화하게 됩니다:

* 러스트 1.30나 그 이후 버전을 사용하고 있는지, `wasm32-unknown-unknown` 타겟이 `rustup`을 통해 설치돼 있는지 확인합니다.
* `cargo`를 사용하여 러스트 소스 코드를 웹어셈블리 `.wasm` 바이너리로 컴파일합니다.
* `wasm-bindgen`을 사용하여 러스트로 생성한 웹어셈블리를 사용할 수 있도록 자바스크립트 API를 생성합니다.

위 작업을 시작하려면, 프로젝트 경로에서 다음 명령어를 실행해 주세요:

```
wasm-pack build
```

빌드가 완료되면 결과물을 `pkg` 경로에서 확인해 보세요. 다음 내용물을 확인할 수 있습니다:

```
pkg/
├── package.json
├── README.md
├── wasm_game_of_life_bg.wasm
├── wasm_game_of_life.d.ts
└── wasm_game_of_life.js
```

`README.md` 파일이 메인 프로젝트에서 복사됐지만 나머지 파일들은 완전히 새로 생성된 부분을 확인할 수 있습니다.

### `wasm-game-of-life/pkg/wasm_game_of_life_bg.wasm`

이러한 `.wasm` 파일은 러스트 컴파일러가 러스트 소스 코드로 생성한 바이너리 파일입니다. 이 파일은 wasm 형식으로 컴파일된 모든 러스트 함수들과 데이터로 구성돼 있습니다. 변환된 "greet" 함수가 예가 될 수 있습니다.

### `wasm-game-of-life/pkg/wasm_game_of_life.js`

이러한 `.js` 파일은 `wasm-bindgen`에 의해 생성됩니다. DOM과 자바스크립트 함수를 러스트에서 호출할 수 있게 해주고, 자바스크립트 환경에서도 웹어셈블리 함수를 호출할 수 있도록 도와주는 유용한 API를 노출시켜줍니다. 예를 들어서, `greet` 이라는 자바스크립트 함수를 통해 웹어셈블리의 해당하는 함수를 호출할 수 있습니다. 현재 시점에서는 이러한 바인딩(bindings glue)들이 큰 역할을 하지는 않지만, 더 복잡한 값들을 웹어셈블리와 자바스크립트 사이에서 주고받을 때 정말 도움이 많이 됩니다.

```js
import * as wasm from './wasm_game_of_life_bg';

// ...

export function greet() {
    return wasm.greet();
}
```

### `wasm-game-of-life/pkg/wasm_game_of_life.d.ts`

`.d.ts` 파일은 생성한 자바스크립트 파일과 함께 사용할 수 있는 [TypeScript][] 타입 정의를 포함합니다. TypeScript를 사용한다면 정적 타입 정의를 통해 IDE가 제공하는 자동 완성과 같은 기능을 사용해서 더 쉽게 웹어셈블리 함수를 호출할 수 있게 됩니다. TypeScript를 사용하지 않는다면, 이 파일은 무시해도 괜찮습니다.

```typescript
export function greet(): void;
```

[TypeScript]: http://www.typescriptlang.org/

### `wasm-game-of-life/pkg/package.json`

[`package.json` 파일은 생성된 자바스크립트와 웹어셈블리 패키지에 대한 메타데이터를 포함합니다.][package.json] 이런 메타데이터는 npm과 자바스크립트 번들러가 여러 패키지들의 종속성, 패키지 이름, 버전 등을 결정할 때 사용됩니다. 자바스크립트 툴링과 함께 사용하고 npm에 웹어셈블리 패키지를 배포할 때 유용합니다.

```json
{
  "name": "wasm-game-of-life",
  "collaborators": [
    "Your Name <your.email@example.com>"
  ],
  "description": null,
  "version": "0.1.0",
  "license": null,
  "repository": null,
  "files": [
    "wasm_game_of_life_bg.wasm",
    "wasm_game_of_life.d.ts"
  ],
  "main": "wasm_game_of_life.js",
  "types": "wasm_game_of_life.d.ts"
}
```

[package.json]: https://docs.npmjs.com/files/package.json

## 웹 페이지에 포함시키기

`wasm-game-of-life` 패키지를 웹페이지에 포함시키고 웹 환경에서 작동시키기 위해, [
`create-wasm-app` 자바스크립트 프로젝트 템플릿][create-wasm-app] 을 사용해 봅시다.

[create-wasm-app]: https://github.com/rustwasm/create-wasm-app

다음 명령어를 `wasm-game-of-life` 경로 내부에서 실행해 주세요:

```
npm init wasm-app www
```

새롭게 생성된 내부 경로인 `wasm-game-of-life/www` 는 다음 파일들을 포함합니다:

```
wasm-game-of-life/www/
├── bootstrap.js
├── index.html
├── index.js
├── LICENSE-APACHE
├── LICENSE-MIT
├── package.json
├── README.md
└── webpack.config.js
```

한 번 더 생성된 파일들을 자세히 살펴봅시다.

### `wasm-game-of-life/www/package.json`

이 `package.json` 파일은 `webpack`과 `webpack-dev-server` 종속성들로 미리 셋업이 되어 있고, npm 에 배포돼 있는 `wasm-pack-template`의 초기 버전인 `hello-wasm-pack` 패키지 또한 종속성으로 포함하고 있습니다.

### `wasm-game-of-life/www/webpack.config.js`

이 파일은 webpack과 로컬 개발 서버를 설정합니다. 미리 셋업 작업이 돼 있는데, webpack과 로컬 개발 서버를 사용할 때에도 전혀 따로 수정할 필요가 없습니다.

### `wasm-game-of-life/www/index.html`

웹사이트에 사용되는 최상단 HTML 파일입니다. `index.js` 를 감싸주는 `bootstrap.js`를 부르는 것 외에는 특별한 동작을 하지 않습니다.

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Hello wasm-pack!</title>
  </head>
  <body>
    <script src="./bootstrap.js"></script>
  </body>
</html>
```

### `wasm-game-of-life/www/index.js`

이 `index.js`는 작업을 하게 될 웹사이트 자바스크립트의 진입점 (entry point) 입니다. 자바스크립트 코드와 `wasm-pack-template`의 컴파일 결과물인 웹어셈블리가 포함돼 있는데, `hello-wasm-pack` npm 패키지를 로드하고 패키지 내부의 `greet` 함수를 호출해 줍니다.

```js
import * as wasm from "hello-wasm-pack";

wasm.greet();
```

### 종속성 설치하기

우선, `wasm-game-of-life/www` 내부 경로에서 `npm install` 명령어를 실행하여 로컬 개발 서버와 종속성들이 설치돼 있는지 확인해 주세요:

```text
npm install
```

이 커맨드는 한 번만 실행돼야 합니다. 실행하면 `webpack` 자바스크립트 번들러와 개발 서버를 설치할 수 있습니다.

> 단순히 간편하게 책을 진행하기 위해 이 번들러와 개발 서버를 사용할 예정이지만
> `webpack`이 러스트와 웹어셈블리 작업에 필수가 아니라는 점을 기억해 주세요.
> Parcel과 Rollup도 웹어셈블리와 ECMAScript 모듈을 부르는 데 사용할 수 있습니다.
> 원한다면 러스트와 웹어셈블리를 [번들러 없이][] 사용할 수도 있습니다.

[번들러 없이]: https://rustwasm.github.io/docs/wasm-bindgen/examples/without-a-bundler.html

### `www` 패키지 내부에서 로컬 `wasm-game-of-life` 패키지 사용하기

npm에서 다운로드한 `hello-wasm-pack` 대신에 로컬 환경에 있는 `wasm-game-of-life`를 사용해 봅시다. 이렇게 함으로써 Game of Life 프로그램을 더 점진적으로 개발할 수 있게 됩니다.

`wasm-game-of-life/www/package.json` 파일을 열고 `devDependencies` 다음에 `dependencies` 필드를 생성해 주세요. 그다음에, `"wasm-game-of-life": "file:../pkg"` 엔트리를 포함시켜주세요.

```js
{
  // ...
  "dependencies": {                     // 이 3줄 길이의 블럭을 추가해주세요!
    "wasm-game-of-life": "file:../pkg"
  },
  "devDependencies": {
    //...
  }
}
```

다음으로, `hello-wasm-pack` 대신에 `wasm-game-of-life`를 부를 수 있도록 `wasm-game-of-life/www/index.js` 파일을 수정해 주세요:

```js
import * as wasm from "wasm-game-of-life";

wasm.greet();
```

새 종속성을 만들었다면, 다음 명령어로 설치해야 합니다:

```text
npm install
```

이제 웹사이트를 로컬 환경에서 구동할 수 있게 됐습니다!

## 로컬 환경에서 구동하기

이제 개발 서버를 구동할 새 터미널을 열어주세요. 새로 연 터미널에서 서버를 구동하면 백그라운드에서 계속 서버를 구동하면서 다른 명령어를 계속 입력할 수 있게 됩니다. 새 터미널에서 `wasm-game-of-life/www` 경로로 들어간 다음 이 명령어를 실행해 주세요:

```
npm run start
```

웹 브라우저를 열고 [http://localhost:8080/](http://localhost:8080/) 를 열면 표시되는 "Hello World" alert 메세지를 확인할 수 있습니다:

[!["Hello, wasm-game-of-life!" 웹 페이지 alert 메세지 스크린샷](../images/game-of-life/hello-world.png)](../images/game-of-life/hello-world.png)

파일을 저장할 때마다 [http://localhost:8080/](http://localhost:8080/) 페이지에 반영되도록 하고 싶다면, `wasm-pack build` 명령어를 `wasm-game-of-life` 경로에서 다시 실행해 주세요.

## 연습해 보기

* `wasm-game-of-life/src/lib.rs` 경로에 있는 `greet` 함수를
  수정해서 표시되는 메세지를 커스터마이징할 수 있도록 `name: &str` 매개변수를 추가해 보고, `wasm-game-of-life/www/index.js` 파일에서 `greet` 함수를 이름과 함께 호출해 보세요.
  `wasm-pack build` 명령어로 `.wasm` 바이너리를 다시 빌드하고, [http://localhost:8080/](http://localhost:8080/) 페이지를 새로고침하면 브라우저에서 수정된 알림 메세지를 확인할 수 있습니다.

  <details>
    <summary>정답</summary>

    `wasm-game-of-life/src/lib.rs` 파일에서 새롭게 수정된 `greet` 함수:

    ```rust
    #[wasm_bindgen]
    pub fn greet(name: &str) {
        alert(&format!("Hello, {}!", name));
    }
    ```
    
    `wasm-game-of-life/www/index.js` 파일에서 수정된 `greet` 함수 호출하기:

    ```js
    wasm.greet("Your Name");
    ```

  </details>
