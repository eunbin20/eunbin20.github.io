---
title: 러닝 타입스크립트 책 리뷰 | 함수, 배열, 인터페이스
date: 2023-07-02 00:00
modified: 2023-07-02 00:00
tags: [typescript]
description: void vs undefined vs never
image: "https://github.com/eunbin20/til/assets/43979066/439eea19-e0ee-49f3-9038-2044b5d95182"
---

최근 러닝 타입스크립트 책을 이용해 타입스크립트 스터디를 진행하고있다.

5장.함수, 6장.배열, 7장.인터페이스 챕터를 읽으며 인상적이었던 내용을 위주로 정리해보려 한다.

## 1. 함수의 매개변수의 타입 추론 방식

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

인상적이었던 부분은 rating의 타입 추론이 함수를 호춭하는 코드에서는 number 타입으로 유추되지만, 함수를 호출하는(사용하는) 코드에서는 선택적 `number '| undefined`로 유추된다는 점이다.

<figure>
  <img src="https://github.com/ianlee92/learning-typescript-study/assets/43979066/0bd404d2-3ce5-45ed-9907-9f4dcc6b46ee" alt="thinking emoji" width=500>
  <figcaption>함수 선언부 - rating의 타입이 number로 유추되었다.</figcaption>
</figure>

<figure>
  <img src="https://github.com/ianlee92/learning-typescript-study/assets/43979066/953ab33c-4179-4fe4-b0e5-6707f4351253" alt="thinking emoji" width=500>
  <figcaption>함수 호출부 - rating의 타입이 number | undefined로 유추되었다.</figcaption>
</figure>

즉, **함수의 매개변수의 초깃값을 지정하면 자동적으로 해당 타입 `| undefined`로 지정된다.**

## 2. ?으로 표현되는 선택적 매개변수와 undefined를 포함하는 유니언 타입의 차이

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

하지만 ?로 표시한 것은 매개변수 자체를 선택적으로 받겠다라고 정의하는 것이고, 선택적 매개변수 없이 `| undefined`를 포함하는 유니언 타입으로 선언하는 것은 값을 무조건 받되, undefined도 가능하게끔 타입을 선언한 것이라는 점에서 이 둘은 다르다.

## 3. 반환 타입으로 void vs undefined vs never

- void: 함수의 반환 타입이 무시된다. **함수가 반환하는 모든 값을 무시**하도록 설정할 때 유용하다.
  - ex) 자바스크립트 내장 함수 forEach() 메서드는 void를 반환하는 콜백을 받는다.
- undefined: 말그대로 undefined가 반환된다는 것을 명시적으로 선언한 것.
- never 타입: (의도적으로) 항상 오류를 발생시키거나 무한루프를 실행하는 함수. 절대 반환하지 않는다.

> **void는 아무것도 반환하지 않는 함수를 표현할 때, never는 절대 반환하지 않는 함수의 타입을 표현할 때 사용한다.**

## 3-1. never의 또다른 용법: 복잡한 조건 타입 표현하기.

어떤 함수는 2가지의 매개변수를 받을 수 있다. 그러나 number 인자를 받으면 string은 받을 수 없고, 반대로 string을 받으면 number는 받을 수 없다.

이러한 조건을 가진 함수의 타입을 정의할 대 아래와 같이 선택적 매개변수와 never 타입을 이용해 타입 정의를 할 수 있다.

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

## 4. enum을 사용하는 것은 성능면에서 좋지 않다!

enum이란 열거형 변수로 Javascript에서는 제공하지 않는 Typescript가 자체적으로 구현하는 기능이다.

enum을 사용한 코드를 컴파일하게 되면 다음과 같은 결과를 얻을 수 있다.

```jsx
export enum FRUITS {
  APPLE = 'apple',
  BANANA = 'banana'
};
```

[컴파일 결과]

```jsx
export var FRUITS;
(function (FRUITS) {
  FRUITS["APPLE"] = "apple";
  FRUITS["BANANA"] = "banana";
})(FRUITS || (FRUITS = {}));
```

위와 같이 즉시실행함수의 형태로 객체를 만들게 되는데, 이 코드는 번들러가 '사용하지 않는'코드라고 판단하지 않으므로 트리쉐이킹되지 않고 최종 번들에 포함된다. 즉, 개발 환경에서만 필요한 불필요한 타입스크립트 코드로 인해 번들의 크기가 증가하게 되는 것이다.

이러한 enum 타입을 대체하기위한 방법으로 const 어서션을 사용할 수 있다.

```jsx
export const FruitsType = {
  APPLE = 'apple',
  BANANA = 'banana'
} as const;

export type FruitsType = keyof typeof FruitsType;
```

## 5. 배열은 불안정하다.

Typescript의 타입 시스템은 완벽하지 않다. Typescript에서는 기본적으로 모든 배열의 멤버에 대한 접근에 대해 해당 배열의 멤버를 반환한다고 가정한다. 즉, 어떤 배열의 해당 인덱스에 실제로 값이 존재하지 않아도 존재한다고 가정한다. (배열에 대해서는 자바스크립트에서도 배열의 길이보다 큰 인덱스로 배열 요소에 접근하면 레퍼런스 에러가 아닌 undefined를 제공한다.)

```jsx
function withElement(elements: string[]) {
  console.log(elements[9001].length); // 타입 오류 없음
}

withElements(["It's", "over"]);
```

Typescript는 검색된 배열의 멤버가 존재하는지 의도적으로 확인하지 않으므로 위와 같은 상황에서 타입 오류를 발생시키지 않는다. 또한 `elements[9001]`는 undefined가 아니라 string 타입으로 간주된다.

지금까지 개발을 하는 과정에서 위와 같은 상황이 생겼을 것 같은데, 타입 오류가 발생하지 않는 것에 대해 큰 의문을 갖지 않았다. 하지만 이 책을 통해 예제를 접해보니 타입스크립트를 기술적으로 불안정하다고 하는 것을 이해할 수 있었고 흥미로웠다.

## 참고

https://beta.reactjs.org/reference/react/useLayoutEffect

https://all-dev-kang.tistory.com/entry/%EB%A6%AC%EC%95%A1%ED%8A%B8-useEffect%EC%99%80-useLayoutEffect-%EB%B9%84%EA%B5%90%EC%8B%9C%EB%A6%AC%EC%A6%88

https://engineering.linecorp.com/ko/blog/typescript-enum-tree-shaking
