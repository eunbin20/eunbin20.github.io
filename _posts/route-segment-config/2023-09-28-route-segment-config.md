---
title: Nextjs13 페이지 dynamic 옵션
date: 2023-09-26 00:00
modified: 2023-09-26 00:00
tags: [nextjs13]
description: "Nextjs 13 버전에서는 페이지, 레이아웃, 또는 Route 동작을 세밀하게 제어할 수 있는 여러 옵션을 제공한다. 그 중 dynamic 설정에 대해 알아보자."
image: https://github.com/eunbin20/til/assets/43979066/08e4df5a-2a47-4a93-a252-ea63168341df
---

Nextjs 13 버전에서는 페이지, 레이아웃, 또는 Route 동작을 세밀하게 제어할 수 있는 여러 옵션을 제공한다. 그 중 dynamic 설정에 대해 알아보자.

<figure>
  <img src="https://github.com/eunbin20/til/assets/43979066/08e4df5a-2a47-4a93-a252-ea63168341df" alt="thinking emoji" width=500>
</figure>

dynamic 옵션은 `auto`, `force-dynamic`, `error`, `force-static`로 총 4가지의 옵션이 있다.

### auto

```tsx
export const dynamic = "auto";

export default function MyComponent() {}
```

옵션의 기본값으로 가능한 많은 데이터를 캐싱하지만 동적으로 불러오는 컴포넌트를 제한하지는 않는다.

### force-dynamic

```tsx
export const dynamic = "force-dynamic";

export default function MyComponent() {}
```

캐싱을 사용하지 않고 코드를 완전히 동적으로 렌더링한다. 해당 코드 내에 있는 모든 데이터 페칭 요청의 캐싱을 비활성화하고 항상 새로운 값을 불러온다. <br/>
이 옵션은 서버사이드 렌더링을 수행하거나 모든 데이터 요청의 캐시를 무시하고 다시 데이터를 가져오게한다.

- 예전 page 폴더구조에서 getServerSideProps()함수를 사용한것과 비슷하게 동작한다.
- 모든 데이터 페칭 함수 요청의 옵션이 { cache: 'no-store', next: { revalidate: 0 } } 로 세팅되어 동작한다.

### error

```tsx
export const dynamic = "error";

export default function MyComponent() {}
```

페이지를 완전히 정적으로 렌더링하되, 컴포넌트가 동적 함수나 캐시되지 않은 데이터를 사용하는 경우 오류를 발생시킨다. 동적인 동작을 강제로 막는 옵션이다.

- 예전 page 폴더구조에서 getStaticProps()함수를 사용한것과 비슷하게 동작한다.
- 모든 데이터 페칭 함수 요청의 옵션이 { cache: 'force-cache', next: { revalidate: 0 } } 로 세팅되어 동작한다.

### force-static

```tsx
export const dynamic = "force-static";

export default function MyComponent() {}
```

페이지를 완전히 정적으로 렌더링하고cookies(), headers() , useSearchParams() 값을 비워버린다.
