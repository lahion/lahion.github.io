---
title: D3를 활용한 고용량 SVG 에셋 처리
author: 태더
date: 2025-02-13
description: Next.js에서 고용량 SVG 에셋을 처리하는 방법을 소개합니다.
---

## 개요

최근 Next.js 프로젝트에서 고용량 SVG 에셋을 처리할 일이 있었습니다.
이 글에서 말하는 고용량 SVG는 5MB 이상의 파일을 말하는데, 이러한 파일이 프로젝트 내 다수 존재할 경우 컴파일을 해도 성능이 좋지 않을 뿐 더러, 개발 서버에서는 메모리 사용량이 매우 많아지게 됩니다.
이 글에서는 이러한 이슈를 해결하기 위해, D3 라이브러리를 활용하여 SVG를 로드하고 이벤트를 처리하는 방법을 소개합니다.

| 서비스                   | 버전                     |
| ------------------------ | ------------------------ |
| <center>Next.js</center> | <center>14.2.23</center> |
| <center>D3</center>      | <center>7.9.0</center>   |

---

## 문제 (요구사항)

- svg 요소로 렌더링 되어야 한다.
- svg 의 하위 path 요소에 대해서 Click, Hover 이벤트를 처리할 수 있어야 한다.
- Next.js 개발 서버로 실행시 string으로 로드되지 않아야 한다. (메모리 사용량 이슈)

### 실패케이스: next/image 사용

```typescript
import Image from "next/image"

export default function SvgComponent() {
  return (
    <Image
      alt="Test"
      src="/path/to/image.svg"
      alt="SVG"
      width={100}
      height={100}
    />
  )
}
```

```html
<img
  alt="Test"
  loading="lazy"
  width="100"
  height="100"
  decoding="async"
  data-nimg="1"
  style="color:transparent"
  src="/path/to/image.svg"
/>
```

- 이처럼 img 태그로 렌더링되기 때문에, 이벤트를 처리할 수 없다.

### 실패케이스: @svgr/webpack 사용

- 빌드 단계에서 svg 파일을 컴포넌트로 만들어 주는 라이브러리
- 설정 방법은 [공식 문서](https://react-svgr.com/docs/next/) 참고

```typescript
import ImageSvg from "/path/to/image.svg"

export default function SvgComponent() {
  return <ImageSvg width={100} height={100} />
}
```

```html
<svg width="100" height="100">...</svg>
```

- svg로 렌더링은 되지만, svg에 포함된 string이 개발 서버에서 문자열로 로드되어 메모리 사용량이 많아지는 문제 발생
- 5~10MB 이상의 svg 파일 10개 내외 사용시 메모리 점유율이 8기가 이상으로 증가
- **svg에 포함된 base64 이미지가 전부 문자열로 로드 됨**

### 성공케이스: d3 라이브러리 사용

```bash
$ npm install d3 @types/d3
```

```typescript
import * as d3 from "d3"
import { MouseEvent, useEffect } from "react"

type Props = {
  onClick: (e: MouseEvent) => void
}

export default function SvgComponent(props: Props) {
  const id = "svg-key"

  useEffect(() => {
    d3.xml("/path/to/image.svg").then((data: any) => {
      d3.select(`#${id} svg`).remove()
      d3.select(`#${id}`).append(() => data.documentElement)
    })
  }, [])

  return <div id={id} {...props}></div>
}
```

```html
<svg width="100" height="100">...</svg>
```

- StrictMode 등에서 중복 렌더링될 수 있기 때문에 append 전에 remove 처리
- svg 요소로 렌더링되어 이벤트 처리가 가능
- 개발 서버에서 문자열로 로드되지 않아 메모리 사용량이 적음

### 주의 사항

- d3.xml 함수는 파일에 포함된 모든 문자열을 처리하기 때문에 minified svg가 아니라면 불필요한 text 노드가 생성된다. (개행, 스페이스 등)
- 렌더링 시 remove 함수를 통해 중복 레더링 되는 경우를 사전에 방지해야한다.
- 만약 svg를 SSR에서 렌더링해야하는 경우 jsdom 라이브러리를 활용하여 가상의 document를 생성한 후 CSR에 전달될 수 있도록 구현해야 한다. (hydration 이슈 방지)

## 참고

- [D3 XML](https://d3js.org/d3-fetch#xml)
- [Load external SVGs with d3](http://zevross.com/blog/2019/08/20/load-external-svgs-with-d3-v5/)
- [How to inject an external SVG with D3.js](https://www.fabiofranchino.com/blog/how-to-inject-external-svg-with-d3/)
