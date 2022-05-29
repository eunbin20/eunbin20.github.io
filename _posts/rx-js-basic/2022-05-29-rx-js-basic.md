---
title: RxJS 개요 | ReactiveProgramming | Observable
date: 2022-05-29 00:00
modified: 2022-05-29 00:00
tags: [rx-js, observable]
description: ApolloClient의 Link에 대해 파보다가 Observable을 접했다. Observable은 무엇이고 RxJS는 무엇인가..🤔 에 대해 간단히 정리해보았다.
image: "https://user-images.githubusercontent.com/43979066/170882759-df0f4c86-6376-4b80-b96f-c97a5ec1692d.png"
---

- [RxJS: Reactive Extensions For JavaScript](#rxjs-reactive-extensions-for-javascript)
- [Observable](#observable)
  - [Observable의 LifeCycle](#observable의-lifecycle)
      - [1.&nbsp;Creating Observables](#1creating-observables)
      - [2.&nbsp;Subscribing to Observables](#2subscribing-to-observables)
      - [3.&nbsp;Executing Observables](#3executing-observables)
      - [4.&nbsp;Disposing Observable Executions](#4disposing-observable-executions)
- [Operator](#operator)
    - [Filter](#filter)
    - [Map](#map)
    - [FlatMap](#flatmap)
- [_Resources_](#resources)

> ApolloClient의 Apollo link는 내부적으로 Observable이라는 객체를 통해 비동기적인 동작을 수행한다. 이 Observable이라는 객체는 RxJS에서 사용되는 객체인데 Observable과 RxJS에 대해 간단히 정리해보았다.

## Rx: Reactive + eXtensions

ReactiveX 공식 문서에서는 Rx를 다음과 같이 소개한다.

> An API for asyncronous programming with observable streams
> 관찰 가능한 스트림으로 비동기 프로그램을 하는 하나의 API

## 명령형 프로그래밍 vs. 반응형 프로그래밍

간단한 예시를 통해 명령형 프로그래밍과 반응형 프로그래밍을 비교해보자.

- 명령형 프로그래밍

```jsx
a = 0;
b = 1;
c = 2;
a = b + c;
print(a); //3
c = 100;
print(a); //3
```

- 반응형 프로그래밍

```jsx
a = 0;
b = 1;
c = 2;
a = b + c;
print(a); //3
c = 100;
print(a); //101
```

**반응형 프로그래밍**은 주변 환경과 끊임없이 상호작용하는 프로그래밍을 의미한다. 프로그램이 로직의 흐름을 주도하는 것이 아니라 환경이 변하면 이벤트를 받아 동작하도록 만드는 프로그래밍 기법이다.

**명령형 프로그래밍**은 개발자에 의해서 작성된 코드가 정해진 순서대로 실행되는 방식의 프로그래밍을 의미한다.

대부분의 프로그래밍 패러다임에서 개발자는 본인이 작성한 순서대로 한 번에 하나씩 로직이 진행되는 것을 예상한다. 그러나 ReactiveX에서 많은 실행문은 **parallel**방식으로 수행되고 그 결과들은 `observer`에 의해 이후에 랜덤한 순서로 포착된다.

<br/>

# RxJS: Reactive Extensions For JavaScript

RxJS는 자바스크립트에서 리액티브(반응형)프로그래밍을 도와주는 일종의 API이다. 데이터를 fetch해오거나, 키 입력, 마우스 클릭 등의 이벤트 소스를 비동기적으로 처리해 기존의 Promise 기반의 코드를 대체할 수 있다.

기본적으로 JavaScript는 멀티 패러다임 언어로 명령형, 함수형, 객체지향 프로그래밍 언어이다. 거기에 Reactive Programming을 위해 RxJS를 사용할 수 있다.

# Observable

> 스마트폰이 Observable(관찰 가능한) 객체라면, 사용자는 Observer(관찰자)이다.

스마트폰은 관찰 가능(observable)하고 알림, 메세지 등과 같은 신호를 방출 한다. 사용자는 자연적으로 스마트폰을 구독(subscribe)하고 있고, 모든 알림을 홈 스크린에서 확인할 수 있다. 그리고 그 신호로 무엇을 할 지 정할 수 있다.

> In ReactiveX an *observer* *subscribes* to an *Observable.* Then that observer reacts to whatever item or sequence of items the Observable *emits*. <br/>
> Observer는 Observable을 구독한다. 그리고 Observer는 Observable이 방출하는 하나 또는 연속된 항목에 반응한다.

## Observable의 LifeCycle

#### 1.&nbsp;Creating Observables

Observable 객체는 `new Observable` 키워드를 통해 만들어진다. 그리고 Observable 생성자는 subscribe라는 한 개의 함수를 인자로 갖는다.

```jsx
import { Observable } from "rxjs";

const observable = new Observable(function subscribe(subscriber) {
  // ...
});
```

#### 2.&nbsp;Subscribing to Observables

Observable에서 구독(Subscribing)한다는 것은 전달할 데이터를 인자로 넣어 콜백 함수를 호출하는 것이다.

```jsx
import { Observable } from "rxjs";

// 1초에 한 번씩 subscriber에게 "hi"라는 문자열을 방출하는 observable 객체
const observable = new Observable(function subscribe(subscriber) {
  const id = setInterval(() => {
    subscriber.next("hi");
  }, 1000);
});
```

여기서 `addEventListener`와 같은 이벤트 핸들러 API와는 확실히 다른 점은 `observable.subscribe`에서 해당 `observer`는 Observable에 리스너를 등록하는 것이 아니라는 것이다.

#### 3.&nbsp;Executing Observables

subscribe 함수를 호출하여은 Observable을 실행을 시작하여 데이터 또는 이벤트를 observer로 전달할 수 있다.
observer에게 데이터를 전달하는 방식에는 세 가지가 있다.

- Next : 숫자, 문자열, 객체 등 값을 보낸다.
- Error: 데이터를 생성하는 데 실패했을 경우, 또는 다른 에러가 발생했을 경우 에러를 발생시킨다.
- Complete: 가장 마지막으로 값을 보낼 경우 사용한다.

#### 4.&nbsp;Disposing Observable Executions

observer가 값을 받고나면 메모리 낭비를 막기 위해 구독하고 있는 모든 대상의 구독을 종료해야 한다.

<br/>

# Operator

Observable에서 받은 이벤트를 변환하고 처리하는 연산자.

### Filter

- 해당 조건식을 만족하는 Observable만 발생시키는 메소드

<figure>
<img src="https://user-images.githubusercontent.com/43979066/170882941-cf3d9b2b-2714-4f73-abf4-6255cbe40e05.png" alt="" width=500>
</figure>

### Map

- Observable을 콜백함수 실행을 통해 가공
- 단일 스트림의 원소를 매핑 시킨 후 매핑 시킨 값을 다시 스트림으로 반환하는 중간 연산

### FlatMap

- Map과 마찬가지로 콜백함수를 실행함으로써 Observable을 가공한다.
- Map과의 차이점은 **Observable 객체를 리턴**한다는 것
- Array나 Object로 감싸져 있는 모든 원소를 단일 원소 스트림으로 반환

<br/>

# _Resources_

- [https://pks2974.medium.com/rxjs-%EA%B0%84%EB%8B%A8%EC%A0%95%EB%A6%AC-41f67c37e028](https://pks2974.medium.com/rxjs-%EA%B0%84%EB%8B%A8%EC%A0%95%EB%A6%AC-41f67c37e028)
- [https://rxjs.dev/guide/overview](https://rxjs.dev/guide/overview)
- [https://code-masterjung.tistory.com/83](https://code-masterjung.tistory.com/83)
