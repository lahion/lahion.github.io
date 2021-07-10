---
layout: post
title: Nuxt + Swiper 에서 발생한 버그 수정기 (1)
description: Nuxt 환경에서 vue-awesome-swiper를 활용하여 Swiper 기능을 제공할 때 발생한 버그를 수정한 내용입니다.
img: swiper.png
tags: [Swiper, vue-awesome-swiper, nuxt, vue, SSR]
---

LAH 개발팀은 Nuxt로 프론트엔드를 개발합니다.\\
요즘은 대부분의 서비스들이 [Swiper](https://swiperjs.com/){:target="_blank"}를 사용하고 있죠.\\
비메이트도 마찬가지로 많은 곳에서 Swiper가 사용되고 있습니다.\\
아마 Vue나 Nuxt로 개발하는 많은 팀들이 겪었을 문제같은데, 알려주는 곳이 없었습니다.

## 문제 현상

메인 페이지에 Swiper를 설정하고 slidesPerView를 4로 설정하였습니다.\\
당연히 잘 나올걸 기대했으나,\\
페이지 렌더링할 때와 페이지 라우팅할 때 스타일이 깨지는 문제가 있습니다.\\
\\
![문제 현상]({{site.baseurl}}/assets/img/2021-07-10/first.gif)
\\
## 페이지 렌더링할 때 개선

렌더링할 때 서버사이드에서 가로길이를 결정하고 띄워주면 렌더링할 때 문제가 없겠지만\\
안타깝게도 swiper의 경우 페이지 렌더링이 된 후에 가로길이를 계산해서 style에 넣게됩니다.\\
따라서 서버사이드에서 레이아웃이 구성된 채로 띄워줄수는 없는 거죠.\\
\\
그래서 다른 서비스처럼.. skeleton을 추가했습니다.\\
\\
```html
<template v-if="is_loading">
  <actor-portfolio-card skeleton />
  <actor-portfolio-card skeleton />
  <actor-portfolio-card skeleton />
  <actor-portfolio-card skeleton />
</template>
<div class="swiper" v-swiper:actorPortfolio="actorPortfolioSwiperOption" v-show="!is_loading">
  ...
</div>
```
\\
주의해야할 점은 v-if의 경우 조건에 따라 Element를 추가/삭제하고,\\
v-show는 Element는 존재하지만 display 속성만 조정하기 때문에 더 자연스럽게 보여줄수 있습니다.\\
하지만 skeleton의 경우 아예 없어지는 것이 좋겠죠.\\
(자연스러운 skeleton을 위해 gif 이미지를 사용하지 않고 animation을 썼기 때문에 필요없을 때는 없애야 할 것 같습니다.)\\
\\
![렌더링할 때 개선후]({{site.baseurl}}/assets/img/2021-07-10/second.gif)
\\
## 라우팅할 때 개선

렌더링할 때 스타일이 늦게 적용되는 것은 충분히 이해가 되었으나\\
라우팅할 때 스타일이 삭제 되는 것은 갈피를 잡을 수 없었습니다.\\
그러다 결국, 크롬 디버깅에서 attribute breakpoint까지 걸어서 확인을 했습니다.\\
\\
![attribute breakpoint 사용]({{site.baseurl}}/assets/img/2021-07-10/breakpoint.png)
\\
물론 디버깅 과정에서 더 많은 내용을 봤지만,\\
Resume script 버튼을 누르면서 스타일이 삭제 되는 지점을 확인했습니다.\\
\\
![attribute breakpoint 사용]({{site.baseurl}}/assets/img/2021-07-10/error.png)
\\
스타일이 삭제되면서 레이아웃이 깨지는 순간 Call Stack을 확인해보니\\
Swiper가 destroy 함수를 실행합니다.\\
조사를 해보니 페이지 렌더링이나 라우팅할 때 swiper 컴포넌트 또는 Element가\\
cleanup과정으로 destroy를 실행한다고 합니다.\\
\\
그 이후.. 더 조사해보니 이미 공식 문서에 옵션이 있었습니다.\\
\\
```javascript
swiper.destroy(deleteInstance, cleanStyles)
/*
deleteInstance - boolean - Set it to false (by default it is true) to not to delete Swiper instance
cleanStyles - boolean - Set it to true (by default it is true) and all custom styles will be removed from slides, wrapper and container. Useful if you need to destroy Swiper and to init again with new options or in different direction
*/
```
\\
아, destroy 함수를 호출하면서 cleanStyles를 false로 설정하면 될 것 같습니다.\\
그런데 개발팀은 destroy를 호출한 적이 없습니다. (?)\\
바로.. [vue-awesome-swiper](https://github.com/surmon-china/vue-awesome-swiper){:target="_blank"} 에서 호출하고 있었습니다.\\
<del>역시 래핑된 컴포넌트를 사용하면 확실히 장단점이 있네요</del>

Issues에서 destroy를 검색하니 관련 이슈가 검색됩니다.\\
[Swiper collapses on route change](https://github.com/surmon-china/vue-awesome-swiper/issues/317){:target="_blank"} 이름으로 이슈가 등록되어있는데\\
해당 이슈를 해결하기 위해 옵션이 추가되어있었습니다.\\
즉 vue-awesome-swiper 공식 문서를 봤다면 알 수 있었겠죠.\\
\\
```javascript
<swiper
  :options="swiperOptionsObject"
  :auto-update="true"
  :auto-destroy="true"
  :delete-instance-on-destroy="true"
  :cleanup-styles-on-destroy="true"
  @ready="handleSwiperReadied"
  @click-slide="handleClickSlide"
/>
```
\\
v4.x에서 예시와 같이 cleanup-styles-on-destroy와 delete-instance-on-destory 옵션이 추가되었습니다.\\
그래서 테스트를 해보았으나 안됩니다.\\
\\
LAH에서는 직접 컴포넌트 태그를 사용하지 않고\\
v-swiper directives 를 사용해서 구현했기 때문인지.. 안되네요.\\
결국 차선책으로 destroy 함수를 오버라이딩하여 cleanup하지 않도록 했습니다.\\
\\
```javascript
mounted() {
  this.actorPortfolio.destroy = () => {}
}
```
\\
결국 해결했습니다.\\
\\
![라우팅할 때 개선]({{site.baseurl}}/assets/img/2021-07-10/third.gif)
