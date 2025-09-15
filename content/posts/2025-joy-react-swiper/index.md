---
title: react에서 swiper 잘 쓰기
author: 조이
date: 2025-09-15
description: react에서 swiper 잘 쓰기
---

react에서 스와이퍼 모달을 쓰면서 겪었던 네비게이션 관련된 반복적인 이슈 발생을 막기위해 기록으로 기억하려고 작성하는 글입니다.<br>

## 개요

---

### 문제상황

1. ref를 이용한 네비게이션 버튼이 작동하지 않음
2. 로컬에서는 이상없고, 실배포한 환경에서만 작동하지 않음

## 본론

---

### 수정 전

- 기존에는 navigation 버튼을 null로 초기화 해놓고, onInit을 이용해 Swiper가 내부적으로 초기화될 때 호출하도록 설정

```react
  ...

  const navigationPrevRef = useRef(null);
  const navigationNextRef = useRef(null);

  const swiperInit = (swiper: any) => {
    swiper.params.navigation.prevEl = navigationPrevRef.current;
    swiper.params.navigation.nextEl = navigationNextRef.current;
    swiper.navigation.init();
    swiper.navigation.update();
  };

  return(
    ...
    <Swiper
    onInit={swiperInit}
    rewind={true}
    navigation={true}
    modules={[Navigation, Pagination, Autoplay, Pagination]}
    ...
    >
    ...
    </Swiper>
    ...
    <Box className="text-16-r-2 pagination" role="button">
        <Image
            src="/images/rolling_left.png"
            alt="prev"
            ref={navigationPrevRef}
        />
        <Image
            src="/images/rolling_right.png"
            alt="next"
            ref={navigationNextRef}
        />
    </Box>
    ...
  )

```

### 수정 전 코드의 문제점

1. ref 타이밍 문제

- ref 값(navigationPrevRef.current, navigationNextRef.current)이 초기 렌더 시 null일 수 있음.
- 배포 환경에서는 DOM 생성 타이밍이 엄격하게 적용되어 ref가 설정되지 않을 수 있음.

2. 네비게이션 버튼이 swiper 내부를 참조하고 있지 않음

- Swiper의 navigation.prevEl, nextEl이 Swiper 외부에 위치해있어 Swiper가 해당 DOM을 제대로 인식하지 못할 가능성 있음.

++ 추가적으로 확인해야 할 것, z-index 의 값

## 수정 후

- useEffect, onSwiper을 이용해서 swiper가 렌더 된 이후에 navigation 연결
- swiper.navigation.init() 및 update()을 수동으로 호출

```react
  ...

  const navigationPrevRef = useRef(null);
  const navigationNextRef = useRef(null);

  useEffect(() => {
    if (
      swiperInstance &&
      navigationPrevRef.current &&
      navigationNextRef.current
    ) {
      swiperInstance.params.navigation.prevEl = navigationPrevRef.current;
      swiperInstance.params.navigation.nextEl = navigationNextRef.current;
      swiperInstance.navigation.init();
      swiperInstance.navigation.update();
    }
  }, [swiperInstance]);

  return(
    ...
    <Swiper
    onSwiper={setSwiperInstance}
    rewind={true}
    navigation={true}
    modules={[Navigation, Pagination, Autoplay]}
    ...
    >
    ...
    </Swiper>
    ...
    <Box className="text-16-r-2 pagination" role="button">
        <Image
            src="/images/rolling_left.png"
            alt="prev"
            ref={navigationPrevRef}
        />
        <Image
            src="/images/rolling_right.png"
            alt="next"
            ref={navigationNextRef}
        />
    </Box>
    ...
  )

```

### 수정 후 코드에 적용한 해결방법

1. ref 타이밍 문제

- ref 값(navigationPrevRef.current, navigationNextRef.current)이 초기 렌더 시 null일 수 있음.
- 배포 환경에서는 DOM 생성 타이밍이 엄격하게 적용되어 ref가 설정되지 않을 수 있음.

**-> React에서 DOM은 렌더링 이후에야 완전히 구성되기 때문에, ref에 연결된 DOM이 Swiper보다 늦게 준비되는 경우가 생김. 특히 배포 환경에서는 렌더 타이밍이 더 빠르거나 SSR 처리 차이로 인해 문제가 드러나기도 함. <U>swiper 렌더 이후 실행되도록 지연이 필요.</U>**<br>
**-> swiper에서는 ref를 직접 넣는 방식을 지원하지 않고 있기 때문에 onSwiper 사용**

> Q. ref를 swiper에 연결하는 방법은 onInit도 있는데 onSwiper를 사용하는 이유는?<br>
> A. onInit는 초기 렌더 중에도 실행되는 반면 <U>onSwiper는 Swiper 인스턴스를 React로 넘겨주는 역할이기 때문에 렌더 이후 실행되므로 안정적이기 때문</U>

> Q. 지연하는 방법으로 setTimeout이 아닌 useEffect를 사용하는 이유는?<br>
> A. DOM의 준비상태를 보장 못하는 setTimeout보다는 <U>useEffect를 사용하는 것이 적절함</U><br>

<br>

2. 네비게이션 버튼이 swiper 내부를 참조하고 있지 않음

- Swiper의 navigation.prevEl, nextEl이 Swiper 외부에 위치해있어 Swiper가 해당 DOM을 제대로 인식하지 못할 가능성 있음.

**-> useEffect 내부에서 ref.current를 확인하고 navigation을 설정하고, swiper.navigation.init() 및 update() 수동 호출**<br>
-> 처음에는 swiper 외부를 참조하고 있는 것을 막기 위해 swiper에 navigation과 onBeforeInit을 추가했었는데, onSwiper를 호출하면서 중복된 코드라는 것을 깨닫고 수정함.

> Q. useEffect 내부에서 네비게이션을 연결하고 재초기화 하는 이유?<br>
> A. onSwiper는 swiper가 초기화 된 이후 실행, 그런데 navigation 모듈은 이미 실행 끝임. 이때 prevEl/nextEl을 바꾸더라도 Swiper는 이미 navigation 버튼을 찾으려고 시도한 후이기 때문에, 다시 navigation을 자동으로 초기화하지 않음. 그렇기 때문에 네비게이션을 다시 연결하고 재초기화 하는 과정이 필요.

<br>

## 실서버에서만 일어나는 문제를 로컬에서 확인하고 싶다면?

```
npm run build
npm run start
```

## 결론

### 마치며

자주 쓰는 라이브러리나 라이프사이클이 아니라면 모든 라이브러리 그리고 라이브러리 내부 옵션들의 실행시점을 파악하기는 어렵다고 생각했다. 그런데 그것을 모두 파악하고 있지 않더라도 렌더링 시점을 궁금해하고 이슈가 발생했을때 렌더링 시점을 분석하고 파악해 가는 과정이 더 중요하다는 생각이 든다. 지금 글을 쓰면서도 react, react swiper에서 더 권장하는 방향은 무엇인지 찾아보며 수정을 거듭했다. (정답이 있다면) 정답과 가까워지는 느낌이 들긴 하지만 여전히 부족하다는 생각이 든다. 그리고 로컬에서는 잘 되지만 실서버에서는 안되는 이슈들을 가끔 보는데, 처음에는 원인조차 감이 안잡히고 당황했지만 이번 이슈를 통해서 렌더링 시점이 원인이 될 수도 있다는 것을 배운 것 같다.&#x1F613;
