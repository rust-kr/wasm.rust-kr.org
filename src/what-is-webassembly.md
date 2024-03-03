# WebAssembly가 뭔가요?

WebAssembly(wasm)는 [포괄적인 사양]이 있는 간단한 기계 모델이자 실행 가능한 포맷입니다. 휴대 가능하고, 가벼우며 거의 네이티브 프로그램과 같은 속도로 실행될수도 있도록 설계됐습니다.

프로그래밍 언어로써, 웹어셈블리는 같은 구조를 나타내면서도 두가지 다른 포맷으로 구성돼 있습니다.

1. `.wat` 텍스트 포맷 ("**W**eb**A**ssembly **T**ext" 에서 유래함)은 [S-expressions] 구조를 사용하고 Scheme이나 Clojure와 같은 Lisp 계열 언어와 유사점을 공유합니다.

2. `.wasm` 바이너리 포맷은 더 저레벨이면서 웹어셈블리 가상 머신에서 바로 사용되도록 의도됐습니다. 개념적으로 ELF 와 Mach-0와 비슷합니다.

참고 자료로, `wat` 언어로 작성된 팩토리얼 함수를 확인해보세요.

```
(module
  (func $fac (param f64) (result f64)
    local.get 0
    f64.const 1
    f64.lt
    if (result f64)
      f64.const 1
    else
      local.get 0
      local.get 0
      f64.const 1
      f64.sub
      call $fac
      f64.mul
    end)
  (export "fac" (func $fac)))
```

위 예제가 `.wasm` 파일로는 어떻게 보일지 궁금하다면, [wat2wasmdemo] 웹사이트를 이용해보세요.

## 선형 메모리
WebAssembly 매우 간단한 [메모리 모델]을 가지고 있고, 하나의 wasm 모듈은 하나의 "선형 메모리" 에 접근할수 있습니다.

이 메모리는 페이지 사이즈 (64K)의 곱만큼 [커질수 있으며] 이 사이즈는 줄어들 수 없습니다. 

## 웹에서만 웹어셈블리를 사용할 수 있나요?

현재로써는 JavaScript 웹 커뮤니티에서 주목을 받고 있지만, wasm은 특정 실행 환경을 필요로 하지 않습니다. 그러므로, wasm이 미래에 다양한 맥락에서 사용할수 있는 "휴대 가능한 실행할수 있는" 포맷이라고 여겨질수도 있습니다. 하지만 오늘날 wasm은 대부분 (웹과 [Node.js]을 포함한) 다양한 형태로 존재하는 JavaScript(JS)와 함께 언급됩니다.

[memory model]: https://webassembly.github.io/spec/core/syntax/modules.html#syntax-mem
[memory can be grown]: https://webassembly.github.io/spec/core/syntax/instructions.html#syntax-instr-memory
[extensive specification]: https://webassembly.github.io/spec/
[value types]: https://webassembly.github.io/spec/core/syntax/types.html#value-types
[Node.js]: https://nodejs.org
[S-expressions]: https://en.wikipedia.org/wiki/S-expression
[wat2wasm demo]: https://webassembly.github.io/wabt/demo/wat2wasm/
