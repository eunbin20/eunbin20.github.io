---
title: ApolloClient onError๋ฅผ ํ์ฉํ Token Error Handling ๐ง๐ปโโ๏ธ
date: 2022-05-21 00:00
modified: 2022-05-21 00:01
tags: [apollo-client]
description: ApolloClient์์ ํ ํฐ์ด ์ ํจํ์ง ์์ ๊ฒฝ์ฐ ๋ฐ์ํ๋ ์๋ฌ๋ฅผ ํธ๋ค๋งํ๋ ๋ฐฉ๋ฒ
image: https://user-images.githubusercontent.com/43979066/158075767-1f23a59a-7726-4140-bd04-8a0301065452.png
---

> ApolloClient์ `onError` ๋ฉ์๋๋ฅผ ํตํด graphql ์๋ฒ, ๋๋ network์์ ๋ฐ์ํ ์๋ฌ๋ฅผ ์บ์นํ์ฌ ๊ฐ๊ฐ์ ์๋ฌ ์ํฉ์ ๋ง๋ ์ ์ ํ ํธ๋ค๋ง์ ํ  ์ ์๋ค.<br/>
> ์ด๋ฒ ํฌ์คํ์์๋ ํ ํฐ์ด ์ ํจํ์ง ์์ ๊ฒฝ์ฐ ๋ฐ์ํ๋ Unauthorized ์๋ฌ๋ฅผ ํธ๋ค๋งํ๋ ๋ฐฉ๋ฒ์ ์์๋ณธ๋ค.

์ฐ์ , Apollo Client ์ธ์คํด์ค๋ฅผ ๋ง๋ ๋ค.

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

์๋ฒ๋ก ํด๋ผ์ด์ธํธ๊ฐ ๊ฐ์ง๊ณ  ์๋ token์ authLink๋ฅผ ์ถ๊ฐํ์ฌ ํค๋์ ์ค์ด ๋ณด๋ด๋ ๋ก์ง์ ์์ฑํด๋ณด์.

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

authLink๋ฅผ ํตํด localStorage์ ๋ด๊ธด token์ด header์ ๋ด๊ฒจ ์๋ฒ๋ก ์ ์ก๋  ๊ฒ์ด๋ค.

์๋ฒ์์ ๊ตฌํ๋ ์ฌ์ฉ์ ์ธ์ฆ ์๋๋ฆฌ์ค์ ๋ฐ๋ผ ๋ค๋ฅด๊ฒ ์ง๋ง,

1. accessToken๊ณผ refreshToken์ ์ด์ฉํด ์ฌ์ฉ์์ ํ ํฐ์ ๊ฒ์ฆํ๋ ์ํฉ์์ ์ฌ์ฉ์์ accessToken์ด ๋ง๋ฃ๋ ๊ฒฝ์ฐ,
2. ๋๋ ๋ก๊ทธ์ธ์ ํ๊ธฐ ์ ์๋ ๋ฏธ๋ก๊ทธ์ธ ์ฌ์ฉ์(anonymous user)๋ก์ ํ ํฐ ๋ฐ๊ธ์ด ํ์ํ ๊ฒฝ์ฐ

์ฌ์ฉ์๊ฐ ์น์ฌ์ดํธ์ ์ง์ํ๋ฉด ์๋ฒ๋ก๋ถํฐ `Unauthorized` ์๋ฌ๊ฐ ๋ฐ์ํ  ๊ฒ์ด๋ค.

<mark>์ด๋ฌํ ๊ฒฝ์ฐ apollo client์ <strong>onError</strong> ๋ฉ์๋๋ฅผ ์ฌ์ฉํ๋ฉด `Unauthorized` ์๋ฌ ๋ฐ์ ์ ํ ํฐ ์ฌ๋ฐ๊ธ์ ํตํด ์ ๋ฌธ์ ๋ฅผ ํด๊ฒฐํ  ์ ์๋ค.</mark>

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

graphQL ์์ฒญ ์ ๋ฐ์ํ๋ ์๋ฌ๋ ํฌ๊ฒ graphQL error, network error๋ก ๋๋ ์ ์๋ค.<br/><br/>
apollo client์์๋ ์๋ฌ๋ฅผ ํธ๋ค๋งํ๊ธฐ ์ํ link๋ก onError์ RetryLink ๊ฐ ์๋๋ฐ
๊ณต์๋ฌธ์์์๋ <u>graphQL error๋ onError ๋ก, network error๋ RetryLink๋ก ๊ฐ๊ฐ ํธ๋ค๋ง</u>ํ  ๊ฒ์ ๊ถ์ฅํ๊ณ  ์๋ค.<br/><br/>
์๋ฒ์์์ ํ ํฐ ์ ํจ์ฑ ๊ฒ์ฌ์์ ํต๊ณผํ์ง ๋ชปํ ์์ฒญ์์ ๋ฐ์ํ๋ ์๋ฌ๋ graphQL์๋ฒ์์ ๋ฐ์์ํค๋ graphQL Error์ด๋ฏ๋ก onError๋ฅผ ์ด์ฉํด ์๋ฌ๋ฅผ ์ฒ๋ฆฌํด์ฃผ์๋ค.

### onError๋

- onError๋ฅผ ์ด์ฉํด graphQL ์๋ฒ๋ก ๋ฐ์ดํฐ ์์ฒญ ์ค์ ๋ฐ์ํ ์๋ฌ๋ฅผ ์บ์นํ์ฌ ์ ์ ํ ํธ๋ค๋ง ๋ก์ง์ ์ถ๊ฐํด์ค ์ ์๋ค.
- ์์ฒญ์ ๋ค์ ๋ณด๋ด๊ธฐ ์ํด forwardํจ์๋ฅผ ํธ์ถํด์ผํ๋ค.
- ๋ง์ฝ ์ฌ์์ฒญ์์๋ ์๋ฌ๊ฐ ๋ฐ์ํ  ๊ฒฝ์ฐ, onError๋ ๋๋ค์ ์์ฒญ์ ๋ณด๋ด์ง๋ ์๋๋ค. (์ด ๊ฒฝ์ฐ ๋ฌดํ ๋ฃจํ์ ๋น ์ง ์ ์๊ธฐ ๋๋ฌธ) ์ฆ, ์คํจํ operation ๋น ํ ๋ฒ๋ง ์ฌ์์ฒญํ๋ค.

<br/>

### โ๏ธProblem

getNewToken()์ ๋น๋๊ธฐ์ ์ผ๋ก ์ํ๋๋ ํจ์์ด๋ค.(์๋ง ๋๋ถ๋ถ์ ๊ฒฝ์ฐ ๊ทธ๋ด ๊ฒ์ด๋ผ ์๊ฐํ๋ค.)

๊ทธ๋์ async ๋ promise์ ๊ฐ์ ๋ฌธ๋ฒ์ ์ถ๊ฐํ์ฌ ํด๊ฒฐํ๊ณ ์ํ์๋๋ฐ, onError์ ์ฝ๋ฐฑํจ์์ async ํค์๋๋ฅผ ์ถ๊ฐํ๋๋ ๋ค์๊ณผ ๊ฐ์ ์๋ฌ๊ฐ ๋ฐ์ํ๋ค.

<figure>
  <img src="https://user-images.githubusercontent.com/43979066/169671820-9ce545a4-95f4-45d9-96b4-0e5c34a3c713.png" alt="" width="100%">
</figure>

**ErrorHandler์ ๋ฐํ๊ฐ์ <mark>Observable</mark> ๊ฐ์ฒด์ด๊ฑฐ๋ ์์ด์ผ ํ๋ค**(void | Observable). <br/>
๊ทธ๋ฐ๋ฐ async ํค์๋๊ฐ ๋ถ์ ํจ์๋ Promise๋ฅผ ๋ฐํํ๊ฒ ๋๋ฏ๋ก ์์๊ฐ์ ์๋ฌ๊ฐ ๋ฐ์ํ๋ค.

..Observable์ ๋ฌด์์ธ๊ฐ๐ค

๋จผ์  RxJS๋ถํฐ ์ดํด๋ณด์.

**RxJS**

- RxJS๋ ํ์ผ ์ฝ๊ธฐ, data fetching, ํค ์๋ ฅ ๋ฑ์ ์ด๋ฒคํธ ์์ค๋ฅผ ๋น๋๊ธฐ์ ์ผ๋ก ์ฒ๋ฆฌํ๊ธฐ ์ํ JS์ ํ์ฅ ๋ผ์ด๋ธ๋ฌ๋ฆฌ์ด๋ค. ์ฝ๋ฐฑ์ด๋ ํ๋ก๋ฏธ์ค๋ฅผ ๋์ฒดํ  ์ ์๋ค.
- <mark>RxJS๋ ์ด๋ฒคํธ ์คํธ๋ฆผ์ observable ์ด๋ผ๋ ๊ฐ์ฒด๋ก ํํํ ํ ๋น๋๊ธฐ๋ก ๋์ํ๋ ๋ก์ง์ ๊ตฌํํ๋ค.</mark>
- observable ๊ฐ์ฒด๋ **๋ฐ์ดํฐ ์คํธ๋ฆผ์ ๋ง๋๋ ๋ฐ์ดํฐ ์์ฐ์**์ด๋ค.
- Apollo ๋ Observable ๊ตฌํ์ ์ํด [zen-observable](https://github.com/zenparsing/zen-observable)์ด๋ผ๋ ํจํค์ง๋ฅผ ๋ด๋ถ์ ์ผ๋ก ์ฌ์ฉํ๊ณ  ์๋ค.

์ฐ์  Observable์ apollo link์์ ๋น๋๊ธฐ์ ์ธ ๋์์ ์ฒ๋ฆฌํ๋ ๊ฐ์ฒด๋ก, getNewToken()์ ๊ฒฐ๊ณผ ๋ฐ์ดํฐ ์ฒ๋ฆฌ๋ฅผ ์ํ ๊ฐ์ฒด๋ผ๊ณ  ์ดํดํ๋ฉด ๋  ๊ฒ ๊ฐ๋ค.

### ๐กSolution

apollo-link ํจํค์ง์์๋ promise๋ฅผ ์ด์ฉํด observable ๊ฐ์ฒด๋ฅผ ์์ฑํด์ฃผ๋ `fromPromise`๋ผ๋ ๋ฉ์๋๋ฅผ ์ ๊ณตํ๋ค.

**fromPromise**

- from promise to observable์ ์๋ฏธ์ธ ๊ฒ ๊ฐ๋ค.(์๋ง๋..?)
- promise ๊ฐ์ฒด๋ฅผ ์ธ์๋ก ์ ๋ฌํ๋ฉด observable ๊ฐ์ฒด๋ฅผ ๋ฐํํ๋ค.

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
                // ์ด์  header๋ฅผ ํ์ฉํด ์ด์  ์์ฒญ์ context๋ฅผ ์ฌ์์ฑ
                operation.setContext({
                  headers: {
                    ...oldHeaders,
                    token: token || "",
                  },
                });

                return forward(operation); // ์คํจํ ์ด์  ์์ฒญ ๋ค์ ๋ณด๋ด๊ธฐ
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

[filter](https://rxjs.dev/api/operators/filter): ๊ฒฐ๊ณผ๊ฐ ์ค ์ฝ๋ฐฑํจ์๋ฅผ ๋ง์กฑํ๋ ๊ฒฐ๊ณผ๊ฐ๋ง ํํฐ๋ง<br/>
[flatMap(mergeMap)](https://rxjs.dev/api/operators/mergeMap): ๋ฐํ๋ observable ๊ฐ์ฒด๋ค์ ๋ฐ์ดํฐ ์คํธ๋ฆผ์ ๋ณํฉํ์ฌ ๊ฒฐ๊ณผ๊ฐ์ ๋ด๋ณด๋ธ๋ค.

##### **์ต์ข ApolloClient Provider ์ฝ๋**

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
                // ์ด์  header๋ฅผ ํ์ฉํด ์ด์  ์์ฒญ์ context๋ฅผ ์ฌ์์ฑ
                operation.setContext({
                  headers: {
                    ...oldHeaders,
                    token: token || "",
                  },
                });

                return forward(operation); // ์คํจํ ์ด์  ์์ฒญ ๋ค์ ๋ณด๋ด๊ธฐ
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

ApolloClient์์ link๋ฅผ ์์ ๊ฐ์ด from์ผ๋ก ์ฐ๊ฒฐํด์ฃผ๋ฉด **link chain**์ด ํ์ฑ๋๋ค.

### Link Chain

<figure>
  <img src="https://user-images.githubusercontent.com/43979066/169672032-be860d16-5a4b-43ca-bd7f-f19d2137c4f1.png" alt="" width="100%">
</figure>

- `from`๋ฉ์๋๋ ์ฌ๋ฌ ๊ฐ์ ๋งํฌ๋ฅผ ๊ฒฐํฉํ์ฌ ํ๋์ link chain์ ํ์ฑํด์ฃผ๋ ๋ฉ์๋์ด๋ค. httpLink๋ ์ข๋ฃ ๋งํฌ([terminating link](https://www.apollographql.com/docs/react/api/link/introduction#the-terminating-link))๋ก์ ๋งํฌ ์ฒด์ด๋์ ๊ฐ์ฅ ์ข๋จ์ ์์นํ์ฌ graphQL ์๋ฒ์ ์์ฒญ์ ๋ณด๋ด๊ณ  ์คํ ๊ฒฐ๊ณผ๋ฅผ ๋ฐ๋ link์ด๋ค.
- link chain์ผ๋ก ์ฐ๊ฒฐ๋์ด์๋ link๋ค์ `forward` ๋ฅผ ํตํด ์ฒด์ธ ์์ ๋ค์ ๋งํฌ๋ฅผ ํธ์ถํ๋ค.

# ๊ฒฐ๋ก 

๊ธฐ์กด์๋ ApolloClientProvider๊ฐ ์ฌ์ฉ๋๋ ์ปดํฌ๋ํธ์์ token์ ๊ฒ์ฌํด token์ด ์กด์ฌํ๋ ๊ฒฝ์ฐ์๋ง ์ปดํฌ๋ํธ๋ฅผ ๋ณด์ฌ์ฃผ๋ ๋ฐฉ์์ผ๋ก ๊ตฌํํ์๋ค. ๊ทธ๋ฌ๋ onError ๋ฉ์๋๋ฅผ ์ฌ์ฉํ๊ฒ ๋๋ฉด์ ๋ ๋ชํํ๊ฒ ํ ํฐ ์๋ฌ๊ฐ ๋ฐ์ํ์ ๊ฒฝ์ฐ์ ๋ํ ํธ๋ค๋ง์ ํด์ค ์ ์๊ฒ ๋์๋ค.
๋ํ ํ ํฐ ์๋ฌ ๋ง๊ณ ๋ ๋ฐ์ดํฐ ์์ฒญ ์ค ๋ฐ์ํ๋ graphQL error, network error๋ค๋ ํ ๊ณณ์์ ํธ๋ค๋งํด์ค ์ ์๊ฒ ๋์ด ์ถํ์ ์๋ฌ ๋ชจ๋ํฐ๋ง ์์คํ์ ๋์ํ๊ฒ ๋  ๊ฒฝ์ฐ ๋ฌธ์ ๋ฅผ ์ฝ๊ฒ ํด๊ฒฐํ  ์ ์์ ๊ฒ ๊ฐ๋ค.<br/>

apollo link๊ฐ link chain ๋ฐฉ์์ผ๋ก ๊ตฌํ๋์ด์๋ค๋ ๊ฒ ๋๋ฌด ์ ๊ธฐํ๋ค. ์ฌ๊ธฐ์ ๋น๋๊ธฐ ๋์ ์ฒ๋ฆฌ๋ฅผ ์ํด ์ฌ์ฉ๋ observable๊ฐ์ฒด์ ๋ํด์๋ ์ข ๋ ์กฐ์ฌํด๋ณด๊ณ  rxJS์ ๋ํด ์ข ๋ ์์๋ณด๊ณ ์ถ๋ค.<br/>

##### _Resources_

- [How to execute an async fetch request and then retry last failed request?](https://stackoverflow.com/questions/50965347/how-to-execute-an-async-fetch-request-and-then-retry-last-failed-request)<br/>
- [apollo-link/packages/apollo-link-error at master ยท apollographql/apollo-link](https://github.com/apollographql/apollo-link/tree/master/packages/apollo-link-error#retrying-failed-requests)<br/>
- [Apollo GraphQL : Async Access Token Refresh](https://able.bio/AnasT/apollo-graphql-async-access-token-refresh--470t1c8)
