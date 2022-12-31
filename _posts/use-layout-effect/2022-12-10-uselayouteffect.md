---
title: useLayoutEffect에 대하여
date: 2022-12-10 00:00
modified: 2022-12-10 00:00
tags: [uselayoutefect]
description: useLayoutEffect는 언제 사용할까
image: "https://velog.velcdn.com/images/dmsqlswkd/post/0893d13b-662a-4096-807c-327770775f19/image.png"
---

> 공식문서에 따르면, `useLayoutEffect`는 브라우저가 스크린을 repaint하기 전에 호출되는`useEffect` 의 다른 버전이다.

#### `useLayoutEffect`는 언제 필요한가?

예를 들어 어떤 버튼에 마우스를 올렸을 때 나타나는 툴팁을 구현한다고 생각해보자. 만약 이 버튼 상단에 충분한 공간이 있다면 툴팁을 요소위에 띄워주면 되지만, 공간이 충분치 않은 경우에는 요소의 아래쪽에 띄워주어야한다.

이러한 경우에 툴팁을 올바른 위치에 띄우기 위해서는 툴팁이 화면에 보여지기 전에 버튼의 위치와 툴팁의 크기를 알아야 한다.

useEffect훅의 이펙트함수는 DOM의 레이아웃 배치와 페인트가 끝난 후, 즉 DOM 요소들이 화면에 보여진 뒤에 실행된다. 그래서 만약 useEffect 내에서 DOM에 영향을 주는 코드가 있을 경우 사용자 입장에서는 화면의 깜빡임을 보게 될 것이다.

위 툴팁을 useEffect만으로 구현한다면 화면상에서 다음과 같이 보일 것이다.

<figure>
  <img src="https://velog.velcdn.com/images/dmsqlswkd/post/74af3967-ed7a-4330-92d3-c1f0b8d24c06/image.gif" alt="tooltip example using useEffect" width=500>
  <figcaption>아주 당황스럽다.</figcaption>
</figure>

useLayoutEffect는 이 문제를 해결해준다.

<figure>
  <img src="https://velog.velcdn.com/images/dmsqlswkd/post/0893d13b-662a-4096-807c-327770775f19/image.png" alt="react-hook-flow" width=500>
</figure>

⬆️ [hook flow](https://github.com/donavon/hook-flow)를 통해 확인해볼 수 있는 react hook flow이다.

useLayoutEffect는 **컴포넌트가 렌더링된 후, 브라우저가 화면에 DOM을 그리기(Paint) 전**에 이펙트 함수를 수행한다.

따라서 useEffect내에서 dom에 영향을 주는 코드를 작성해야하는 예외적인 상황에서 useLayoutEffect를 사용하면 유저 입장에서 더 편안한 화면을 볼 수 있게 될 것이다.

<figure>
  <img src="https://velog.velcdn.com/images/dmsqlswkd/post/d708ee81-fb59-4bdc-8944-9bdac493f4a1/image.gif" alt="tooltip example using useLayoutEffect" width=500>
</figure>

위 예제에 사용한 코드는 [여기](https://gist.github.com/eunbin20/cc634e750746e3c88ea90b3f549148bd)에서 확인할 수 있다.

## 참고

https://beta.reactjs.org/reference/react/useLayoutEffect

https://all-dev-kang.tistory.com/entry/%EB%A6%AC%EC%95%A1%ED%8A%B8-useEffect%EC%99%80-useLayoutEffect-%EB%B9%84%EA%B5%90%EC%8B%9C%EB%A6%AC%EC%A6%88
