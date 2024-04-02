# npm에 배포하기

이제 잘 작동하는 빠르고 작은 `wasm-game-of-life` 패키지가 있으니, 다른 JavaScript 개발자들이 Game of Life 코드를 바로 받아 사용해야 할 때 종속성으로 사용할수 있도록 npm에 배포해보겠습니다.

## 준비 사항

우선, [기존에 생성된 npm 계정이 있는지 확인해주세요](https://www.npmjs.com/signup).

그 다음, 다음 명령어를 실행하여 로컬 머신에서 계정에 로그인돼 있는지 확인해주세요:

```
wasm-pack login
```

## 배포하기

`wasm-game-of-life` 디렉토리에서 `wasm-pack` 명령어를 실행하여 `wasm-game-of-life/pkg` 작성한 코드가 잘 빌드돼 있는지 확인해주세요:

```
wasm-pack build
```

`wasm-game-of-life/pkg` 폴더 안의 내용물을 한변 살펴보겠습니다. 이 폴더에 있는 파일들을 바로 다음 단계에서 npm에 배포해볼 예정입니다!

준비가 됐다면, `wasm-pack publish` 를 실행해서 패키지를 npm에 업로드 해보세요:

```
wasm-pack publish
```

정말 놀랍게도 이게 답니다! 이렇게 npm에 패키지를 업로드할수 있는데...

.. 이 튜토리얼을 끝낸 다른 사람들도 npm에 배포를 할것 같습니다. `wasm-game-of-life` 라는 패키지 이름이 높은 확률로 이미 사용중일것 같고, 마지막으로 실행한 명령어가 아마 안됐을겁니다.

`wasm-game-of-life/Cargo.toml` 파일을 열고 고유한 패키지 이름을 만들수 있도록 `name` 필드에 유저네임(username)을 추가해주세요.

```toml
[package]
name = "wasm-game-of-life-my-username"
```

이제 다시 빌드하고 배포해보겠습니다:

```
wasm-pack build
wasm-pack publish
```

이번에는 잘 작동될겁니다!