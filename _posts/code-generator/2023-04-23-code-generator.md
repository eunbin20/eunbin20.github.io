---
title: TypeScript GraphQL Code Generator 도입기
date: 2023-04-22 00:00
modified: 2023-04-22 00:00
tags: [apollo-client]
description: 명령어 한 줄로 서버 타입을 정의해주는 Code Generator 도입기
image: https://user-images.githubusercontent.com/116791055/233842013-486fe126-7185-4169-a6fe-a94cf943dc64.png
---

> 기존 프로젝트에서는 Apollo Client를 이용해 GraphQL로 구축되어있는 서버와 통신하고있습니다. 또한 Typescript를 이용한 정적 타이핑을 사용하여 개발하고있어 기존에는 서버에서 생성된 GraphQL Schema를 참고하여 타입을 직접 선언하였습니다.
> <br/>
> 그러나 이 방법은 지루하기도 하고, 휴먼 에러가 발생할 가능성이 있습니다.(특히 nullable을 체크하는 부분에서요😱)
> <br/> <br/>
> 이 문제를 해결하기 위해 **GraphQL Code Generator**를 도입하게 되었습니다!

**Code Generator는 Apollo Client, URQL, 또는 ReactQuery를 사용하는 React, Vue, Angular, NestJS 등의 프론트엔드 어플리케이션에서 Query, Mutation, Subscription의 타입을 자동으로 정의해줍니다.**

쿼리 또는 뮤테이션 요청을 작성하면 서버의 API 스키마를 참고하여 자동적으로 타입을 정의해주어 수동으로 타입을 입력하는 과정에서 발생할 수 있는 오타, 타입정의 실수 등의 휴먼 에러가 방지됩니다.

특히 이미 구현되어있는 기능에서 일부 UI가 변경되어 쿼리 또는 뮤테이션 요청을 수정해야하는 경우, 명령어 한 줄로 수정된 타입을 즉각적으로 반영하여 개발할 수 있어 개발 생산성 향상에 큰 도움을 줍니다.

# Code Generator 세팅 방법

- **필요한 패키지를 설치합니다.**

```bash
yarn add @graphql-codegen/cli @graphql-codegen/typescript-operations @graphql-codegen/typescript-react-apollo ts-node
```

- **@graphql-codegen/cli:** graphql-codegen을 사용할 수 있게 하는 패키지입니다.
- **@graphql-codegen/typescript-operations:** Graphql Schema와 Graphql 쿼리, 뮤테이션 요청을 바탕으로 Typescript 타입을 만들어줍니다.
- **@graphql-codegen/near-operation-file-preset:** 각각의 오퍼레이션(gql 태그로 만들어진 쿼리 또는 뮤테이션)파일에 대한 타입 정의 파일을 생성해줍니다.
- **@graphql-codegen/typescript-react-apollo:** 타입이 정의된 React Apollo 컴포넌트(ex. useQuery, useMutation,,,)를 생성해줍니다.
- **ts-node:** ts 익스텐션의 configuration파일을 사용하기 위한 패키지입니다. TypeScript Compiler를 통하지 않고도, 직접 TypeScript를 실행시키는 역할을 합니다.

# Development workflow

package.json에서 codegen을 실행할 수 있는 명령어를 세팅합니다.

```js
{
  "scripts": {
    "generate": "graphql-codegen",
    "generate:watch": "graphql-codegen --watch"
  },
  ...
}
```

<br/>

- `yarn generate`

새로 추가되거나 업데이트된 쿼리 또는 뮤테이션 요청에 대한 타입을 자동적으로 생성해줍니다.

- `yarn generate:watch`

작업 과정에서 추가되거나 수정되는 쿼리 필드를 실시간으로 반영하여 타입을 자동적으로 생성해줍니다.

위 명령어를 실행하면 구성 설정 파일에 따라서 타입이 생성됩니다. 구성 설정 파일은 다음과 같습니다. (codegen.ts)

```tsx
import { CodegenConfig } from "@graphql-codegen/cli";

const config: CodegenConfig = {
  schema: "https://api.dev.storyplay.com/graphql",
  documents: ["src/operations/**/*.{ts,tsx}", "!src/**/*.generated.ts"],
  generates: {
    "src/baseType.ts": {
      plugins: ["typescript"],
    },
    "src/": {
      preset: "near-operation-file",
      presetConfig: {
        baseTypesPath: "baseType.ts",
        folder: "__generated__",
      },
      plugins: ["typescript-operations", "typescript-react-apollo"],
    },
  },
};

export default config;
```

<br/>

- **schema**: 스키마 타입을 어떤 서버로부터 가져올지를 지정하는 속성입니다. 저희는 공통 개발서버의 스키마를 참조하여 타입을 정의하도록 설정되어있습니다.
- **documents**: 저희가 작성한 graphQL operation들이 어디에 위치해있는지를 지정하는 속성입니다.
  `src/operations/**/*.{ts,tsx}` 안에 있는 쿼리 또는 뮤테이션문을 참조하고, `src/**/*.generated.ts` 파일은 참조하지 마라! 라는 의미입니다.
- **generates**: 타입으로 변환할 때 사용할 preset, plugin등을 명시합니다.
  - `src/baseType.ts` 파일에는 전체 base 스키마를 저장합니다.,
  - `src/` 하위 어딘가에 존재하는 operation파일을 찾아 해당 파일과 같은 depth에 타입 정의 파일을 만듭니다. 이때 src 하위에 있는 `baseType.ts` 스키마 파일을 참조하여 해당 operation에 대한 타입을 정의합니다.
    (이 부분에 대한 자세한 설명은 아래 폴더 구조를 참고해주세요.)

## 폴더 구조

새롭게 추가된 src/operations폴더의 구조는 다음과 같습니다.

```
src
│   ├── operations
│   │   ├── mutations
│   │   │   └── comment
│   │   │       ├── ReportchapterComment.ts
│   │   │       └── updateChapterComment.ts
│   │   ├── queries
│   │   │   └── comment
│   │   │       ├── getChapterComment.ts
│   │   │       └── getListChapterCommentReportingReason.ts

```

codegen을 실행하면 gql태그가 포함되어있는 파일을 찾아 해당 파일과 같은 depth에 **generate**폴더를 만들고 해당 파일 이름.generate.ts라는 이름으로 타입을 정의한 파일을 생성해줍니다.

```
├── src
│   ├── operations
│   │   ├── mutations
│   │   │   └── comment
│   │   │       ├── __generated__
│   │   │       │   ├── ReportchapterComment.generated.ts
│   │   │       │   └── updateChapterComment.generated.ts
│   │   │       ├── cancelLikeChapterComment.ts
│   │   │       └── updateChapterComment.ts
│   │   ├── queries
│   │   │   └── comment
│   │   │       ├── __generated__
│   │   │       │   ├── getChapterComment.generated.ts
│   │   │       │   └── getListChapterCommentReportingReason.generated.ts
│   │   │       ├── getChapterComment.ts
│   │   │       └── getListChapterCommentReportingReason.ts

```

예시) deleteChapterComment.generated.ts

```tsx
import * as Types from "../../../../baseType";

import { gql } from "@apollo/client";
import * as Apollo from "@apollo/client";
const defaultOptions = {} as const;
export type DeleteChapterCommentMutationVariables = Types.Exact<{
  id: Types.Scalars["Int"];
}>;

export type DeleteChapterCommentMutation = {
  __typename?: "Mutation";
  deleteChapterComment: { __typename?: "OkResponse"; ok: number };
};

export const DeleteChapterCommentDocument = gql`
  mutation DeleteChapterComment($id: Int!) {
    deleteChapterComment(data: { id: $id }) {
      ok
    }
  }
`;
export type DeleteChapterCommentMutationFn = Apollo.MutationFunction<
  DeleteChapterCommentMutation,
  DeleteChapterCommentMutationVariables
>;

/**
 * __useDeleteChapterCommentMutation__
 *
 * To run a mutation, you first call `useDeleteChapterCommentMutation` within a React component and pass it any options that fit your needs.
 * When your component renders, `useDeleteChapterCommentMutation` returns a tuple that includes:
 * - A mutate function that you can call at any time to execute the mutation
 * - An object with fields that represent the current status of the mutation's execution
 *
 * @param baseOptions options that will be passed into the mutation, supported options are listed on: https://www.apollographql.com/docs/react/api/react-hooks/#options-2;
 *
 * @example
 * const [deleteChapterCommentMutation, { data, loading, error }] = useDeleteChapterCommentMutation({
 *   variables: {
 *      id: // value for 'id'
 *   },
 * });
 */
export function useDeleteChapterCommentMutation(
  baseOptions?: Apollo.MutationHookOptions<
    DeleteChapterCommentMutation,
    DeleteChapterCommentMutationVariables
  >
) {
  const options = { ...defaultOptions, ...baseOptions };
  return Apollo.useMutation<
    DeleteChapterCommentMutation,
    DeleteChapterCommentMutationVariables
  >(DeleteChapterCommentDocument, options);
}
export type DeleteChapterCommentMutationHookResult = ReturnType<
  typeof useDeleteChapterCommentMutation
>;
export type DeleteChapterCommentMutationResult =
  Apollo.MutationResult<DeleteChapterCommentMutation>;
export type DeleteChapterCommentMutationOptions = Apollo.BaseMutationOptions<
  DeleteChapterCommentMutation,
  DeleteChapterCommentMutationVariables
>;
```

상위에 만들어진 `DeleteChapterCommentMutationVariables`와 `DeleteChapterCommentMutation` 는 각각 해당 요청을 보낼 때 필요한 variable의 타입과 요청에 대한 결과 데이터에 대한 타입이 정의되어있습니다.

그리고 그 아래에는 실제로 요청을 보낼 수 있는 hook이 있고 그 위에는 주석으로 사용방법이 간단하게 소개되고있습니다.

주석을 참고하셔서 이 hook을 바로 가져다 사용하셔도 되지만, 이 hook을 커스텀하여 컴포넌트 내에서 정의될 필요가 없는 비즈니스 로직은 `getListChapterCommentReportingReason.ts` 파일에서 작성해주시는 것을 권장합니다.

예를 들면 이렇게요!

```tsx
import { gql } from "@apollo/client";
import { CHAPTER_COMMENT } from "~/operations/queries/comment/useListChapterComment";
import { CommentItemFragment } from "~/operations/queries/comment/__generated__/useListChapterComment.generated";
import { useAppDispatch, useAppSelector } from "@store/hooks";
import { setParentData } from "~/slices/reCommentSlice";
import { setToastMessage } from "~/slices/commentSlice";
import { useDeleteChapterCommentMutation as useMutation } from "./__generated__/deleteChapterComment.generated";
import useTranslation from "next-translate/useTranslation";

export const DELETE_CHAPTER_COMMENT = gql`
  mutation DeleteChapterComment($id: Int!) {
    deleteChapterComment(data: { id: $id }) {
      ok
    }
  }
`;

export const useDeleteChapterCommentMutation = () => {
  const parentData = useAppSelector((state) => state.reComment.parentData);
  const dispatch = useAppDispatch();
  const { t } = useTranslation();

  const [mutate] = useMutation({
    update(cache) {
      // apollo cache에서 특정 댓글 삭제를 처리하는 로직
    },
    onError: () => {
      // 또는 에러핸들링 로직
    },
    // 등등 컴포넌트 내의 로직과 분리 가능한 비즈니스로직들...
  });

  return { mutate };
};
```

위와 같이 cache를 수정하거나 조작하는 등의 로직을 hook으로 분리해주면 컴포넌트의 UI와 상태 관리 로직이 분리되어 관심사가 분리되는 효과가 있습니다.

위와 같은 방법을 권장하지만, 상황에 맞게 필요한 type, 또는 hook을 가져다 사용하시면 됩니다 🙂

# Reference

[https://the-guild.dev/graphql/codegen/docs/getting-started](https://the-guild.dev/graphql/codegen/docs/getting-started)
