---
title: 사이드 프로젝트 개발기 | Yumiko Simulator를 소개합니다
date: 2024-10-27 00:00
modified: 2024-10-27 00:00
tags: [side_project]
description: 원하는 대로 발레복 색상과 디자인을 바꿔볼 수 있는 디자인 시뮬레이터 Yumiko Simulator 사드 프로젝트 소개글
image: https://github.com/user-attachments/assets/cc12d9d9-652c-4124-a527-84ff88cb62d1
---

저는 취미로 발레를 즐기고 있는 5년 차 취미 발레러입니다 🙌

우연히 발레 수업을 접한 뒤 그 매력에 푹 빠져, 지금까지 꾸준히 발레 학원에 다니고 있어요.

<div align=center>
  <img
    align=center
    src="https://velog.velcdn.com/images/dmsqlswkd/post/fdec2802-2024-4713-867c-988d2312735c/image.webp"
    width="350">
</div>

발레의 매력은 여러 가지가 있지만, 특히 다양한 색상과 소재, 디자인의 발레복을 고르고 입는 즐거움이 크죠 🤓

그중에서도 일본 발레복 브랜드 Yumiko는 저 같은 취미 발레러부터 전문 발레리나들까지 많은 사랑을 받고 있는 브랜드인데요. Yumiko에서는 고객이 원하는 색상과 소재를 선택해 자신만의 발레복을 디자인할 수 있는 커스텀 오더 시스템을 제공하고 있어요.

저도 이 서비스를 이용해 다양한 색상과 소재를 조합해봤지만, 사용하면서 한 가지 아쉬운 점을 느꼈습니다…

## 개발 동기

<div align=center>
  <img
    align=center
    src="https://github.com/user-attachments/assets/334073be-07c7-4d4f-ae08-2d14fbaf3ef5"
    width="300">
</div>

<br/>

위 화면은 [Yumiko](https://www.yumiko.com/products/anna) 홈페이지에서 실제로 커스텀 오더 방식으로 주문할 때 보여지는 화면입니다.

옷의 디자인과 색상을 셀렉 박스로만 선택이 가능할 뿐, **이 색상들의 조합이 실제로 어떤 모습일지 즉시 확인할 수 없었습니다.🥲**

**그래서 저는 이러한 경험에서 영감을 받아, 사용자가 선택한 커스텀 디자인이 실제로 반영된 모습을 실시간으로 확인할 수 있는 "Yumiko Simulator"라는 서비스를 기획하게 되었습니다.**

<br/>

<div align=center>
  <img
    align=center
    src="https://github.com/user-attachments/assets/cc12d9d9-652c-4124-a527-84ff88cb62d1"
    width="500">
</div>

<h1 align=center> Yumiko Simulator 🩰 </h1>

<h4 align=center>원하는 대로 발레복 색상과 디자인을 바꿔볼 수 있는 디자인 시뮬레이터</h4>
<br/>
<p align=center><a href="https://balletbin.com">👉사이트 바로가기👈</a></p>

<hr/>

## 기능 설명

<div align=center>
  <img
    align=center
    src="https://github.com/user-attachments/assets/e983cdf3-f535-4e05-b9eb-1783380372d6"
    width="500">
</div>

- 발레복의 디자인 타입과 몸판 색상, 트림 색상을 선택하면 해당 디자인이 반영된 모습을 왼쪽 미리보기 화면에서 실시간으로 확인할 수 있습니다.
- 다양한 선택 옵션 제공: Yumiko에서 제공하는 다양한 색상, 소재, 트림 등을 자유롭게 조합하여 자신만의 발레복을 디자인할 수 있습니다.
- 반응형 디자인을 통해 PC와 모바일 화면에서 유연하게 대응되어 편안한 사용자 경험을 줄 수 있도록 하였습니다.

## 기술 스택

**Common**

- Prettier & ESLint: 코드 스타일 통일성과 정적 검사 기능을 통해 협업 시 코드 품질을 높이고, 유지보수를 쉽게 하기 위함입니다. 개발 과정에서 실수를 줄이고, 일관된 코드를 유지하는 데 도움을 주기 때문에 이 두 가지 도구를 필수적으로 도입했습니다.

**Frontend**

- TypeScript: 타입 안전성을 위해 선택했습니다. 런타임 환경에서 발생하는 타입 오류를 발견할 수 있어 예상치 못한 에러를 줄이고 디버깅을 용이하게 하는 데 큰 도움이 됩니다.
- Next.js: SEO 최적화와 SSR(Server-Side Rendering)을 통해 검색엔진에 잘 노출되도록 하며, 초기 렌더링 속도를 높일 수 있습니다. 또한, next/image의 이미지 최적화 기능과 페이지 단위의 코드 스플리팅 등 다양한 최적화 옵션을 제공하여, 이러한 기능들을 활용하기 위해 Next.js를 채택했습니다.
- Tailwind CSS: 클래스 네이밍의 부담을 덜고 CSS와 HTML 간 컨텍스트 전환을 줄여 개발 생산성 향상에 큰 도움을 줍니다. 또한 반응형 디자인을 손쉽게 처리할 수 있고, 컴포넌트 재사용 측면에서도 유리한 측면이 있어 선택하게 되었습니다.

**Backend**

- Supabase: 간단한 데이터베이스 관리를 위해 Supabase를 선택했습니다. Supabase는 Firebase와 비슷한 기능을 제공하면서도 SQL 기반의 접근이 가능해 데이터의 구조를 관리하는 데 효율적입니다. 현재 발레복의 이름과 이미지, 색상 이름과 이미지 등을 저장하는 데 사용하고있습니다.

**Deployment**

- Vercel: 빠르고 손쉬운 배포를 위해 Vercel을 선택했습니다. Next.js와의 통합이 최적화되어 있어 빌드 및 배포가 매우 간편합니다.

## Future Plan

**feature**

- [ ] 유저 로그인 기능 추가하기
- [ ] 나만의 코디를 저장하는 기능 추가하기
  - 로그인 후에 이용할 수 있느 기능으로, 본인이 조합한 디자인을 저장하여 리스트형태로 확인할 수 있습니다.
- [ ] SEO 개선하여 구글 검색 순위 향상 시키기
- [ ] 이미지 로딩속도, 로딩 화면 개선하기

**technical**

- [ ] 컴포넌트 로직 리팩토링하기. <- 아토믹 디자인 적용하기
- [ ] 적절한 상태관리 라이브러리 추가하기

## 마무리

Yumiko Simulator는 제가 발레복 쇼핑을 하면서 느꼈던 불편함을 해소하기 위해 시작한 사이드 프로젝트입니다. 아직은 완벽하지 않고 보완할 부분도 많지만, 차근차근 개선해나가보려고합니다!

앞으로 기능을 점차적으로 개선해나가며, 프로젝트 발전 과정과 기술적 고민들을 블로그에 차근차근 기록할 예정이에요. 이 과정들을 통해 제가 조금 더 성장할 수 있기를 기대하며, 저의 첫 사이드 프로젝트 소개글을 마칩니다. 😊
