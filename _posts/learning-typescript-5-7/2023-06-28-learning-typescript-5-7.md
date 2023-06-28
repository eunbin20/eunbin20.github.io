---
title: 러닝 타입스크립트 책 리뷰 | 함수, 배열, 인터페이스
date: 2022-12-10 00:00
modified: 2022-12-10 00:00
tags: [uselayoutefect]
description: useLayoutEffect는 언제 사용할까
image: "https://velog.velcdn.com/images/dmsqlswkd/post/0893d13b-662a-4096-807c-327770775f19/image.png"
---

최근 러닝 타입스크립트 책을 이용해 타입스크립트 스터디를 진행하고있다.

5장.함수, 6장.배열, 7장.인터페이스 챕터를 읽으며 인상적이었던 내용을 위주로 정리해보려 한다.

## 함수의 매개변수의 타입 추론 방식

기본 함수 매개변수에서도 초기 변숫값과 유사하게 타입 추론이 이루어진다.

```jsx
function rateSong(song: string, rating = 0) {
  console.log(`${song} gets ${rating}/5 stars`);
}

rateSong("Photograph"); //ok
rateSong("Set Fire to the Rain", 5); //ok
rateSong("Set Fire to The Rain", undefined); //ok

rateSong("At Last!", "100"); // error
```

song에는 string타입을 명시적으로 선언해줬지만 rating에는 초기값만 할당되어있고 따로 타입 애너테이션이 없다.

그러나 타입스크립트는 위와 같은 경우 매개변수의 초기값을 기반으로 타입을 유추한다.

인상적이었던 부분은 rating의 타입 추론이 함수를 호춭하는 코드에서는 number 타입으로 유추되지만, 함수를 호출하는(사용하는) 코드에서는 선택적 number | undefined로 유추된다는 점이다.

<figure>
  <img src="https://github.com/ianlee92/learning-typescript-study/assets/43979066/0bd404d2-3ce5-45ed-9907-9f4dcc6b46ee" alt="thinking emoji" width=500>
  <figcaption>함수 선언부 - rating의 타입이 number로 유추되었다.</figcaption>
</figure>

<figure>
  <img src="https://github.com/ianlee92/learning-typescript-study/assets/43979066/953ab33c-4179-4fe4-b0e5-6707f4351253" alt="thinking emoji" width=500>
  <figcaption>함수 호출부 - rating의 타입이 number | undefined로 유추되었다.</figcaption>
</figure>

즉, **함수의 매개변수의 초깃값을 지정하면 자동적으로 해당 타입 | undefined로 지정된다.**

## ?으로 표현되는 선택적 매개변수와 | undefined를 포함하는 유니언 타입의 차이

```jsx
// | undefined를 포함하는 유니언 타입
function getSingerA(singer: string | undefined) {}
// 선택적 매개변수
function getSingerB(singer?: string) {}

getSingerA("Greensleeves"); // error
getSingerB("Greensleeves"); // ok

getSingerA(undefined); // ok
getSingerB(undefined); // ok

getSingerA("Sia"); //ok
getSingerB("Sia"); //ok
```

> _막연하게 타입에 ?으로 표시하면 undefined를 유니온타입을 추가한 것과 동일하나, 좀 더 편리하다._ 정도로만 생각하고있었다.

하지만 ?로 표시한 것은 매개변수 자체를 선택적으로 받겠다라고 정의하는 것이고, 선택적 매개변수 없이 | undefined를 포함하는 유니언 타입으로 선언하는 것은 값을 무조건 받되, undefined도 가능하게끔 타입을 선언한 것이라는 점에서 이 둘은 다르다.

## 반환 타입으로 void vs undefined

- void: 함수의 반환 타입이 무시된다. **함수가 반환하는 모든 값을 무시**하도록 설정할 때 유용하다.
  - ex) 자바스크립트 내장 함수 forEach() 메서드는 void를 반환하는 콜백을 받는다.
- undefined: 말그대로 undefined가 반환된다는 것을 명시적으로 선언한 것.

## void vs never

### never 타입

(의도적으로) 항상 오류를 발생시키거나 무한루프를 실행하는 함수.
절대 반환하지 않는다.

never는 이렇게도 쓸 수 있다.

```jsx
interface TextOnly {
  number?: never;
  string: string;
}
interface NumberOnly {
  number: number;
  string?: never;
}

type Interface = TextOnly | NumberOnly;

function foo({ number, string }: Interface) {
  // ...
}

foo({ number: 1, string: "a" }); // error
foo({ number: 1 }); // ok
foo({ string: "a" }); //ok
```

foo 함수는 2가지의 매개변수를 받을 수 있다. 그러나 number를 받으면 string은 들어오면 안되고, 반대로 string을 받으면 number는 들어오면 안된다.

위와 같은 경우 선택적 매개변수와 never 타입을 이용해 위와 같이 타입 정의를 할 수 있다.

- **void: 아무것도 반환하지 않는 함수**
- **never: 절대 반환하지 않는 함수**

## enum 대신 as const를 써야하는 이유

## Type-alias와 Interface의 차이점. 둘 중 어느 것을 써야 할까?

## Chapter 5. Function

## 참고

https://beta.reactjs.org/reference/react/useLayoutEffect

https://all-dev-kang.tistory.com/entry/%EB%A6%AC%EC%95%A1%ED%8A%B8-useEffect%EC%99%80-useLayoutEffect-%EB%B9%84%EA%B5%90%EC%8B%9C%EB%A6%AC%EC%A6%88
