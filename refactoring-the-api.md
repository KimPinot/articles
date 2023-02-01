---
title: "API 리팩터링 훑어보기"
description: "이 글에서는 리팩터링 2판의 챕터중 하나인 API 리팩터링 기법과 관련된 내용들을 짧고 간단하게 훑어볼 예정입니다."
date: 2023-2-1 01:43:10
tags:
  - 리팩터링
  - 사내 스터디
categories:
  - 리팩터링 2판
---

# 질의 함수와 변경 함수 분리하기

해당 리팩터링의 핵심은 **각 함수는 부수효과 (side effect) 가 없이 값을 반환해야 한다** 를 전제로 하고 있다.

Before 문의 `getTotalOutstandingAndSendBill` 함수의 경우, `emailGateway.send` 함수의 호출에 실패할 경우 값을 반환할 수 없는 문제가 발생할 것이다.

이것을 별도의 함수로 분리함으로써 `customer.invoices` 를 외부 효과 없이 가져올 수 있도록 고칠 수 있을것이다.

## Before

```ts
function getTotalOutstandingAndSendBill() {
  // 질의함수
  const result = customer.invoices.reduce(
    (total, each) => each.amount + total,
    0
  );
  // 변경함수 (만약 여기서 Error가 throw 된다면?)
  emailGateway.send(formatBill(customer));
  return result;
}
```

## After

```ts
function totalOutstanding() {
  return customer.invoices.reduce((total, each) => each.amount + total, 0);
}

function sendBill() {
  emailGateway.send(formatBill(customer));
}
```

# 함수 매개변수화하기

함수 매개변수화의 핵심은 **비슷한 로직이지만 리터럴(상수)이 다를 경우 하나의 함수로 합칠수 있다** 를 전제로 하고 있다.

다음과 같이 `usage` 에 따라 charge를 부여하는 baseCharge 함수의 경우에는 middleBand의 리터럴을 함수의 인자로 바꿀 경우
쉽게 하나의 함수로 합칠 수 있게 될 것이다.

## Before

```ts
function baseCharge(usage: number) {
  if (usage < 0) return usd(0);
  const amount =
    bottomBand(usage) * 0.03 + middleBand(usage) * 0.05 + topBand(usage) * 0.07;
  return usd(amount);
}

// usage 또는 100 중 작은 값을 리턴함
function bottomBand(usage: number) {
  return Math.min(usage, 100);
}

// usage가 100보다 작을 경우 0을, 클 경우 usage와 200 중 작은 값에서 100을 뺀 값을 리턴함
function middleBand(usage: number) {
  return usage > 100 ? Math.min(usage, 200) - 100 : 0;
}

// usage가 200보다 작을 경우 0을, 클 경우 usage에서 200을 뺀 값을 리턴함
function topBand(usage: number) {
  return usage > 200 ? usage - 200 : 0;
}
```

## After

```ts
// bottom보다 작은 값이 올 경우 0을, 클 경우 top과 usage중 작은 값에서 bottom을 뺀 값을 리턴함
function withInBand(usage: number, bottom: number, top: number) {
  return usage > bottom ? Math.min(usage, top) - bottom : 0;
}

function baseCharge(usage: number) {
  if (usage < 0) return usd(0);
  const amount =
    // usage와 top중 작은 값을 가져온다 (bottomBand의 기능)
    withInBand(usage, 0, 100) * 0.03 +
    // 상수가 인자로 빠졌을 뿐, 로직이 middleBand와 동일하다
    withInBand(usage, 100, 200) * 0.05 +
    // usage가 200보다 작을 경우 0을, 클 경우 usage에서 200을 뺀 값을 가져온다 (topBand의 기능)
    // 왜냐하면 Math.min(number, Infinity)는 항상 number를 리턴하니까
    withInBand(usage, 200, Infinity) * 0.07;
  return usd(amount);
}
```

# 플래그 인수 제거하기

플래그 인수란 **함수가 실행할 로직을 선택하기 위해 전달하는 인수** 를 뜻한다.

플래그 인수가 있는 함수는 플래그가 있는 로직과 필요 없는 로직이 한데 뭉쳐있어 코드를 한번에 파악하기 어렵게 만든다.

Before의 `bookConcert` 가 대표적인 예시인데, 프리미엄 고객 로직과 일반 고객 로직이 섞여있어 코드를 파악하기가 어려워진다.

이것을 별도의 함수로 분리함으로써 (After) 개발자가 쉽게 로직을 분리해 파악하도록 도울 수 있다.

## Before

```ts
// 여기서 isPremium이 flag 인수이다.
function bookConcert(aCustomer: Customer, isPremium: boolean) {
  if (isPremium) {
    // premium 고객 로직
  } else {
    // 일반 고객 로직
  }
}
```

## After

```ts
function premiumBookConcert(aCustomer: Customer) {
  // 프리미엄 고객 로직
}

function bookConcert(aCustomer: Customer) {
  // 일반 고객 로직
}
```

# 객체 통째로 넘기기

객체 통째로 넘기기의 핵심은 **하나의 레코드에서 둘 이상의 값을 가져와 인수를 쓰는 경우 레코드를 통째로 인자로 넣는다** 이다.

레코드를 통째로 인자로 넣을 경우 장점이 두가지 있는데.

1. 값을 가져오는 로직이 함수 안으로 들어가기 때문에, 레코드가 변경될 경우 인자를 변경할 필요 없이 함수의 로직만 수정하면 된다.
2. 매개변수 목록이 짧아져 함수의 사용법을 쉽게 이해할 수 있다.

Before에서는 withInRange 함수에 들어가는 `low` 와 `high` 변수를 After에서 레코드를 인자로 넣음으로써 두가지 장점을 얻게 되었다.

단, 함수가 레코드 자체에 의존하기를 원하지 않을때 (전역적으로 동작하는 함수일 경우)에는 이 리팩토링을 수행하지 않아도 된다.

## Before

```ts
const low: number = aRoom.daysTempRange.low;
const high: number = aRoom.daysTempRange.high;

if (aPlan.withinRange(low, high)) {
  // do some logic...
}
```

## After

```ts
aPlan.withinRange(tempRange: { low: number; high: number; }) {
  return true // or false
}

if (aPlan.withinRange(aRoom.daysTempRange)) {
  // do some logic...
}
```

# 매개변수를 질의함수로 바꾸기

매개변수를 질의함수로 바꾸기의 핵심은 **피호출함수가 쉽게 접근할 수 있는 매개변수를 삭제한다** 이다.

Before의 `availableVacation` 함수를 예시로 들자면 `anEmployee` 는 피호출함수가 쉽게 접근할 수 없지만 `anEmployee.grade` 는 피호출 함수가 쉽게 접근할수 있을것이다.

이런 매개변수를 함수의 본문에 추가하면 호출하는 입장에서 간단한 코드를 작성할 수 있을것이다.

## Before

```ts
availableVacation(anEmployee, anEmployee.grade);

function availableVacation(anEmployee: Employee, grade: EmployeeGrade) {
  // 로직...
}
```

## After

```ts
availableVacation(anEmployee);

function availableVacation(anEmployee: Employee) {
  const grade: EmployeeGrade = anEmployee.grade;
  // 로직...
}
```

# 질의함수를 매개변수로 바꾸기

매개변수를 질의함수로 바꾸기 와는 반대로, 전역 변수를 참조하거나 제거하길 원하는 원소를 참조하는 경우에는 이 리팩터링 기법을 사용할 수 있다.

Before 의 `targetTemperature` 는 `thermostat.currentTemperature` 에 의존되어 다른 온도계를 등록하지 못하는 문제가 있는데,

이런 경우에 질의함수를 매개변수로 바꿈으로써 이 문제를 해결할 수 있다.

## Before

```ts
targetTemperature(aPlan);

function targetTemperature(aPlan: Plan) {
  // thermostat 값을 글로벌로 가져오는것이 불만이다...
  const currentTemperature: number = thermostat.currentTemperature;
  // 로직...
}
```

## After

```ts
targetTemperature(aPlan, thermostat.currentTemperature);

function targetTemperature(aPlan: Plan, currentTemperature: number) {
  // 로직...
}
```

# 세터 제거하기

세터 제거하기의 핵심은 \*\*값이 변경되면 안되는 필드는 세터

## Before

```ts

```

## After

```ts

```

# 생성자를 팩터리 함수로 바꾸기

## Before

```ts

```

## After

```ts

```

# 함수를 명령으로 바꾸기

## Before

```ts

```

## After

```ts

```

# 명령을 함수로 바꾸기

## Before

```ts

```

## After

```ts

```

# 수정된 값 반환하기

## Before

```ts

```

## After

```ts

```

# 오류코드를 예외로 바꾸기

## Before

```ts

```

## After

```ts

```

# 예외를 사전확인으로 바꾸기

## Before

```ts

```

## After

```ts

```
