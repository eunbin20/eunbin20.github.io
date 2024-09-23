---
title: ApolloClient의 Cache Rehydration 구현하는 방법
date: 2023-01-14 11:58:47 +09:00
modified: 2023-01-14 11:58:47 +09:00
tags: [nextjs, ssr]
description: Next.js에서 ApolloClient의 캐시 Rehydration을 이용하여 SSR로 데이터를 불러오는 방법
image: https://github.com/user-attachments/assets/b44aa5fe-36e0-45a9-a426-4e7a8b6a170d
---

> 요즘 ApolloClient에서 캐시를 더 잘 활용할 수 있는 방법을 고민하고있다.<br/>
> 그러던 중 ApolloClient를 이용해 SSR + 무한 스크롤을 구현하는 방법을 소개하는 유투브 영상을 보게 되었다. 내가 기존에 구현했던 SSR과 약간 비슷했지만 내부적인 작동 방법에 대한 자세한 설명을 통해 이해가 부족했던 부분을 채울 수 있었다.
> 유튜브 영상의 더보기란에는 해당 영상에서 다룬 주제에 대한 블로그 포스팅도 볼 수 있었는데, 이 내용을 더 명확히 이해하면 아폴로에서 캐시를 다루는 방법에 대한 힌트를 얻을 수 있을 것 같아 해당 포스팅을 번역해보았다.

영상 링크: [https://www.youtube.com/watch?v=OtOeIXL1LRw](https://www.youtube.com/watch?v=OtOeIXL1LRw)<br/>
블로그 포스팅 링크: [https://developers.wpengine.com/blog/apollo-client-cache-rehydration-in-next-js](https://developers.wpengine.com/blog/apollo-client-cache-rehydration-in-next-js)

Apollo client는 쿼리 요청의 결과를 cache하는 다른 많은 GraphQL의 클라이언트들 중 유명한 라이브러리이다. 만약 당신이 클라이언트 렌더링으로 만들어진 웹 어플리케이션에서 모든 데이터를 브라우저에서 가져오도록 구현한 경우, 빈 Apollo Client 캐시에서 시작하여 GraphQL 요청이 실행된 이후에야 데이터를 채울 수 있을 것다.

하지만 만약 당신이 서버사이드 렌더링(SSR), 또는 정적 사이트 렌더링(SSG), 또는 ISR을 사용하고싶다면, 서버에서 어떻게 데이터를 받아올것인가? Apollo client를 이용해 데이터를 가져온 상태로 client를 만들어 다시 쿼리요청을 보낼 필요가 없게 할 수 있을까?

감사하게도, 서버로부터 데이터를 받아와서 Apollo client 캐시를 채우고, 해당 캐시 데이터를 client로 전송하여 브라우저에서 Apollo client의 캐시를 “rehydrate”하거나 초기화하는 데 사용할 수 있다. 캐시를 “rehydrate”하는 방식은 SSR/SSG/ISR(완전히 채워진 HTML 문서를 초기 페이지 로딩 시 렌더링할 수 있고, SEO에도 유리,,) 그 중에서도 가장 좋은 것은 그 초기에 fetch한 데이터를 Apollo client 에 여전히 가지고있다는 것이다. 이는 이미 가지고 있는 데이터를 다시 가져오기 위한 불필요한 네트워크 요청을 방지해준다.

이 포스팅에서 우리는 어떻게 Next.js앱에서 Apollo client cache를 통한 rehydraion을 수행할 수 있을지 알아본다.

# 프로젝트 세팅

캐시 rehydration을 세팅하기 전에 다음과 같은 준비를 완료해야한다.

1. 다음 리포지토리를 당신의 폴더에 클론받는다: [https://github.com/kellenmace/apollo-client-cache-rehydration-in-next-js](https://github.com/kellenmace/apollo-client-cache-rehydration-in-next-js)
2. 프로젝트에 필요한 라이브러리 다운로드를 위해 `npm install` (또는 `yarn`)을 실행한다.
3. 프로젝트의 root 경로에 `.env.local`파일을 만들고 다음과 같은 변수를 정의한다.
   `NEXT_PUBLIC_WORDPRESS_API_URL=http://my-local-site.local/graphql`
   WPGrapQL이 설치된 WordPress 사이트의 도메인과 [http://my-local-site.local을](http://my-local-site.xn--local-ei3w/) 교체한다.
4. `npm run dev`를 실행하여 어플리케이션을 로컬환경에서 구동한다.

# 접근 방식

Apollo 클라이언트를 통한 SSR(Server-side Rendering)은 다음과 같은 접근 방식을 따른다:

1. Apollo Client의 `getDataFromTree()`메소드를 사용하여 현재 페이지 내 모든 컴포넌트에서 사용되는 모든 쿼리들의 결과를 얻는다.
2. Apollo Client의 캐시를 추출하고 해당 데이터를 `window.APOLLO_STATE`
    전역 객체를 통해 클라이언트에 보낸다.
3. Apollo 클라이언트가 클라이언트에서 초기화되면 새로운 `InMemoryCache().restore()` 함수를 호출하여 `window.APOLLO_STATE` 객체를 서버로부터 받은 캐시 데이터로 사용한다.

Next.js에서 SSR 접근방식을 사용해도되지만, Apollo client에서 캐시 rehydration을 위해 추천되는 방식은 조금 다르다. 우리는 다음과 같은 방식을 사용할 것이다.

1. Next.js의 `getStaticProps()` 또는 `getServerSideProps()`함수를 사용하여 Apollo client 의 캐시를 채운다.
2. Apollo client의 캐시를 추출하여 현재 페이지의 `pageProps`로 넣는다.
3. 클라이언트에서 Apollo client가 초기화되면,  `new InMemoryCache().restore()` 함수를 호출하고 서버로부터 캐시된 데이터를 전달한다.

Apollo client가 어떻게 동작하는지에 대한 더 자세한 설명이 필요하다면, [공식적인 Next.js 예제 프로젝트](https://github.com/vercel/next.js/tree/canary/examples/with-apollo)에서 확인할 수 있다. 우리는 해당 예제 프로젝트와 같은 방식을 사용하되, JS가 아닌 Typescript를 사용할 것이고 데이터 요청 코드는WordPress와 WPGraphQL로 작성해볼 것이다.

# Apollo Client 설치 및 구성

각각의 Apollo Client의 [Next.js 예제 프로젝트](https://github.com/vercel/next.js/tree/canary/examples/with-apollo)에서는 `@apollo/client`와 `graphql` 은 이미 설치되어있다. 우리 프로젝트에서 활용할 `deepmerge`와 `lodash`패키지도 이미 설치되어있다.

client를 구성하기위해, 코드 에디터에서 `lib/apolloClient.ts`파일을 열어보자.

```tsx
import { useMemo } from "react";
import {
  ApolloClient,
  HttpLink,
  InMemoryCache,
  NormalizedCacheObject,
} from "@apollo/client";
import { relayStylePagination } from "@apollo/client/utilities";
import merge from "deepmerge";
import isEqual from "lodash/isEqual";

export const APOLLO_STATE_PROP_NAME = "__APOLLO_STATE__";

let apolloClient: ApolloClient<NormalizedCacheObject> | undefined;

function createApolloClient() {
  return new ApolloClient({
    ssrMode: typeof window === "undefined",
    link: new HttpLink({
      uri: process.env.NEXT_PUBLIC_WORDPRESS_API_URL, // Server URL (must be absolute)
      credentials: "same-origin", // Additional fetch() options like `credentials` or `headers`
    }),
    cache: new InMemoryCache({
      // typePolicies is not required to use Apollo with Next.js - only for doing pagination.
      typePolicies: {
        Query: {
          fields: {
            posts: relayStylePagination(),
          },
        },
      },
    }),
  });
}

export function initializeApollo(
  initialState: NormalizedCacheObject | null = null
) {
  const _apolloClient = apolloClient ?? createApolloClient();

  // If your page has Next.js data fetching methods that use Apollo Client, the initial state
  // gets hydrated here
  if (initialState) {
    // Get existing cache, loaded during client side data fetching
    const existingCache = _apolloClient.extract();

    // Merge the existing cache into data passed from getStaticProps/getServerSideProps
    const data = merge(initialState, existingCache, {
      // combine arrays using object equality (like in sets)
      arrayMerge: (destinationArray, sourceArray) => [
        ...sourceArray,
        ...destinationArray.filter((d) =>
          sourceArray.every((s) => !isEqual(d, s))
        ),
      ],
    });

    // Restore the cache with the merged data
    _apolloClient.cache.restore(data);
  }

  // For SSG and SSR always create a new Apollo Client
  if (typeof window === "undefined") return _apolloClient;

  // Create the Apollo Client once in the client
  if (!apolloClient) apolloClient = _apolloClient;

  return _apolloClient;
}

export function addApolloState(
  client: ApolloClient<NormalizedCacheObject>,
  pageProps: any
) {
  if (pageProps?.props) {
    pageProps.props[APOLLO_STATE_PROP_NAME] = client.cache.extract();
  }

  return pageProps;
}

export function useApollo(pageProps: any) {
  const state = pageProps[APOLLO_STATE_PROP_NAME];
  const store = useMemo(() => initializeApollo(state), [state]);
  return store;
}
```

각각의 함수들의 정의에 대해 알아보자.

### createApolloClient()

이 함수는 Apollo Client의 새로운 인스턴스를 생성하는 역할을 한다.

서버에서 수행되는 코드에서는 `ssrMode` 를 true로, 클라이언트에서 수행되는 코드에서는 false로 세팅한다.

`uri: process.env.NEXT_PUBLIC_WORDPRESS_API_URL` 명령줄에서는 `.env.local` 파일 내에 정의되어있는 GraphQL 엔드포인트를 Apollo Client에게 알려준다.

### initializeApollo()

이름에서 알 수 있듯이, 이 함수는 Apollo Client를 초기화한다. 이것은 getServerSideProps() 또는 getStaticProps()에서 전달받은 초기 데이터를 기존 클라이언트 측의 Apollo 캐시 데이터와 병합한 다음 새로 병합된 데이터 세트를 Apollo Client의 새 캐시로 설정한다.

### addApolloState()

이 함수는 현재 페이지에 대해 getStaticProps() 또는 getServerSideProps()에서 반환된 Pageprops를 가져와 Apollo의 캐시 데이터를 추가한다. 여기서 Next.js는 Apollo의 캐시 데이터와 이외의 페이지별 props를 페이지 컴포넌트로 전달해주는 역할을 한다.

### useApollo()

이 함수는 `initializeApollo()`를 호출하여 Apollo의 캐시 데이터가 추가된 Apollo Client의 인스턴스를 가져온다. 이 클라이언트는 궁극적으로 Apollo Client가 제공하는 Apollo Provider의 prop으로써 전달된다.

# ApolloProvider 렌더링

`_app.tsx` 내부에서, useApollo()를 호출하여 Apollo Client의 새 인스턴스를 가져와 현재 페이지에 대한 pageProps로 전달한다. 그리고 ApolloClient인스턴스는 ApolloProvider 컴포넌트의 prop으로 전달되어 전체 어플리케이션을 감싼다.

```tsx
import { AppContext, AppInitialProps } from "next/app";
import { ApolloProvider } from "@apollo/client";
import { useApollo } from "../lib/apolloClient";

import "../styles/globals.css";

function MyApp({ Component, pageProps }: AppContext & AppInitialProps) {
  const apolloClient = useApollo(pageProps);

  return (
    <ApolloProvider client={apolloClient}>
      <Component {...pageProps} />
    </ApolloProvider>
  );
}

export default MyApp;
```

# Page Component 구현하기

Next.js 페이지 구성 요소에서 이 Apollo Client 캐시 rehyedration을 어떻게 사용할 수 있는지 알아본다.

page/ssg.tsx를 보자. 해당 파일의 단순화된 버전은 다음과 같다.

```tsx
import { gql, useQuery } from "@apollo/client";
import { initializeApollo, addApolloState } from "../lib/apolloClient";

const POSTS_PER_PAGE = 10;

const GET_POSTS = gql`
  query getPosts($first: Int!, $after: String) {
    posts(first: $first, after: $after) {
      # etc.
    }
  }
`;

export default function SSG() {
  const { loading, error, data } = useQuery(GET_POSTS, {
    variables: {
      first: POSTS_PER_PAGE,
      after: null,
    },
  });

  // etc.
}

export async function getStaticProps(context: GetStaticPropsContext) {
  const apolloClient = initializeApollo();

  await apolloClient.query({
    query: GET_POSTS,
    variables: {
      first: POSTS_PER_PAGE,
      after: null,
    },
  });

  return addApolloState(apolloClient, {
    props: {},
  });
}
```

Next.js의 getStaticProps() 함수 내에서 우리는 InitializeApollo()를 호출하여 GraphQL 요청을 수행하는 데 사용할 수 있는 Apollo Client의 인스턴스를 가져온다.

그런 다음 `apolloClient.query()` 메소드를 호출하는데, SSG 페이지 내에서 useQuery() 훅을 호출할 때 사용되는 것과 정확히 동일한 쿼리 및 변수를 전달한다. 현재 코드에서 apolloClient.query()의 반환값도 사용하지 않는다는 것에 주목하라. 우리는 이 반환값을 가져다 사용할 필요도 없다. 단지 query 메소드로 실행된 데이터로 ApolloClient의 캐시를 채운다.

SSG 페이지 구성 요소가 브라우저에서 렌더링되고 useQuery() 훅이 호출되면 Apollo Client는 실행 중인 쿼리의 데이터가 이미 캐시에 존재함을 "확인"하고 네트워크 요청 대신 해당 데이터를 사용한다. 이렇게 하면 페이지의 내용이 즉시 렌더링된다. 브라우저에서 http://localhost:3000/ssg를 방문하여 DevTools의 Network 탭을 열고 페이지를 새로 고쳐서 확인해보면 클라이언트 측 GraphQL 요청은 실행되고있지 않은 것을 확인할 수 있다.

Next.js의 getStaticProps() 함수에 대한 자세한 내용은 [여기](https://nextjs.org/docs/basic-features/data-fetching#getstaticprops-static-generation)에서 확인할 수 있다.

## SSR 페이지

page/ssr.tsx를 열고 `[http://localhost:3000/ssr](http://localhost:3000/ssr)`에 접속한다.

getStaticProps 대신 getServerSideProps()를 사용하기 때문에 Next.js가 모든 요청에서 pageProps를 전달한다는 점을 제외하면 이 파일은 ssg.tsx와 정확히 동일하게 동작한다. 이 방법 역시 DevTools의 Network 탭을 열고 페이지를 새로 고쳐서 확인해보면 클라이언트 측 GraphQL 요청은 실행되고있지 않은 것을 확인할 수 있다.

Next.js의 getServerSideProps() 함수에 대한 자세한 내용은 [여기](https://nextjs.org/docs/basic-features/data-fetching/overview#getserversideprops-server-side-rendering%EC%97%90%EC%84%9C)에서 확인할 수 있다.

### SSG + CSR 혼합 페이지

마지막으로 page/blos.tsx를 연다. WordPress 백엔드에 11개 이상의 게시물이 작성되었는지 확인한 후 [http://localhost:3000/blog](http://localhost:3000/blog%EB%A5%BC)에 접속해보면 getStaticProps() 내부에서 Apollo Client 쿼리가 실행되어 블로그 게시물 10개의 초기 목록을 가져와 페이지가 로드된 즉시 렌더링되는 것을 확인할 수 있다.

WordPress 백엔드에 더 많은 게시물이 있는 경우 사용자에게 추가 로드 버튼이 표시된다. 이 버튼을 클릭하면 Apollo Client의 fetchMore() 함수가 호출되며, 이 함수는 다음 5개의 게시물을 가져오기 위해 네트워크 요청을 실행하고 목록에 추가한다. 모든 게시물이 로드되면 ✅ All posts loaded(모든 게시물이 로드됨) 메시지가 표시된다.

blog.tsx 페이지는 서버에서 일부 데이터를 가져와 Apollo의 캐시를 채울 수 있는 방법을 보여준다. 또한 페이지가 로드된 즉시 데이터가 나타난다. 이후 요청에서는 클라이언트 측 요청을 통해 추가 데이터를 가져오도록 요청을 실행하여 아폴로의 캐시에 추가한다.

# 정리

위 글이 Next.js에서 SSG/SSR/ISR 데이터를 fetch하기 위해 Apollo Client를 활용하는 데 도움이 되기를 바란다.
