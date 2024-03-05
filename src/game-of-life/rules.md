# Conway's Game of Life의 규칙

*Note: 이미 Conway's Game of Life 와 이 게임의 규칙을 잘 알고있다면 다음 섹션으로 넘어가도 괜찮습니다.*

[Wikipedia에 Conway's Game of Life의 규칙이 아주 잘 설명돼 있습니다.][wikipedia]

> Game of Life의 세상은 사각형 세포로 이루어진 무한한 사이즈의 2차원 직교 격자인데,
> 각각의 세포는 살아있거나 죽어있거나, 혹은 "주거"나 "무주거" 중 한 상태일수 있습니다.
> 각 세포은 수평, 수직, 혹은 대각선으로 이웃하는 여덟개의 이웃 세포과 상호 작용합니다.
> 매 단계마다, 다음 전이가 발생합니다.
>
> 1. 인구 부족으로 2개 미만의 이웃을 가진 세포는 죽게 됩니다.
> 2. 2개 혹은 3개의 이웃을 가진 세포는 다음 세대에서 계속 살아있습니다.
> 3. 과잉 인구로 3개 초과의 이웃을 가진 모든 세포는 죽게 됩니다.
> 4. 세포 증식으로 정확히 3개의 이웃을 가진 세포는 살아나게 됩니다.
>
> 이 초기 패턴은 게임 시스템의 시작점(seed)을 만들게 됩니다. 위 규칙들을 시작점(seed)의 모든 세포에
> 적용하면서 첫번째 세대가 생성되게 됩니다. 출생과 사망은 동시에 일어나고, 이가 발생하는
> 이산적인 순간을 틱(tick) 이라고 부릅니다. (다시 말해, 각 세대는 직전 세대의 순수 함수입니다.)
> 이 규칙은 계속 적용되어 반복적으로 추가 세대를 만듭니다.

[wikipedia]: https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life

다음 이미지를 초기 세상이라고 생각해봅시다:

<img src='../images/game-of-life/initial-universe.png' alt='Initial Universe' width=80 />

다음 세대를 계산할 때 각 세포를 하나씩 고려해볼 수 있는데, 하나씩 살펴보도록 합시다. 왼쪽 최상단 세포는 죽어있습니다. 규칙 (4)는 죽어있는 세포에만 적용되는 유일한 전이 세포이지만, 최상단 왼쪽 셀은 정확히 3개의 살아있는 이웃을 가지고 있지 않기 때문에 전이 규칙이 적용되지 않고 다음 세대에서도 죽어 있는 상태로 유지됩니다. 동일한 이유로 첫 번째 행의 다른 세포들도 그대로 죽어있게 됩니다.

두번째 행, 세번째 열의 살아있는 세포를 보면 매우 흥미로운 내용을 확인할수 있는데, 세포가 살아있다면 첫번째 세 규칙이 적용될수도 있습니다. 이 세포는 단 한 개의 살아있는 이웃을 가지고 있기 때문에 규칙 (1)이 적용되게 되어 다음 세대에서 죽게 됩니다. 최하단의 살아있는 세포도 동일하게 죽습니다.

가운데에 있는 살아있는 세포는 위 아래로 두 개의 살아있는 이웃을 가지고 있습니다. 그러므로 규칙 (2)가 적용이 되어 다음 세대에서도 살아있게 됩니다.

마지막으로 가운데에 살아있는 세포들의 왼쪽과 오른쪽 이웃들을 보면 정말 흥미로운 부분을 확인할 수 있는데, 이 3개의 살아있는 세포들은 양쪽 방향으로 두 세포들과 이웃해 있기 때문에 규칙 (4)가 적용이 됩니다. 그러므로 이 세포들은 다음 세대에서 살아있게 됩니다.

위에서 다룬 내용을 토대로, 다음 틱의 세상은 이렇게 보이게 됩니다:

<img src='../images/game-of-life/next-universe.png' alt='Next Universe' width=80 />

이러한 간단하고 결정적인 규칙을 적용하는 것으로 다음과 같은 이상하지만 흥미로운 동작을 볼수 있습니다:

| Gosper's glider gun                                                                                | Pulsar                                                                                 | Space ship                                                                                                   |
| -------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| ![Gosper's glider gun](https://upload.wikimedia.org/wikipedia/commons/e/e5/Gospers_glider_gun.gif) | ![Pulsar](https://upload.wikimedia.org/wikipedia/commons/0/07/Game_of_life_pulsar.gif) | ![Lighweight space ship](https://upload.wikimedia.org/wikipedia/commons/3/37/Game_of_life_animated_LWSS.gif) |

<center>
<iframe width="560" height="315" src="https://www.youtube.com/embed/C2vgICfQawE?rel=0&amp;start=65" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
</center>

## 연습해보기

* 방금 보여드린 두번째 틱 예시를 손으로 직접 계산해보세요. 어떻게 감이 오시는것 같나요?

  <details>
    <summary>정답</summary>

    예시의 초기 세상은 다음과 같이 생겼습니다:

    <img src='../images/game-of-life/initial-universe.png' alt='Initial Universe' width=80 />

    이 패턴은 두 틱마다 처음 상태로 돌아기기 때문에 주기적이라고 볼수 있습니다.

  </details>

* 안정된 초기 세상를 찾아보실수 있으신가요? 세대가 바뀌어도 내용이 바뀌지 않는 세상을 찾아보세요.

  <details>
    <summary>정답</summary>

    안정된 세상은 무한하게 많습니다. 지루하게도 텅 비어있는 세상도 안정된 세상이고, 살아있는 세포들이 2:2 사각형 모양을 형성할 때도 안정된 세상을 볼수 있습니다.

  </details>
