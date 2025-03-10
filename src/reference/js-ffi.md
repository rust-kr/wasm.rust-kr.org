# 자바스크립트 상호 운용하기

## 자바스크립트 함수 임포트하고 익스포트하기

### 러스트 사이드

자바스크립트 환경에서 웹어셈블리를 사용할 때 러스트 사이드에서 함수들을 임포트하고 익스포팅하는 작업은 의외로 매우 간단합니다. 그리고 C 언어와 유사하게 작동하기도 합니다.

웹어셈블리 모듈은 임포트 시퀀스(sequence)를 선언하는데, 각각 시퀀스는 *모듈 이름*과 *임포트 이름*으로 구성됩니다. 기본값으로 "env" 파일에 설정된 것과 같이, [`#[link(wasm_import_module)]`][wasm_import_module]를 사용하여 `extern { ... }` 블럭 모듈 이름을 직접 지정해 줄 수도 있습니다.

익스포트는 한가지 이름만 가질 수 있는데, 다른 `extern` 함수들과 마찬가지로, 웹어셈블리 인스턴스의 선형 메모리는 기본값으로는 "memory" 라는 이름으로 익스포트 됩니다.

[wasm_import_module]: https://github.com/rust-lang/rust/issues/52090

```rust
// `foo`라는 자바스크립트 함수를 `mod` 모듈에서 임포트해옵니다.
#[link(wasm_import_module = "mod")]
extern { fn foo(); }

// `bar`라는 러스트 함수를 익스포트합니다.
#[no_mangle]
pub extern fn bar() { /* ... */ }
```

웹어셈블리는 제한된 수의 값 타입들을 가지고 있기 때문에, 이러한 함수들은 원시 숫자 타입에서만 작동해야 한다는 부분도 참고해 주세요.

### 자바스크립트 사이드

자바스크립트 코드 내에서는 웹어셈블리 바이너리가 ES6 모듈로 변환되는 것을 확인할 수 있습니다. 이 모듈은 선형 메모리와 함께 먼저 *인스턴스화* 돼야하고, 임포트한 내용과 일치하는 자바스크립트 함수 그룹을 필요로 합니다. 인스턴스화에 대해 더 자세히 알아보려면 [MDN][instantiation]를 참고해 주세요.

[instantiation]: https://developer.mozilla.org/en-US/docs/Web/자바스크립트/Reference/Global_Objects/WebAssembly/instantiateStreaming

출력하게 되는 ES6 모듈은 러스트에서 익스포트한 함수들을 모두 포함하게 되는데, 이런 함수들은 자바스크립트 함수에서 호출할 수 있습니다.

셋업을 하고 실행시키는 전반적으로 간단한 예시를 확인하고 싶다면 [여기][hello world]를 참고해 주세요.

[hello world]: https://www.hellorust.com/demos/add/index.html

## 숫자 외에 다른 값도 사용해보기

자바스크립트에서 웹어셈블리를 사용할 때는 웹어셈블리 모듈의 메모리와 자바스크립트 메모리 사이가 명확하게 구분됩니다:

- (이 문서 최상단에 설명된 대로) 각각의 웹어셈블리 모듈은 인스턴스화 과정에서 생성하게 되는 선형 메모리를 가지게 됩니다. **자바스크립트 코드는 자유롭게 이 메모리를 읽고 쓸 수 있습니다.**

- 반면에, 웹어셈블리에서는 자바스크립트 객체에 직접 접근할 수 없습니다.

그런 이유로 보통은 두 가지 방법으로 정교화된 상호작용을 처리하게 됩니다:

- 바이너리 데이터를 웹어셈블리 메모리에서 복사하거나 붙여놓습니다. 예를 들어서, 이 방법으로 러스트 코드가 소유하는 `String` 값을 사용할 수도 있습니다.

- 명시적으로 자바스크립트 객체의 힙을 설정하고 이후에 "addresses"를 할당합니다. 이렇게 웹어셈블리 코드에서 (정수 값을 사용하여) 간접적으로 자바스크립트 객체를 참조할 수 있고, 임포트한 자바스크립트 함수를 호출해서 사용할 수 있게 됩니다.

다행스럽게도 이 상호 운용 작업은 [wasm-bindgen]이라는 `bindgen` 스타일의 프레임워크를 사용해서 아주 쉽게 처리할 수 있습니다. 이 프레임워크를 사용해서 관용적인 스타일의 자바스크립트 함수에 자동으로 매핑되도록 관용적인 러스트 함수 시그니처를 생성할 수 있습니다.

[wasm-bindgen]: https://github.com/rustwasm/wasm-bindgen

## 사용자 정의 섹션 (Custom Sections)

사용자 정의 섹션을 통해 웹어셈블리 모듈에 명명된 임의의 데이터를 임베딩할 수 있습니다. 이 섹션 데이터는 컴파일 시점에 설정되며 웹어셈블리 모듈에서 직접 읽을 수 있지만 런타임에서는 이 모듈을 수정할 수 없습니다.

러스트 코드에서 다음과 같이 사용자 정의 섹션을 정적 배열 (`[T; size]`)로 나타내고 `#[link_section]` 속성으로 노출시킬 수 있습니다:

```rust
#[link_section = "hello"]
pub static SECTION: [u8; 24] = *b"This is a custom section";
```

이 코드는 wasm 파일에 `hello` 라는 사용자 정의 섹션을 추가합니다. 변수 이름 `SECTION` 은 임의로 명명됐으며 이 이름을 변경해도 코드의 동작이 바뀌지 않습니다. 이 변수에 텍스트 바이트 (bytes of text) 를 사용했지만 다른 임의의 데이터를 사용할 수도 있습니다.

이런 사용자 정의 섹션은 JS 사이드에서 [`WebAssembly.Module.customSections`]  함수를 사용하여 읽을 수도 있습니다. 이 함수를 호출할 때, wasm 모듈과 섹션 이름을 인자로 주고 [`ArrayBuffer`]의 배열을 반환받게 됩니다. 여러 섹션들이 같은 이름을 공유하게 할 수도 있는데, 이 경우에는 한 배열에 여러 섹션들을 담게 됩니다.

```js
WebAssembly.compileStreaming(fetch("sections.wasm"))
.then(mod => {
  const sections = WebAssembly.Module.customSections(mod, "hello");

  const decoder = new TextDecoder();
  const text = decoder.decode(sections[0]);

  console.log(text); // -> "사용자 정의 섹션을 콘솔에 출력합니다"
});
```

[`ArrayBuffer`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer
[`WebAssembly.Module.customSections`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WebAssembly/Module/customSections
