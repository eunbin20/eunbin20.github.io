---
title: GraphQL로 서버 구축하기 (with Apollo Server)
date: 2022-03-13 11:58:47 +09:00
modified: 2022-03-13 11:58:47 +09:00
tags: [graphql, apollo_server]
description: 인프런 강의 ⌜얄팍한 GraphQL과 Apollo⌟을 수강하며 정리한 내용이다. GraphQL을 이용해 서버를 구축할 때 필요한 몇 가지 개념과 Apollo Server를 이용해 간단하게 서버를 구축해본다.
image: https://user-images.githubusercontent.com/43979066/158075767-1f23a59a-7726-4140-bd04-8a0301065452.png
---

> 인프런 강의 [⌜얄팍한 GraphQL과 Apollo⌟](https://www.inflearn.com/course/%EC%96%84%ED%8C%8D%ED%95%9C-graphql-apollo/dashboard)을 수강하며 정리한 내용이다.

# Query

Query는 데이터를 읽을 때(READ) 사용된다.<br/>
루트타입, Type, Resolver로 구성된다.

- 1. Query 루트 타입

```gql
type Query {
  teams: [Team]
}
```

- 자료 요청에 사용된 쿼리들 정의
- 쿼리 명령문마다 반환될 데이터의 형태가 지정되어있다.
  반환되는 데이터의 형태는 Type에 정의되어있다.⤵️

- 2. Type

```gql
type Team {
  id: Int
  manager: String
  office: String
  extension_number: String
  mascot: String
  cleaning_duty: String
  project: String
}
```

반환될 데이터의 형태를 지정한다. 자료형을 가진 필드들로 구성된다.

- 3. Resolver

```gql
const resolvers = {
  Query: {
    teams: () => database.teams
  }
}
```

- 쿼리 각각의 항목에 대해 적절한 데이터를 반환하는 액션 함수를 선언한다.
- 실제 데이터베이스와 쿼리를 연결

# Mutation

Mutation은 데이터를 추가하거나(CREATE), 삭제하거나(DELETE), 업데이트(UPDATE)할 때 사용된다.

- 1. Mutation 루트 타입

```gql
type Mutation {
  deleteTeam(id: String): Team
}
```

(String형태로 id를 받아 해당 아이디를 가진 Team 데이터를 삭제하고 삭제된 Team을 반환하는 Mutation)<br/>

- 자료 추가, 삭제 업데이트에 사용되는 뮤테이션들을 정의
- 쿼리 명령문마다 반환될 데이터의 형태가 지정되어있다.
  반환되는 데이터의 형태는 Type에 정의되어있다.⤵️

- 2. Type

```gql
type Team {
  id: Int
  manager: String
  office: String
  extension_number: String
  mascot: String
  cleaning_duty: String
  project: String
}
```

반환될 데이터의 형태를 지정한다. 자료형을 가진 필드들로 구성된다.

- 3. Resolver

```gql
Mutation: {
  deleteEquipment: (parent, args, context, info) => {
    const deleted = database.equipments
      .filter((equipment) => {
        return equipment.id === args.id
      })[0]
    database.equipments = database.equipments
      .filter((equipment) => {
        return equipment.id !== args.id
      })
    return deleted
  }
}
```

- 각각의 뮤테이션에 대해 데이터를 적절하게 핸들링해주는 액션 함수를 정의한다.

# GraphQL의 자료형

## 스칼라 타입

GraphQL의 내장 자료형

```gql
type ExampleType {
  id: ID!
  used_by: String!
  count: Int!
  use_rate: Float
  is_new: Boolean!
}
```

- ID: 기본적으로는 String, 고유 식별자의 역할임을 나타낸다.
- String: utf-8 문자열
- Int: 부호가 있는 32비트 정수
- Float: 부호가 있는 부동소수점 값
- Boolean: 참/거짓

> !: Non null
> null을 반환할 수 없음 (= 반드시 값이 있음)

## 열거 타입(Enum)

이미 지정된 값들 중에서 반환

## 리스트 타입

특정 타입의 배열을 반환

```gpl
type Equipment {
  ...
  users: [String!]
}
```

\* users = [String] | [String!] | [String]! | [String!]!

> (users 내 요소를 user라고 칭함)

- users = [String] <br/>
  users와 user 모두 null이 될 수 없다.
- users = [String!]<br/>
  users는 null이 될 수 있지만 user는 null이 될 수 없다.<br/>
  (users = [..., null] ❌)<br/>
- users = [String]!
  users는 null이 될 수없지만 user는 null이 될 수 있다.<br/>
- users = [String!]!
  users와 null 모두 null이 될 수 있다.<br/>

  +) 위 경우 모두 [](빈 배열) 가능!

- 객체 타입

사용자에 의해 정의된 타입들

- Union

여러 개의 타입을 한 배열에 반환하고자할 때 사용

주어진 데이터에 Equipment항목이 있으면 count, used_by 반환,

주어진 데이터에 Supply항목이 있으면 id 반환

```gql
query {
  givens {
    __typename
    ... on Equipment {
      count
      used_by
    }
    ... on Supply {
      id
    }
  }
}
```

- Interface

유사한 객체 타입을 만들기 위한 공통 필드 타입

- 인자와 인풋 타입

- 별칭으로 받아오기

```gql
query {
  badGuys: peopleFiltered(sex: male, blood_type: B) {
    first_name
    last_name
    sex
    blood_type
  }
  newYorkers: peopleFiltered(from: "New York") {
    first_name
    last_name
    from
  }
}
```

하나의 쿼리문에서 여러 종류의 데이터를 받아올 때, 적절한 별칭을 이용해 데이터를 받아오면 효율적으로 데이터를 받아올 수 있다.
<br/>

[서버 구축 코드](https://github.com/eunbin20/yalco-inflearn-graphql-server)

# _Resources_

- [⌜얄팍한 GraphQL과 Apollo⌟](https://www.inflearn.com/course/%EC%96%84%ED%8C%8D%ED%95%9C-graphql-apollo/dashboard)
