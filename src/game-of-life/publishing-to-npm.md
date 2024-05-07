# npm에 배포하기

사이즈가 작고 빠르게 작동하는 `wasm-game-of-life` 패키지가 준비됐습니다. 이제 다른 JavaScript 개발자들이 Game of Life 코드를 바로 종속성으로 받아 사용해볼수 있도록 npm에 배포해보겠습니다.

## 준비 사항

우선, [기존에 생성된 npm 계정이 있는지 확인해주세요](https://www.npmjs.com/signup).

그 다음, 다음 명령어를 실행하여 로컬 머신에서 계정에 로그인되어 있는지 확인해주세요:

```
wasm-pack login
```

## 배포

`wasm-game-of-life` 경로에서 `wasm-pack` 명령어를 실행하여 `wasm-game-of-life/pkg` 경로에 작성한 코드가 잘 빌드돼 있는지 확인해주세요:

```
wasm-pack build
```

`wasm-game-of-life/pkg` 폴더의 내용물들을 한변 살펴보겠습니다. 이 폴더에 있는 파일들을 이제 npm에 배포해볼 예정입니다!

준비가 됐다면, `wasm-pack publish` 를 실행해서 패키지를 npm에 업로드해보세요:

```
wasm-pack publish
```

정말 놀랍게도 이게 답니다! 이렇게 npm에 패키지를 업로드할수 있는데...

... 이 튜토리얼을 끝낸 다른 사람들도 npm에 배포를 했을 가능성이 높습니다. 그렇기 때문에 `wasm-game-of-life` 라는 패키지 이름이 높은 확률로 이미 사용중일수도 있습니다. 마지막으로 실행한 명령어가 이러한 이유로 성공적으로 실행되지 못한 부분이 확인되나요?

`wasm-game-of-life/Cargo.toml` 파일을 열고 작성한 패키지가 고유한 이름을 가질수 있도록 `name` 필드에 원하는 유저 이름을 추가해주세요.

```toml
[package]
name = "wasm-game-of-life-my-username"
```

이제 다시 빌드하고 배포해보겠습니다:

```
wasm-pack build
wasm-pack publish
```

이번에는 잘 배포됐을겁니다!