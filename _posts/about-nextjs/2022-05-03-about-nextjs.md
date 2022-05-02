---
title: NextJS란? NextJS을 사용해야하는 이유
date: 2022-05-02 11:58:47 +09:00
modified: 2022-05-02 11:58:47 +09:00
tags: [nextjs]
description: 본격 NextJS 영업하는 글. Next 왜 안써??!!~~ framework와 library의 차이점과 SSR, file-based routing 등 nextjs의 특징 알아보기.
image: https://user-images.githubusercontent.com/43979066/166263265-83dec7cf-5cd8-4fc9-8224-e2c380f85c83.png
---

- [Framework vs library](#framework-vs-library)
    - [_예제) URL 라우팅_](#예제-url-라우팅)
      - [next.js](#nextjs)
      - [react](#react)
- [Next.js란](#nextjs란)
- [왜 사용하는가](#왜-사용하는가)
    - [Next.js의 장점](#nextjs의-장점)
- [_Resources_](#resources)

> Next.js는 SPA인 리액트의 강점을 살리면서 **SSR**를 쉽게 구현하도록 최적화되어있는 프레임워크이다.<br/>
> 또한 <mark>파일 기반의 자동 라우팅 기능 제공, 자동 코드 스플리팅 기능을 제공하는 등 개발자 경험</mark>을 높일 수 있다.<br/> <mark>SSR을 통해 SEO 성능을 높일 수 있고</mark>, 이는 직접적으로 사용자의 유입을 증대시킬 수 있어 마케팅적인 측면에서 유리하다. <br/><br/>마케팅을 위한 비용을 따로 들이지 않고도 기술 스택을 선택하는 것만으로도 사용자의 유입을 늘릴 수 있다는 점에서 매력적인 기술이라고 생각한다. 이러한 이유로 최근 많은 기업에서 next.js를 채택하여 사용하고 있다.

# Framework vs library

- **framework** : **뼈대, 기반 구조** - <mark>제어의 역전</mark>이 일어난다<br/>
  프레임워크가 개발자의 코드를 가져다 쓴다.<br/>
  개발자가 코드를 적절한 위치에 적으면 framework가 코드를 불러와 실행<br/>
  특정한 규칙에 따라 작성해야 함

제어의 흐름이 **프레임워크**에 있다
(통합성, 일관성, 수동적)

- **library** : **단순 활용가능한 도구들의 집합**<br/>
  개발자가 라이브러리의 코드를 가져다 쓴다.<br/>
  개발자가 원하는대로 코드를 작성,<br/>
  개발자가 사용하고싶을 때 사용<br/>
  언제 코드를 부를지, 어떤 구조로 프로젝트를 작성할 지 개발자가 결정하여 코드를 작성

제어의 흐름이 **개발자**에게 있다.
(능동적, 자율적)

### _예제) URL 라우팅_

#### next.js

pages폴더 하위에 url명으로 파일명을 지정<br/>
→ 파일 내부에 컴포넌트를 정의<br/>
→ next.js가 알아서 해당 컴포넌트를 해당 url주소 입력시 실행

> **\* 규칙**
>
> - 파일명은 url명이 된다.(홈페이지단에서는 기본적으로 index.js파일이 실행된다.)
> - 파일 내에서 export default로 컴포넌트를 정의해야 한다.
> - 컴포넌트의 이름은 상관없다

#### react

react-router-dom 패키지를 다운<br/>
→ router 설계<br/>
→ router 정의<br/>
→ 컴포넌트를 작성하여 export<br/>
→ 컴포넌트를 불러와 해당 router에 지정<br/>
이러한 작업들을 개발자가 직접 코드로 작성

**- 다른 개발자가 코드를 봤을 때 라우팅을 어떤 방법으로 구현했는지 파악하는 데 시간이 소요된다.**<br/>
**- url 라우팅을 서드파티 라이브러리에 의존하는 것은 안정성 측면에서 좋지 않은 영향을 준다.**

# Next.js란

- 리액트 프레임워크이다 : 리액트 애플리케이션을 만들 때 고려해야하는 **\*여러가지 세부적인 사항**들을 손쉽게 해결할 수 있는 인터페이스를 제공한다.<br/><br/>
  **\* 세부사항**
  - 코드는 webpack과 같은 번들러를 이용해 번들링되어야하고, babel과 같은 컴파일러를 이용해 변환되어야한다.
  - 코드 스플리팅과 같은 방법을 이용해 성능 최적화를 수행한다.
  - SEO 성능 개선을 위해 몇몇의 페이지는 정적인 pre-rendering이 필요하다. 또한 SSR과 CSR 모드를 사용하고자할 수 있다.
  - React 애플리케이션과 데이터 저장소를 연결하기 위해 서버사이드 코드를 작성해야할 수 있다.

# 왜 사용하는가

### Next.js의 장점

- **서버사이드 렌더링(Server-side Rendering, SSR)**
  - CSR의 경우 브라우저는 빈 껍데기(`<div class=”root”></div>`)상태의 html파일을 받아 js를 작동시켜 유저에게 보여지는 UI를 빌드한다. 이 경우, 지연시간이 발생해 인터넷 속도가 느린 유저에게 로딩 메세지조차 없는 빈 화면이 보여질 수 있는 등 UX적으로 좋지 못한 결과를 초래한다.
    SSR의 경우 data fetching작업이 서버측에서 이루어지기 때문에 지연시간이 발생하지 않아 첫 페이지 로딩 시 더 나은 사용자 경험을 제공한다.
  - 대부분의 검색엔진봇들은 서버로부터 전달받은 html파일만을 이용해 페이지를 크롤링하기 때문에 js 스크립트 파일이 실행 된 뒤 페이지의 내용이 채워지는 CSR에 비해 SEO측면에서 유리하다.
  - CSR + SSR 혼합된 어플리케이션 빌드 가능
  - UI를 구성하는 데 js가 사용되지 않기 때문에 브라우저의 js가 비활성화되어있더라도 사용자는 서버로부터 응답받은 정적인 페이지를 볼 수 있게 된다.
- **파일 기반 라우팅(File-based Routing)**
  - 위 예시에서 보듯이 React를 이용하는 경우 라우팅을 하려면 추가적인 패키지 설치와 코드 작성이 필요하다. 그러나 Next.js의 경우에는 코드가 아닌 폴더와 파일의 구조를 통해 페이지를 정의하고 routing할 수 있게 된다.
- **리액트 애플리케이션 풀스택(FE + BE) 빌드**
  - 자체적으로 서버 API를 삽입할 수 있다
- **자동 코드 스플리팅**
  - next.js는 리소스가 임포트 된것들을 분석하여, 로딩한 페이지가 꼭 필요로하는 JS 파일만 로드
  - 그래서 첫페이지 로드가 빠르고 나중에 꼭 사용되는 JS 파일들만 클라이언트에 보낸다.
  - 다만 사이트에서 절반 이상 자주 사용되는 JS 파일은 main JS bundle로 이동된다.
- **typescript에 친화적**
- **static pre-rendering**

<br/>

> 나의 결론은 "안 쓸 이유가 없다" 이다. Vercel을 이용하면 배포도 쉽다. <br/>
> react와 문법이 거의 동일하기 때문에 react에 익숙한 개발자라면 러닝커브도 높지 않다.<br/>
> 다만 SSR이나 next/image 등 NextJS에서 제공하는 기능을 잘 사용하기 위해서는 쪼끔 공부 해야한다.😉 <br/>
> 아무튼 나는 NextJS를 강추한다!👏👏

<br/>

# _Resources_

[NextJS 공식문서](https://nextjs.org/learn/basics/create-nextjs-app?utm_source=next-site&utm_medium=homepage-cta&utm_campaign=next-website)<br/>
[Next.js 그거 어떻게 하는 건데.](https://well-balanced.medium.com/next-js-%EA%B7%B8%EA%B1%B0-%EC%96%B4%EB%96%BB%EA%B2%8C-%ED%95%98%EB%8A%94-%EA%B1%B4%EB%8D%B0-ea5637f25fa4)
