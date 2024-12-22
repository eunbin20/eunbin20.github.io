---
title: React 프로젝트에서 SVG를 효율적으로 활용하기
date: 2024-12-22 11:58:47 +09:00
modified: 2024-12-22 11:58:47 +09:00
tags: [svgr]
description: SVGR 도입과 설정 가이드
image: https://github.com/user-attachments/assets/d31c676f-a309-47e5-b2c0-67c221749989
---
SVG(Scalable Vector Graphics)는 XML 기반의 벡터 이미지 포맷으로, 크기와 해상도에 구애받지 않고 품질을 유지하며, 파일 크기가 작아 웹 어플리케이션에서 많이 사용되고있습니다. 특히, 아이콘, 로고 등 UI 요소에 적합하며, React와 같은 현대 프론트엔드 프레임워크에서도 SVG 활용도가 높습니다.

SVG 이미지는 <img> 태그를 통해서도 쉽게 렌더링할 수 있지만, SVG를 React 컴포넌트로 변환하면 Props를 통해 크기, 색상 등의 속성을 동적으로 조정할 수 있고, 이미지 로딩 속도도 개선됩니다. 

이번 포스팅에서는 SVG 이미지를 컴포넌트로 변환하는 작업을 보다 효율적으로 수행하기 위해 SVGR을 도입하고, 이를 프로젝트에 적용한 경험을 공유합니다!


### 샘플 프로젝트
svgr 라이브러리를 이용해 svg 아이콘 컴포넌트 생성 자동화한 샘플 프로젝트를 생성해두었습니다! 아래 링크를 통해 접근하실 수 있습니다 🙂

- https://github.com/eunbin20/svgr-generater

# SVGR 소개
SVGR은 SVG 파일을 JSX로 변환해 React 컴포넌트로 활용할 수 있게 해주는 도구입니다. 
이를 통해 SVG 이미지를 Props로 유연하게 제어하며, 코드베이스에서 SVG 파일 관리가 간소화됩니다. SVGR은 다양한 방식으로 사용할 수 있습니다.

- Playground: SVGR Playground에서 간단히 변환할 수 있습니다. 적은 개수의 svg 이미지 변환이 필요한 경우 가볍게 사용할 수 있는 방법입니다.
- CLI(Command Line Interface): 명령어를 통해 다수의 SVG 파일을 일괄적으로 변환할 수 있습니다.
- Webpack Loader: Webpack을 통해 프로젝트 빌드 과정에서 SVG 변환

제가 선택한 방법은 CLI를 사용해 명령어 기반으로 SVG를 컴포넌트화하는 방식으로, 이 방법은 대량의 SVG 파일을 관리하는 데 유용합니다.

# 프로젝트에 SVGR 설정하기
1. SVGR CLI 설치
프로젝트에서 SVGR CLI를 사용하려면 다음 명령어를 실행해 @svgr/cli를 설치합니다:

```bash
npm install --save-dev @svgr/cli
# 또는 Yarn 사용 시
yarn add --dev @svgr/cli
```

2. 스크립트 추가
SVG 파일을 변환하는 명령어를 package.json의 스크립트에 추가합니다:

```
// package.json
{
  "scripts": {
    "svg-generate": "npx @svgr/cli -- public/assets/svgs"
  }
}
```

위 스크립트는 public/assets/svgs 폴더에 있는 SVG 파일들을 JSX 기반의 React 컴포넌트로 변환합니다.

3. SVGR 설정 파일 생성
프로젝트 루트에 .svgrrc.js 파일을 생성하고 아래와 같이 옵션을 설정합니다:

```jsx
// .svgrrc.js
module.exports = {
  typescript: true,
  dimensions: false,
  outDir: './src/components/icons/generated',
  template: require('./src/utils/svgr.template'),
  svgProps: {
    className: '{className}',
  },
  expandProps: false,
  replaceAttrValues: {
    '#000000': '{color}',
    '#000': '{color}',
  },
};
```
**주요 옵션**

- typescript: TypeScript 기반 컴포넌트 생성 여부.
- dimensions: 기본 width와 height 속성 제거.
- outDir: 변환된 컴포넌트를 저장할 디렉토리.
- template: 커스텀 템플릿 사용 (예: 추가 Props 정의).
- svgProps: 공통적으로 적용할 Props 설정.
- replaceAttrValues: 특정 속성 값을 Props로 대체.

이외에도 다양한 커스텀 옵션들이 있습니다!

➡️ (https://react-svgr.com/docs/options/)


4. 커스텀 템플릿 작성
template 옵션에서 지정한 템플릿 파일(src/utils/svgr.template.js)을 생성합니다. 


```jsx
// svgr.template.js
const template = ({ componentName, props, jsx }, { tpl }) => tpl`
import React from 'react';

const ${componentName} = (${props}) => {
  return ${jsx};
};

export default ${componentName};
`;

module.exports = template;
```

이 템플릿은 SVG 컴포넌트를 표준화된 형태로 출력합니다.

## 변환 결과

<img src="https://github.com/user-attachments/assets/51f649ca-4a41-42fd-943f-7cd12dee19e8" width={600} />


위와 같은 과정을 통해 SVG 이미지 변환을 진행하게 되면, 다음과 같은 SVG파일이

```html
<?xml version="1.0" encoding="utf-8"?>
<svg width="800px" height="800px" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M7 12H17M8 8.5C8 8.5 9 9 10 9C11.5 9 12.5 8 14 8C15 8 16 8.5 16 8.5M8 15.5C8 15.5 9 16 10 16C11.5 16 12.5 15 14 15C15 15 16 15.5 16 15.5M21 12C21 16.9706 16.9706 21 12 21C7.02944 21 3 16.9706 3 12C3 7.02944 7.02944 3 12 3C16.9706 3 21 7.02944 21 12Z" stroke="#000000" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
</svg>
```

다음과 같은 컴포넌트 파일로 변환됩니다. ✨

```jsx 
interface IconComponentType {
  className: string;
  color: string;
}

export const CircleHeatSvgrepoCom = ({
  className,
  color = "#000000",
}: IconComponentType) => {
  return (
    <svg
      xmlns="http://www.w3.org/2000/svg"
      fill="none"
      viewBox="0 0 24 24"
      className={className}
    >
      <path
        stroke={color}
        strokeLinecap="round"
        strokeLinejoin="round"
        strokeWidth={2}
        d="M7 12h10M8 8.5S9 9 10 9c1.5 0 2.5-1 4-1 1 0 2 .5 2 .5m-8 7s1 .5 2 .5c1.5 0 2.5-1 4-1 1 0 2 .5 2 .5m5-3.5a9 9 0 1 1-18 0 9 9 0 0 1 18 0"
      />
    </svg>
  );
};
```


# SVGR 작동 방식
SVGR은 SVG를 React 컴포넌트로 변환하는 과정에서 다음 단계를 거칩니다

- SVGO 최적화
  - SVG 파일은 불필요한 속성과 태그를 포함하는 경우가 많습니다. SVGR은 내부적으로 SVGO를 사용해 이러한 불필요한 요소를 제거하고 파일을 최적화합니다.
- Babel 변환
  - 최적화된 SVG 파일은 Babel을 사용해 JSX 구조로 변환됩니다. 이 과정에서 SVG의 태그와 속성이 React 컴포넌트로 변환되도록 재구성됩니다.
- Replace Attributes
  - SVGR은 replaceAttrValues 옵션을 사용해 특정 속성 값을 Props로 대체하거나, 값 자체를 변경할 수 있습니다.
- Prettier 포맷팅
  - 마지막으로 생성된 JSX 코드를 읽기 쉽게 포매팅합니다.


# 느낀점

SVGR을 도입하기 전에는 새로운 SVG 아이콘을 추가할 때마다 개별적으로 컴포넌트 파일을 생성하고, SVG 코드를 복사한 뒤 Props를 수동으로 설정하는 반복적인 작업이 필요했습니다. 이러한 방식은 시간이 많이 소요되고, 실수나 누락이 발생하기 쉬워 유지보수에도 어려움이 있었습니다.

SVGR을 도입한 이후에는 SVG 파일을 자동으로 React 컴포넌트로 변환하면서 Props를 통해 크기나 색상을 동적으로 제어할 수 있어 재사용성과 유연성이 크게 향상되었습니다. 또한, 변환된 컴포넌트를 표준 디렉토리에 체계적으로 정리하고, 템플릿과 설정 파일을 활용해 규격화된 컴포넌트를 제공함으로써 관리와 유지보수가 훨씬 수월해져서 보람을 느낄 수 있었던 작업이었습니다.


👉 [프로젝트 깃허브 바로가기](https://github.com/eunbin20/svgr-generater)