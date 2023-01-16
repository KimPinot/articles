---
title: ReScript 바인딩 101
description: ReScript의 함수 / 모듈 바인딩을 알아봅시다
date: 2023-1-15 03:41:00
categories:
  - 개발에 도움이 되는 글
---

이번시간에는 ReScript 바인딩에 대해서 알아보도록 하겠습니다.

# ReScript란?

ReScript는 JavaScript를 위한 새로운 함수형 언어입니다.

특정 문법에 따라서 코드를 작성하면 읽기 쉬우며 성능도 뛰어난 JS 코드로 컴파일 됩니다.

이 과정에서 강력한 타입 힌트를 제공해 별도의 타입 주석이나 타입 정의 없이도 간단하게 사용할수 있다는것이 특징입니다.

# ReScript 바인딩이란?

ReScript 바인딩은 ReScript로 작성되지 않은 JS/TS 코드를 ReScript에서 사용할수 있게 바꾸는 일종의 타입 정의입니다.

타입스크립트와의 타입 지정과는 어떻게 타입을 지정하느냐에 따라서 컴파일되는 자바스크립트의 결과가 다르다는 특징이 있습니다.

# 모듈 바인딩 해보기

## default export 된 모듈 바인딩

`export default function something()` 와 같은 함수는 어떻게 바인딩 할 수 있을까요?

이때 사용할 수 있는 코드가 두개나 있습니다.

- `@module("something")` 을 사용하여 something이란 이름의 모듈을 가져올 수 있습니다.

```typescript
@module("something") external something: unit => string = "default"
```

이 코드는 이렇게 컴파일됩니다.

```js
import Something from "something";
```

## named export 된 모듈 바인딩

그러면 `export function something()` 와 같은 함수는 어떻게 바인딩 할 수 있을까요?

바로 아래 코드를 사용하면 됩니다 :tada:

```typescript
@module("something") external something: unit => string = "something"
```

이 코드는 이렇게 컴파일 됩니다

```js
import { something } from "something";
```

생각보다 쉽죠?

# 함수 바인딩 해보기

그러면 `setTimeout()`, `Math.random()` 같은 함수는 어떻게 바인딩 할 수 있을까요?

물론 `Js.Global.setTimeout()` 이나, `Js.Math.random()` 같은 좋은 내장 모듈이 있지만, 직접 만들어보도록 합시다.

## `setTimeout()` 바인딩 해보기

- `@val` 을 사용해서 모듈에 없는 함수를 바인딩 할 수 있습니다.

```typescript
@val external setTimeout: (unit => unit, int) => float = "setTimeout"
@val external clearTimeout: float => unit = "clearTimeout"
```

하지만 이렇게 함수를 바인딩 한다면 이런것도 가능합니다.

```js
const id = 3.141592;
cleartTimeout(id);
```

왜냐하면 우리는 인자로 `float` 타입을 받기로 바인딩했기 때문이죠.

이런것들을 막기 위해서는 별도의 타입을 추가함으로써 막을 수 있습니다.

```typescript
type timerId
@val external setTimeout: (unit => unit, int) => timerId = "setTimeout"
@val external clearTimeout: timerId => unit = "clearTimeout"

let id = setTimeout(() => Js.log("hello"), 100)
clearTimeout(id)
```

```js
var id = setTimeout(function (param) {
  console.log("hello");
}, 100);

clearTimeout(id);
```

## 전역 함수 / 변수 가져와보기

- `@scope` 를 사용해서 특정 모듈을 선택할 수 있습니다.

```typescript
@scope("Math") @val external random: unit => float = "random"
let someNumber = random()
```

```js
var someNumber = Math.random();
```

- `window.location.href` 와 같이 복잡한 바인딩도 할수 있습니다.

```typescript
@val @scope(("window", "location"))
external href: string = "href"
let locaiton = href
```

```js
var locaiton = window.location.href;
```

# 잡기술 알아보기

## 메소드 체이닝

`express().use().use()` 와 같이 메소드를 연속해서 사용하는것을 메소드 체이닝 (Method Chaining)이라고 부릅니다.

이런 코드가 있을때

```typescript
type express

@module("express") external express: unit => express = "default"
@send external use: (express, string) => express = "use"

let methodChaining = () => express() -> use("something") -> use("something")
```

자바스크립트에서는 이렇게 컴파일 됩니다

```js
express().use("something").use("something");
```

## 타입 변환하기

ReScript에서 string을 특정한 타입으로 바꾸고 싶을때는 이 코드를 사용할 수 있습니다.

```typescript
type filename
@module external stringtoFilename: string => filename = "%identity"
```

여기서 `%identity` 는 아무런 동작도 하지 않고 단지 타입을 변환시키는 용도로만 사용됩니다.

더욱 자세한 설명은 [공식 문서](https://rescript-lang.org/docs/manual/latest/bind-to-global-js-values#global-modules) 에 있습니다!

모두 재밌는 ReScript 하세요~~~
