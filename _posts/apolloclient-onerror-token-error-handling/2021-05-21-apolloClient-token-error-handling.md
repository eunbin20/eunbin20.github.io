---
title: ApolloClient onError를 활용한 Token Error Handling 🧙🏻‍♀️
date: 2022-05-21 00:00
modified: 2022-05-21 00:01
tags: [apollo-client]
description: ApolloClient에서 토큰이 유효하지 않을 경우 발생하는 에러를 핸들링하는 방법
image: https://user-images.githubusercontent.com/43979066/158075767-1f23a59a-7726-4140-bd04-8a0301065452.png
---

> ApolloClient의 `onError` 메소드를 통해 graphql 서버, 또는 network에서 발생한 에러를 캐치하여 각각의 에러 상황에 맞는 적절한 핸들링을 할 수 있다.<br/>
> 이번 포스팅에서는 토큰이 유효하지 않을 경우 발생하는 Unauthorized 에러를 핸들링하는 방법을 알아본다.

우선, Apollo Client 인스턴스를 만든다.

```jsx
import { ApolloClient, createHttpLink, InMemoryCache } from "@apollo/client";
import { setContext } from "@apollo/client/link/context";

const token = localStorage.getItem("token");

const httpLink = createHttpLink({
  uri: "http://localhost:4000/graphql",
});

export const client = new ApolloClient({
  link: httpLink,
  cache: new InMemoryCache(),
});
```

서버로 클라이언트가 가지고 있는 token을 authLink를 추가하여 헤더에 실어 보내는 로직을 작성해보자.

```jsx
import { ApolloClient, createHttpLink, InMemoryCache } from "@apollo/client";
import { setContext } from "@apollo/client/link/context";

const httpLink = createHttpLink({
  uri: "http://localhost:4000/graphql",
});

const authLink: ApolloLink = setContext(async (_, { headers }) => {
  const token = localStorage.getItem("token");

  return {
    headers: {
      ...headers,
      token: token,
    },
  };
});

export const client = new ApolloClient({
  link: from(authLink, httpLink),
  cache: new InMemoryCache(),
});
```

authLink를 통해 localStorage에 담긴 token이 header에 담겨 서버로 전송될 것이다.

서버에서 구현된 사용자 인증 시나리오에 따라 다르겠지만,

1. accessToken과 refreshToken을 이용해 사용자의 토큰을 검증하는 상황에서 사용자의 accessToken이 만료된 경우,
2. 또는 로그인을 하기 전에도 미로그인 사용자(anonymous user)로서 토큰 발급이 필요한 경우

사용자가 웹사이트에 진입하면 서버로부터 `Unauthorized` 에러가 발생할 것이다.

<mark>이러한 경우 apollo client의 <strong>onError</strong> 메소드를 사용하면 `Unauthorized` 에러 발생 시 토큰 재발급을 통해 위 문제를 해결할 수 있다.</mark>

```jsx
import { ApolloClient, createHttpLink, InMemoryCache } from "@apollo/client";
import { setContext } from "@apollo/client/link/context";

// ...

const errorLink = onError(
  ({ graphQLErrors, networkError, operation, forward }) => {
    if (graphQLErrors) {
      for (let error of graphQLErrors) {
        switch (error.extensions.code) {
          case "UNAUTHENTICATED":
            const oldHeaders = operation.getContext().headers;

            operation.setContext({
              headers: {
                ...oldHeaders,
                authorization: getNewToken(),
              },
            });
        }
      }
    }

    if (networkError) {
      console.log(`[Network error]: ${networkError}`);
    }
  }
);

export const client = new ApolloClient({
  link: from(errorLink, authLink, httpLink),
  cache: new InMemoryCache(),
});
```

<figure>
  <figcaption>code from <a href="https://www.apollographql.com/docs/react/data/error-handling#retrying-operations">apollo-client doc</a></figcaption>
</figure>

graphQL 요청 시 발생하는 에러는 크게 graphQL error, network error로 나눌 수 있다.<br/><br/>
apollo client에서는 에러를 핸들링하기 위한 link로 onError와 RetryLink 가 있는데
공식문서에서는 <u>graphQL error는 onError 로, network error는 RetryLink로 각각 핸들링</u>할 것을 권장하고 있다.<br/><br/>
서버에서의 토큰 유효성 검사에서 통과하지 못한 요청에서 발생하는 에러는 graphQL서버에서 발생시키는 graphQL Error이므로 onError를 이용해 에러를 처리해주었다.

### onError란

- onError를 이용해 graphQL 서버로 데이터 요청 중에 발생한 에러를 캐치하여 적절한 핸들링 로직을 추가해줄 수 있다.
- 요청을 다시 보내기 위해 forward함수를 호출해야한다.
- 만약 재요청시에도 에러가 발생할 경우, onError는 또다시 요청을 보내지는 않는다. (이 경우 무한 루프에 빠질 수 있기 때문) 즉, 실패한 operation 당 한 번만 재요청한다.

<br/>

### ❗️Problem

getNewToken()은 비동기적으로 수행되는 함수이다.(아마 대부분의 경우 그럴 것이라 생각한다.)

그래서 async 나 promise와 같은 문법을 추가하여 해결하고자하였는데, onError의 콜백함수에 async 키워드를 추가했더니 다음과 같은 에러가 발생했다.

<figure>
  <img src="https://user-images.githubusercontent.com/43979066/169671820-9ce545a4-95f4-45d9-96b4-0e5c34a3c713.png" alt="" width="100%">
</figure>

**ErrorHandler의 반환값은 <mark>Observable</mark> 객체이거나 없어야 한다**(void | Observable). <br/>
그런데 async 키워드가 붙은 함수는 Promise를 반환하게 되므로 위와같은 에러가 발생했다.

..Observable은 무엇인가🤔

먼저 RxJS부터 살펴보자.

**RxJS**

- RxJS는 파일 읽기, data fetching, 키 입력 등의 이벤트 소스를 비동기적으로 처리하기 위한 JS의 확장 라이브러리이다. 콜백이나 프로미스를 대체할 수 있다.
- <mark>RxJS는 이벤트 스트림을 observable 이라는 객체로 표현한 후 비동기로 동작하는 로직을 구현한다.</mark>
- observable 객체란 **데이터 스트림을 만드는 데이터 생산자**이다.
- Apollo 는 Observable 구현을 위해 [zen-observable](https://github.com/zenparsing/zen-observable)이라는 패키지를 내부적으로 사용하고 있다.

우선 Observable은 apollo link에서 비동기적인 동작을 처리하는 객체로, getNewToken()의 결과 데이터 처리를 위한 객체라고 이해하면 될 것 같다.

### 💡Solution

apollo-link 패키지에서는 promise를 이용해 observable 객체를 생성해주는 `fromPromise`라는 메서드를 제공한다.

**fromPromise**

- from promise to observable의 의미인 것 같다.(아마도..?)
- promise 객체를 인자로 전달하면 observable 객체를 반환한다.

```jsx
const errorLink = onError(
  ({ graphQLErrors, networkError, operation, forward }) => {
    if (graphQLErrors) {
      for (let error of graphQLErrors) {
        switch (error.extensions.code) {
          case "UNAUTHENTICATED":
            return fromPromise(getNewToken())
              .filter((value) => {
                return Boolean(value);
              })
              .flatMap((token) => {
                setUserToken({ isKeepLogin: false, token });
                const oldHeaders = operation.getContext().headers;
                // 이전 header를 활용해 이전 요청의 context를 재생성
                operation.setContext({
                  headers: {
                    ...oldHeaders,
                    token: token || "",
                  },
                });

                return forward(operation); // 실패한 이전 요청 다시 보내기
              });
        }
      }
    }

    if (networkError) {
      console.log(`[Network error]: ${networkError}`);
    }
  }
);
```

[filter](https://rxjs.dev/api/operators/filter): 결과값 중 콜백함수를 만족하는 결과값만 필터링<br/>
[flatMap(mergeMap)](https://rxjs.dev/api/operators/mergeMap): 반환된 observable 객체들의 데이터 스트림을 병합하여 결과값을 내보낸다.

##### **최종 ApolloClient Provider 코드**

```jsx
import { ApolloClient, createHttpLink, InMemoryCache } from "@apollo/client";
import { setContext } from "@apollo/client/link/context";

const httpLink = createHttpLink({
  uri: "http://localhost:4000/graphql",
});

const authLink: ApolloLink = setContext(async (_, { headers }) => {
  const token = localStorage.getItem("token");

  return {
    headers: {
      ...headers,
      token: token,
    },
  };
});

const errorLink = onError(
  ({ graphQLErrors, networkError, operation, forward }) => {
    if (graphQLErrors) {
      for (let error of graphQLErrors) {
        switch (error.extensions.code) {
          case "UNAUTHENTICATED":
            return fromPromise(getNewToken())
              .filter((value) => {
                return Boolean(value);
              })
              .flatMap((token) => {
                setUserToken({ isKeepLogin: false, token });
                const oldHeaders = operation.getContext().headers;
                // 이전 header를 활용해 이전 요청의 context를 재생성
                operation.setContext({
                  headers: {
                    ...oldHeaders,
                    token: token || "",
                  },
                });

                return forward(operation); // 실패한 이전 요청 다시 보내기
              });
        }
      }
    }

    if (networkError) {
      console.log(`[Network error]: ${networkError}`);
    }
  }
);

export const client = new ApolloClient({
  link: from(errorLink, authLink, httpLink),
  cache: new InMemoryCache(),
});
```

ApolloClient에서 link를 위와 같이 from으로 연결해주면 **link chain**이 형성된다.

### Link Chain

<figure>
  <img src="https://user-images.githubusercontent.com/43979066/169672032-be860d16-5a4b-43ca-bd7f-f19d2137c4f1.png" alt="" width="100%">
</figure>

- `from`메소드는 여러 개의 링크를 결합하여 하나의 link chain을 형성해주는 메서드이다. httpLink는 종료 링크([terminating link](https://www.apollographql.com/docs/react/api/link/introduction#the-terminating-link))로서 링크 체이닝의 가장 종단에 위치하여 graphQL 서버에 요청을 보내고 실행 결과를 받는 link이다.
- link chain으로 연결되어있는 link들은 `forward` 를 통해 체인 상의 다음 링크를 호출한다.

# 결론

기존에는 ApolloClientProvider가 사용되는 컴포넌트에서 token을 검사해 token이 존재하는 경우에만 컴포넌트를 보여주는 방식으로 구현했었다. 그러나 onError 메서드를 사용하게 되면서 더 명확하게 토큰 에러가 발생했을 경우에 대한 핸들링을 해줄 수 있게 되었다.
또한 토큰 에러 말고도 데이터 요청 중 발생하는 graphQL error, network error들도 한 곳에서 핸들링해줄 수 있게 되어 추후에 에러 모니터링 시스템을 도입하게 될 경우 문제를 쉽게 해결할 수 있을 것 같다.<br/>

apollo link가 link chain 방식으로 구현되어있다는 게 너무 신기했다. 여기서 비동기 동작 처리를 위해 사용된 observable객체에 대해서도 좀 더 조사해보고 rxJS에 대해 좀 더 알아보고싶다.<br/>

##### _Resources_

- [How to execute an async fetch request and then retry last failed request?](https://stackoverflow.com/questions/50965347/how-to-execute-an-async-fetch-request-and-then-retry-last-failed-request)<br/>
- [apollo-link/packages/apollo-link-error at master · apollographql/apollo-link](https://github.com/apollographql/apollo-link/tree/master/packages/apollo-link-error#retrying-failed-requests)<br/>
- [Apollo GraphQL : Async Access Token Refresh](https://able.bio/AnasT/apollo-graphql-async-access-token-refresh--470t1c8)
