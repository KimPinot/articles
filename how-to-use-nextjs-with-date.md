---
title: NextJS에서 Date 객체를 사용하는 방법 3가지
description: NextJS SSR에서 기본적으로 지원하지 않는 Date, Map, Set 과 같은 객체를 사용하는 방법 3가지를 장/단점과 함께 정리했습니다.
date: 2022-09-02 00:44:31
tags: [nextjs]
categories:
- 개발에 도움이 되는 글 
---

안녕하세요. 시간과 함께하는 개발자 나비입니다.

이 글에서는 NextJS SSR에서 Date 객체를 Props으로 넘기는 방법을 알아보고자 합니다.

# TL;DR

- NextJS에서 Date 객체를 SSR Props으로 넘기는걸 지원하지 않는 이유는 퍼포먼스를 위해서
- [blitz-js/next-superjson-plugin](https://github.com/blitz-js/next-superjson-plugin) 이거 쓰세요
- 모든 props을 직접 인코딩한다는 발상은 좋지만, 관리하기가 힘듭니다.

# Date, 넌 누구냐

NextJS에서는 `getServerSideProps`, `getStaticProps` 처럼 SSR 또는 SSG 단계에서 Props을 내려주는 기능을 가지고 있습니다.

코드로 작성한다면 이런 느낌이네요.

```tsx
import { NextPage, GetServerSideProps } from 'next';

interface Props {
  message: string;
}

const Page: NextPage<Props> = ({ message }) => {
  return <div>{message}</div>;
}

export const getServerSideProps: GetServerSideProps = () => {
  return {
    props: {
      message: "Hello World!"
    }
  }
}
```

여기까지는 괜찮지만... 만약 여러분들이 이 글을 작성한 날짜를 Props으로 넘긴다면 무슨 일이 일어날까요?

```tsx
import { NextPage, GetServerSideProps } from 'next';

interface Props {
  createdAt: Date;
}

const Page: NextPage<Props> = ({ createdAt }) => {
  return <div>I Wrote this article at {createdAt.toIsoString()}</div>;
}

export const getServerSideProps: GetServerSideProps = () => {
  return {
    props: {
      // 그거 아셨나요? JS Date 객체의 month는 0부터 시작합니다!
      createdAt: new Date(2022, 7, 21, 12, 24, 36)
    }
  }
}
```

이런 오류를 내며 서비스가 망가져버리고 맙니다.

```text
Error: Error serializing `.meta.date` returned from `getStaticProps` in "/[filename]".
Reason: `object` ("[object Date]") cannot be serialized as JSON.
Please only return JSON serializable data types.
```

이런 오류는 왜 발생하는 걸까요?

구글에 'Why NextJS app broken use date object?' 이라고 검색해봅니다.

[Reason: `object` ("[object Date]") cannot be serialized as JSON. Please only return JSON serializable data types](https://stackoverflow.com/questions/70449092/reason-object-object-date-cannot-be-serialized-as-json-please-only-ret) 라는 이름의 스택 오버플로우가 보입니다.

> Looks like nextJS doesn't like serializing anything but scalar types for performance reasons.
> You can read more in this [github issue](https://github.com/vercel/next.js/issues/13209#issuecomment-633149650).
> Best way you can handle this is that you convert your Date objects to UNIX timestamp before returning them.

아하! NextJS 에서는 특정 (Date, Map, Set) 타입을 퍼포먼스 이슈로 인해 지원하지 않는다고 하네요.

그럼 이 문제를 어떻게 해결할 수 있을까요?

## 방법 1 : 직접 변환한다

우선 가장 먼저 드는 생각을 `Date` 객체를 타임스탬프로 변환하는것이 방법일 것 같아요.

```tsx
import { NextPage, GetServerSideProps } from 'next';

interface Props {
  createdAt: number;
}

const Page: NextPage<Props> = ({ createdAt }) => {
  return <div>I Wrote this article at {new Date(createdAt).toIsoString()}</div>;
}

export const getServerSideProps: GetServerSideProps = () => {
  return {
    props: {
      createdAt: +new Date(2022, 7, 21, 12, 24, 36)
    }
  }
}
```

흠... 이렇게 createdAt이 한개라면 괜찮겠지만, 두가지 문제점이 보이고 있습니다.

1. 일일히 변환하는게 너무 불편하다.

2. createdAt을 사용하는 부분에서 반드시 `Date` 객체로 계속 변환 해줘야한다.

3. 만약 `Date` 객체를 사용하는 컴포넌트가 3,000 개라면...? 😱😱😱

## 방법 2 : 외부 라이브러리를 사용한다

구글에 이 문제를 해결할 수 있는 해결사를 찾아봅니다.

그랬더니 [remorses/next-superjson](https://github.com/remorses/next-superjson) 이라는 이름의 라이브러리가 보입니다.

레포지토리에 들어가보니 이렇게 사용할 수 있다고 합니다.

```javascript
// next.config.js
const { withSuperjson } = require('next-superjson');

module.exports = withSuperjson()({});
```

문제는 해결되었지만, 이런 불편한 점이 있었습니다.

1. `module.exports` 에 `withSuperjson` 을 넣어줘야 한다.

    - wrapper를 추가할 때마다 depth가 항상 늘어난다 => 코드를 보기가 불편해진다

      ```javascript
      const { withSuperjson } = require("next-superjson");
      const withAnotherWrapper = require("with-another-wrapper");
      const withAnotherAnotherWrapper = require("with-another-another-wrapper");

      module.exports = withAnotherAnotherWrapper(withAnotherWrapper(withSuperjson()({})));
      ```


## 방법 3 : NextJS 플러그인을 사용한다

다시 검색해봅니다.

NextJS 12.2 버전부터는 SWC 플러그인을 지원하는데요 [NextJS 12.2 - SWC Plugins (Experimental)](https://nextjs.org/blog/next-12-2#swc-plugins-experimental)

이에 맞는 기능을 지원하는 [blitz-js/next-superjson-plugin](https://github.com/blitz-js/next-superjson-plugin) 라이브러리를 찾았습니다. (야호!)

이 플러그인을 추가하려면 `superjson`, `next-superjson-plugin` 디펜던시를 설치하고 next.cofig.js를 이렇게 수정하면 플러그인이 작동한다고 합니다.

```javascript
// next.config.js
module.exports = {
  experimental: {
    swcPlugins: [
      ["next-superjson-plugin", {}],
    ],
  },
}
```

이제 코드를 실행시켜보면...

```tsx
import { NextPage, GetServerSideProps } from 'next';

interface Props {
  createdAt: number;
}

const Page: NextPage<Props> = ({ createdAt }) => {
  return <div>I Wrote this article at {new Date(createdAt).toIsoString()}</div>;
}

export const getServerSideProps: GetServerSideProps = () => {
  return {
    props: {
      createdAt: new Date(2022, 7, 21, 12, 24, 36)
    }
  }
}
```

Date를 타임스탬프로 변환시키지 않아도, 정상적으로 동작합니다!

# 느낀점

Date 객체를 사용하기 위해서 많은 방법을 시도해봤는데, 힘들었지만 재미있었던 기억으로 남았던 것 같습니다.

NextJS SWC Plugins가 추가되고 디펜던시를 하나 설치만 해도 동작한다는 점이 너무나도 신선했고, 편리했습니다.

여러분들도 쉽고 빠르며 편리한 NextJS 한입 어떠신가요?