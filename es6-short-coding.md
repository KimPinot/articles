---
title: ES6 축약코딩
date: 2020-02-26 03:50:00
tags: [javascript, es6]
categories:
- 개발에 도움이 되는 글
---

## 1. 삼항조건 연산자

```jsx
const answer = x > 10
	? 'greater than 10'
	: 'less than 10';
```

## 2. 간략 계산법

```jsx
// undefind & null & <empthy string>
const variable2 = variable1  || 'new';
```

## 3. 변수 선언

```jsx
let x, y, z=3;
```

## 4. if문

[Truthy한 값들](https://developer.mozilla.org/en-US/docs/Glossary/Truthy)

```jsx
// 긍정시 실행
if (likeJavaScript) {}

// 부정시 실행
if ( !a ) {}
```

## 5. For loop

```jsx
function logArrayElements(element, index, array) {
  console.log("a[" + index + "] = " + element);
}
[2, 5, 9].forEach(logArrayElements);
// logs:
// a[0] = 2
// a[1] = 5
// a[2] = 9
```

## 6. 간략 계산법

```jsx
const dbHost = process.env.DB_HOST || 'localhost';
```

## 7. 십진수 지수

```jsx
for (let i = 0; i < 1e7; i++) {}

1e0 === 1;
1e1 === 10;
1e2 === 100;
1e3 === 1000;
1e4 === 10000;
1e5 === 100000;
```

## 8. key : value

```jsx
const obj = { x, y };
```

## 9. Arrow Function

```jsx
sayHello = name => console.log('Hello', name);

setTimeout(() => console.log('Loaded'), 2000);

list.forEach(item => console.log(item));
```

## 10. 묵시적 반환

```jsx
// 한 문장 이상 반환시 {} 대신 ()를 사용함
calcCircumference = diameter => (
  Math.PI * diameter;
)
```

## 11. 파라미터 기본 값 지정

```jsx
volume = (l, w = 3, h = 4 ) => (l * w * h);
volume(2) //output: 24
```

## 12. 템플릿 리터럴 ````

```jsx
const welcome = `You have logged in as ${first} ${last}`;
const db = `http://${host}:${port}/${database}`;
```

## 13. 비구조화 할당

```jsx
const { store, form, loading, errors, entity } = this.props;
// entity === contact
const { store, form, loading, errors, entity:contact } = this.props;
```

## 14. 여러줄로 문자열 쓰기

```jsx
const lorem = `Lorem ipsum dolor sit amet, consectetur
    adipisicing elit, sed do eiusmod tempor incididunt
    ut labore et dolore magna aliqua. Ut enim ad minim
    veniam, quis nostrud exercitation ullamco laboris
    nisi ut aliquip ex ea commodo consequat. Duis aute
    irure dolor in reprehenderit in voluptate velit esse.`
```

## 15. 전개 연산자

```jsx
// joining arrays
const odd = [1, 3, 5 ];
const nums = [2 ,4 , 6, ...odd];
console.log(nums); // [ 2, 4, 6, 1, 3, 5 ]

// cloning arrays
const arr = [1, 2, 3, 4];
const arr2 = [...arr];

// 구조화 대입법과 사용 가능함.
// a, b는 따로의 객체로 인식되고, 나머지 객체를 z로 묶음
const { a, b, ...z } = { a: 1, b: 2, c: 3, d: 4 };
console.log(a) // 1
console.log(b) // 2
console.log(z) // { c: 3, d: 4 }
```

## 16. 기본 파라미터

```jsx
mandatory = () => {
  throw new Error('Missing parameter!');
}

foo = (bar = mandatory()) => {
  return bar;
}
```

## 17. `Array.find()`

```jsx
// pet.type이 Dog이고, name이 Tommy인 값을 찾음.
pet = pets.find(pet => pet.type ==='Dog' && pet.name === 'Tommy');
console.log(pet); // { type: 'Dog', name: 'Tommy' }
```

## 18. `Object['key']`

```jsx
// object validation rules (객체로 만든 validation 규칙)
const schema = {
  first: {
    required:true
  },
  last: {
    required:true
  }
}

// universal validation function (공통적으로 사용할 수 있는 validation 함수)
const validate = (schema, values) => {
  for(field in schema) {
    if(schema[field].required) {
      if(!values[field]) {
        return false;
      }
    }
  }
  return true;
}

console.log(validate(schema, {first:'Bruce'})); // false
console.log(validate(schema, {first:'Bruce',last:'Wayne'})); // true
```

## 19. 단항 비트 논리부정 연산자

```jsx
// Math.floor과 똑같음
~~4.9 === 4  // true
```
