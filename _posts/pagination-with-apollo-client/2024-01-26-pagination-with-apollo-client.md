---
title: ApolloClient로 페이지네이션 구현하기
date: 2024-01-26 00:00
modified: 2024-01-26 00:00
tags: [apollo_client, graphql, pagination]
description: 아폴로 클라이언트에서 페이지네이션을위해 제공하는 API 알아보기
image: https://user-images.githubusercontent.com/43979066/214864762-bf268ce1-f0b6-4051-b0c5-80092c05c30f.png
---

> ApolloClient를 이용해 무한 스크롤 리스트를 구현하기위해 공식문서를 참고하여 공부한 내용을 정리한 문서입니다.
> 제가 개인적으로 이해한 내용을 한국어로 풀어서 정리한거라 의역, 또는 오역이 있을 수 있으니 정확한 내용은 [공식문서](https://www.apollographql.com/docs/react/pagination/core-api)를 참고해주세요!

- [_Resources_](#resources)

Apollo Server에서 어떤 전략으로 리스트 필드를 내려주던간에 Apollo Client에서는 리스트를 효율적으로 가져오기 위해 해당 쿼리에 다음과 같은 작업을 해주어야한다.

- 다음 페이지 리스트를 요청할 때 `fetchMore` 함수 사용하기
- Apollo cache 내에 있는 단일 리스트에 서버로부터 가져온 각각의 리스트 병합하기

## fetchMore 함수

페이지네이션을 구현하기 위해서는 추가적인 페이지의 결과를 Graphql 서버로부터 받아야 한다. Apollo Client에서는 이를 구현하기 위해 `fetchMore`함수를 사용하는 것을 권장한다.

- `fetchMore`함수는 `client.watchQuery` 의 반환값인 'ObservableQuery'의 한 종류로서, `useQuery` hook을 통해서도 사용할 수 있다.

```jsx
export const LIST_CHAPTER_COMMENT = gql`
  query ListChapterComment(
    $storyId: Int
    $chapterId: Int
    $parentId: Int
    $page: Int!
    $pageSize: Int
  ) {
    listChapterComment(
      data: {
        storyId: $storyId
        chapterId: $chapterId
        parentId: $parentId
        page: $page
        pageSize: $pageSize
      }
    ) {
      page
      totalCount
      list {
        id
        # ...
      }
    }
  }
`;

const CommentWithData = () => {
  const { data, loading, fetchMore } = useQuery(LIST_CHAPTER_COMMENT, {
    variables: {
      storyId: 100,
      chapterId: 200,
      parentId: 300,
      page: 1,
      pageSize: 10,
    },
  });

  // 아래 예제에서 계속
};
```

일반적으로 유저가 “더보기”버튼을 클릭하거나 화면의 최하단까지 스크롤하는 등의 액션이 발생했을 때 `fetchMore` 함수를 호출하여 추가적인 데이터를 불러올 것이다.

기본적으로 `fetchMore` 함수는 초기에 보냈던 쿼리 매개변수(variables)와 동일한 형태와 값으로 요청을 보낸다. 그래서 새로운 요청에서 필요한 매개변수만 추가적으로 보내주면된다.

```jsx
const CommentWithData = () => {
	const { data, loading, fetchMore } = useQuery(LIST_CHAPTER_COMMENT, {
		variables: {
      storyId: 100,
      chapterId: 200,
      parentId: 300,
      page: 1,
      pageSize: 10,
    }
  });

  if (loading) return 'Loading...';

  return (
    <CommentList
      data={data.listChapterComment}
      onLoadMore={
        vairables: {
          page: data.listChapterComment.page + 1
        }
      }
    />
  );
};
```

여기서 우리는 page 매개변수에 `data.listChapterComment.page + 1` 를 할당하여 현재 캐시되어있는 데이터의 다음 페이지의 리스트를 불러오도록 설정하였다. 우리가 넣어준 `variables` 는 기존의 `variables` 변수와 병합되어 결과적으로 storyId, chapterId, parentId, pageSize 등의 변수는 기존 요청 그대로 전달될 것이다.

추가적으로 , `variables` 외에도 `fetchMore` 함수를 통해 실행할 쿼리를 기존 필드와 완전히 다른 형태로 제공할 수도 있다. 이 기능은 `fetchMore` 에서 가져올 데이터가 특정한 단일 필드만 가져와야하지만, 초기에 불러오는 쿼리는 관련없는 필드들만 포함하고있는 경우에 유용하다(`fetchMore` 에서 가져올 쿼리와 초기 요청에서 가져올 쿼리의 형태가 완전히 다른 형태인 경우 유용하게사용할 수 있을 것 같다.)

**이제 fetchMore 함수는 준비가 되었다. 하지만 아직 끝나지 않았다!** 아직 캐시 저장소에는 추가적으로 불러온 쿼리의 결과와 기존 캐시 데이터가 병합되어있지 않다. 대신 각각 다른 결과물로서 완전히 분리되어 저장되어있다. 이를 병합시켜주기위해서는 페이지네이션된 결과물을 병합시켜주는 작업이 필요하다.

<figure>
  <img src="https://user-images.githubusercontent.com/43979066/214867057-c64d3a63-2f3f-4391-ade5-54d2bc3a266b.png" alt="" width=800>
  <figcaption>각 페이지에 대한 쿼리 결과가 각각 다른 결과물로 분리되어 캐시 저장소에 저장되어있다.</figcaption>
</figure>

## 페이지네이션 결과 데이터 병합하기

위에서 설명한 것처럼, `fetchMore` 함수에서 실행된 쿼리는 기존 쿼리의 결과 데이터와 자동적으로 병합되지 않는다. 이를 병합하게 위해서는 페이지네이션 필드에 **field policy**를 적절히 설정해주어야한다.

### 왜 field policy가 필요한가?

다음과 같이 id 매개변수를 가지는 GraphQL 스키마가 있다고 가정해보자.

```graphql
type Query {
  user(id: ID!): User
}
```

이제 다음과 같은 쿼리를 두 번 호출하는데, 각각 다른 id 매개변수를 전달하여 실행한다면, 어떤 결과가 나올까?

```graphql
query GetUser($id: ID!) {
  user(id: $id) {
    id
    name
  }
}
```

두 개의 쿼리는 각각 완전히 다른 User 객체를 반환할 것이다. 적어도 한 개의 다른 매개변수(id)가 주어졌으므로 Apollo Client는 이 객체들을 스토어에 각각의 개체로서 분리하여 저장한다. 만약, 그렇게하지 않는다면 첫 번째 User 개체를 두 번째 User 개체가 덮어쓸 수도 있기 때문에 둘 다 캐시해야한다.

이제 서로 다른 page 매개변수값을 전달하는 두 개의 쿼리를 실행시켜보자.

```jsx
export const LIST_CHAPTER_COMMENT = gql`
  query ListChapterComment(
    $storyId: Int
    $chapterId: Int
    $parentId: Int
    $page: Int!
    $pageSize: Int
  ) {
    listChapterComment(
      data: {
        storyId: $storyId
        chapterId: $chapterId
        parentId: $parentId
        page: $page
        pageSize: $pageSize
      }
    ) {
      page
      totalCount
      list {
        id
        # ...
      }
    }
  }
`;
```

이 경우, 페이지네이션되어있는 `listChapterComment` 쿼리를 두 번 실행한다면 두 개의 서로 다른 데이터를 얻게 될 것이고, 우리는 이 데이터들을 병합해야 한다. 하지만 캐시는 이걸 모른다! 캐시는 이렇게 페이지네이션되어있는 쿼리를 실행하는 경우와 위 예제와 같이 다른 id 매개변수를 이용해 서로 다른 User 데이터를 얻는 상황을 구별하지 못한다.

우리는 field policy를 통해 특정 필드에 대한 캐시 동작을 커스텀할 수 있다. 즉, field policy를 적절히 설정하여 각각 다른 page 매개변수를 사용하여 실행한 쿼리들에 대해서는 개별적으로 데이터를 저장하지 않도록 지정할 수 있다.

### field policy 정의하기

field policy는 InMemoryCache의 특정 field를 읽고(read) 쓰는(write) 방법을 지정한다. 페이지로 묶인 쿼리의 결과들을 단일 리스트로 병합하는 필드 정책을 정의해보자.

InMemoryCache의 constructor에서 제공하는 typePolicies 옵션 내에서 field policy을 정의할 수 있다.

```jsx
const cache = new InMemoryCache({
  typePolicies: {
    Query: {
      fields: {
        listChapterComment: {
          // 결과를 개별적으로 캐싱할 기준이 되는 매개변수들 정의
          keyArgs: ["data", ["storyId", "chapterId", "parentId"]],
          // 기존 리스트와 새로운 리스트를 결합하는 방법 정의
          merge(
            existing = {
              list: [],
            },
            incoming
          ) {
            return {
              ...incoming,
              list: [...existing.list, ...incoming.list],
            };
          },
        },
      },
    },
  },
});
```

페이지네이션 작업을 위해서는 위 두 가지 구성요소를 정의해야한다.

- keyArgs: 필드의 매개변수 중 캐시가 해당 매개변수를 기준으로 별도로 데이터를 저장하도록하는 매개변수를 정의한다.

현재 variables에는 storyId, chapterId, parentId, page, pageSize가 있는데, 이 중 storyId, chapterId, parentId 중 하나라도 다르면 저장소의 다른 영역에 데이터를 별개로 저장해야 한다. 따라서 `keyArgs` 값에 storyId, chapterId, parentId 값을 정의해주었다.

또한 실제로 variables가 보내지는 형태는 `{ data: { storyId: 100, chapterId: 200, ,,, } }` 이므로

```jsx
keyArgs: ["data", ["storyId", "chapterId", , , ,]];
```

위와 같은 형태로 전달되어야 한다. ([참고](https://www.apollographql.com/docs/react/pagination/key-args))

<figure>
  <img src="https://user-images.githubusercontent.com/43979066/214866993-150d59c6-83bb-4c2d-84ba-7422c80ce9b4.png" alt="" width=800>
  <figcaption>keyArgs만 설정한 상태. 마지막으로 불러온 6 페이지의 결과값이 기존 데이터를 오버라이딩했다.</figcaption>
</figure>

- merge: 들어오는 데이터를 특정 필드의 기존 캐시된 데이터와 결합하는 방법을 Apollo Client 캐시에 알려준다. (이 함수를 정의해주지 않으면 새로 들어온 데이터가 기존 데이터를 override한다.)

<figure>
  <img src="https://user-images.githubusercontent.com/43979066/214866942-6f123f1b-f9bf-4201-80d7-aee51cd022c0.png" alt="" width=800>
  <figcaption>merge함수까지 설정한 상태. 6번째 페이지 데이터까지 불러와진 상태이고 이전 결과값이 list에 병합되었다.</figcaption>
</figure>

### ⚠️주의할 점(이자 내가 가장 헤맸던 점)

keyArgs를 정의할 때는 쿼리로 보내는 variables의 형태와 동일한 형태로 매개변수들을 정의해주어야한다.

listChapterComment 쿼리를 다시 한 번 보자.

```jsx
export const LIST_CHAPTER_COMMENT = gql`
  query ListChapterComment(
    $storyId: Int
    $chapterId: Int
    $parentId: Int
    $page: Int!
    $pageSize: Int
  ) {
    listChapterComment(
      data: {
        storyId: $storyId
        chapterId: $chapterId
        parentId: $parentId
        page: $page
        pageSize: $pageSize
      }
    ) {
      page
      totalCount
      list {
        id
        # ...
      }
    }
  }
`;
```

위 쿼리를 사용하는 코드에서는

```jsx
const { data, loading, fetchMore } = useQuery(LIST_CHAPTER_COMMENT, {
  variables: {
    storyId: 100,
    chapterId: 200,
    parentId: 300,
    page: 1,
    pageSize: 10,
  },
});
```

위와 같이 사용하기 때문에 variables의 형태가 아래와 같다고 생각하기 쉽다.

```jsx
{
  storyId: 100,
  chapterId: 200,
  parentId: 300,
  page: 1,
  pageSize: 10,
}
```

그래서 나는 한동안 keyArgs를 `['storyId', 'chapterId', 'parentId']` 로 설정해놓았고, 그 결과 listChapterComment 쿼리에 대한 모든 리스트가 병합되어 단일 개체로 캐시되는 문제가 있었다.

그러나 실제로 요청이 보내지는 형태는 아래와 같다.

```jsx
{
  data: {
    storyId: 100,
    chapterId: 200,
    parentId: 300,
    page: 1,
    pageSize: 10,
  }
}
```

즉, data라는 값에 객체가 중첩되어있는 형태로 매개변수가 보내지고있는데, 아폴로에서 이 중첩된 객체를 배열로 표현하는 방법이 조금 독특하다.

<figure>
  <img src="https://user-images.githubusercontent.com/43979066/214866863-24f10656-9c88-4528-aea1-25ca1b1fb987.png" alt="" width=800>
  <figcaption>merge함수까지 설정한 상태. 6번째 페이지 데이터까지 불러와진 상태이고 이전 결과값이 list에 병합되었다.</figcaption>
</figure>

즉 이 경우에는 keyArgs를 `['data', ['storyId', 'chapterId', 'parentId']]` 로 설정해주어야한다.

# _Resources_

- https://www.apollographql.com/docs/react/pagination/core-api
