# Theory

## 타입
### let

변수

### const

상수. 변하지 않는다.

### var

let처럼 값을 바꿀 수 있다. 하지만 같은 변수명으로 선언해도 정상적으로 작동한다.

```javascript
var name = 'a';
console.log(name);   // a

var name = 'b';
console.log(name);   // b
```

따라서 코드량이 많이질 수록 상태를 파악하기가 힘들다.

## Function
### 콘솔

console.log도 함수다. 콘솔에서 스트링 처리하는 법을 알아보자.

자바스크립트는 싱글, 더블 quote보다 백틱을 사용한다.

```javascript
function sayHello(name, age) {
  console.log('Hello' + name + "you are " + age);
}
```

위 대신에,

```javascript
function sayHello(name, age) {
  console.log(`Hello ${name} you are ${age}`);
}
```

이렇게 사용하는 것이다.

```javascript
function sayHello(name, age) {
  console.log(`Hello ${name} you are ${age}`);
}

// greetNicolas는 함수의 리턴값이다.
const greetNicolas = sayHello("Nicolas", 14);

console.log(greetNicolas);
```

위의 코드는 실행하면 다음과 같은 결과가 나온다.

```text
Hello Nicolas you are 14
undefined
```

왜 undefined가 뜰까? 함수 정의 부분에서 콘솔 로그만 찍을 뿐 아무 것도 리턴하지 않기 때문이다.

```javascript
function sayHello(name, age) {
  // 값을 리턴한다.
  return `Hello ${name} you are ${age}`;
}

const greetNicolas = sayHello("Nicolas", 14);

console.log(greetNicolas);
```

```text
Hello Nicolas you are 14
```

## DOM(Document Object Module) 조작

```javascript
console.log(document);
```

document는 HTML 요소를 리턴하는 객체다.

```javascript
const title = document.getElementById("title");
title.innerHTML = "Hi! From JS";
```

innerHTML을 쓰면 `Hi! From JS`가 화면에 출력된다.

```javascript
const title = document.getElementById("title");
// 내용을 조회 후
console.dir(title);
// 활용한다.
title.style.color = "red";
```

dir는 title에 대한 이벤트, 속성 등의 정보를 가져온다.

## Event and event handlers

```javascript
const title = document.querySelector("#title");

function handleResize() {
    console.log("I have been resized");
}

// 이벤트 받기를 기다린다. 이때 두 번째 인자로 이벤트를 다룰 함수를 넘긴다.
// 함수에 괄호 안 치는 것에 주의한다. 
// ()를 안 쓰면 내가 필요할 때 즉, 윈도우가 resize 될 때 호출한다. 
// ()를 쓰면 지금 바로 호출이다.
window.addEventListener("resize", handleResize);
```

이벤트 리스너의 두 번째 인자를 주의하자.

```javascript
const title = document.querySelector("#title");

// event 인자는 어디서 온 걸까? 
// 이벤트를 다룰 함수를 만들 때마다 자바스크립트가 자동으로 event를 붙인다.
function handleResize(event) {
    console.log(event);
}

// 이벤트가 발생할 때마다 event 객체가 호출되어 찍힌다.
window.addEventListener("resize", handleResize);
```

```javascript
const title = document.querySelector("#title");

function handleClick() {
    title.style.color = 'red';
}

// 타이틀을 클릭할 때마다 빨간색으로 바뀐다.
title.addEventListener("click", handleClick);
```