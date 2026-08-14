# JavaScript 문법 코드 심화 분석

JavaScript는 문법 자체보다 코드가 실행되는 과정과 구조를 이해하는 것이 중요하다.

이 문서는 JavaScript 코드가 어떻게 동작하는지 심화 분석한다.

---

## 1. 변수 선언 분석 - 메모리 관점

### 코드

```javascript
let score = 100;
```

### 분석

위 코드는 세 부분으로 나눌 수 있다.

```
let     score      = 100;
│        │           │
│        │           └─ 값 (Value)
│        └───────────── 변수 이름 (Identifier)
└────────────────────── 변수 선언 방식 (Keyword)
```

### JavaScript 엔진의 처리 과정

1. **메모리 할당 (Memory Allocation)**
   - JavaScript 엔진이 메모리에서 특정 공간을 예약
   - 이 공간에 고유한 메모리 주소가 부여됨

2. **값 저장 (Value Storage)**
   - 100이라는 값을 할당된 메모리에 저장
   - 이진수 형태로 변환되어 메모리에 기록됨

3. **식별자 바인딩 (Identifier Binding)**
   - `score`라는 이름을 메모리 주소와 연결
   - 이후 `score`로 접근하면 해당 메모리 주소의 값을 반환

### 메모리 구조 시각화

```
메모리 주소      변수명      값
─────────────────────────────────
0x0001    →     score   →   100
0x0002    →     name    →   "John"
0x0003    →     active  →   true
```

### var, let, const 차이 - 스코프 관점

```javascript
// var - 함수 스코프
function testVar() {
  if (true) {
    var x = 10;
  }
  console.log(x);  // 10 (블록 스코프 무시)
}

// let - 블록 스코프
function testLet() {
  if (true) {
    let y = 20;
  }
  console.log(y);  // ReferenceError: y is not defined
}

// const - 블록 스코프 + 재할당 금지
function testConst() {
  const z = 30;
  z = 40;  // TypeError: Assignment to constant variable
}
```

### 호이스팅(Hoisting) 차이

```javascript
console.log(a);  // undefined (var는 호이스팅됨)
var a = 1;

console.log(b);  // ReferenceError (let은 temporal dead zone)
let b = 2;
```

---

## 2. const 분석 - 깊은 복사 vs 얕은 복사

### 코드

```javascript
const player = {
  name: "Hero",
  hp: 100
};

player.name = "Knight";  // 가능
player.hp = 50;          // 가능
```

### 일반적인 오해

많은 사람이 `const`는 완전히 변경 불가능하다고 생각한다.

**❌ 오해:** `const player = {}` → 이 객체는 절대 변경 불가능

**✅ 정확:** `const` 키워드는 변수의 **참조(Reference)를 고정**하는 것

### 메모리 관점에서의 이해

```javascript
const player = { name: "Hero" };
// 메모리 구조:
// 변수명: player
// 메모리 주소: 0x0010
// 저장된 값: 객체의 참조(reference) → 0x1000
// 0x1000번지의 객체: { name: "Hero" }

// 가능: 객체 내부 데이터 변경 (0x1000번지의 값 변경)
player.name = "Knight";

// 불가능: 새로운 객체로 재할당 (0x0010번지에 다른 참조 저장)
player = { name: "Warrior" };  // TypeError
```

### 참조(Reference) vs 원시값(Primitive)

```javascript
// 원시값: 값 자체가 메모리에 저장
const num = 100;
const str = "Hello";
const bool = true;

// 참조값: 메모리 주소(참조)가 저장
const obj = { value: 100 };
const arr = [1, 2, 3];
const func = function() {};
```

### 깊은 복사(Deep Copy) vs 얕은 복사(Shallow Copy)

```javascript
// 얕은 복사 - 참조만 복사
const original = { name: "Hero", stats: { hp: 100 } };
const shallow = { ...original };

shallow.name = "Knight";       // original에 영향 없음
shallow.stats.hp = 50;         // original.stats.hp도 50으로 변경됨!

console.log(original.stats.hp);  // 50 (깊은 객체는 여전히 같은 참조)

// 깊은 복사 - 모든 레벨을 완전히 복사
const deep = JSON.parse(JSON.stringify(original));
deep.stats.hp = 100;
console.log(original.stats.hp);  // 50 (변경 없음)
```

---

## 3. 함수 분석 - 함수 선언 vs 함수 표현식

### 코드

```javascript
// 함수 선언식 (Function Declaration)
function attack() {
  console.log("Attack");
}

// 함수 표현식 (Function Expression)
const defend = function() {
  console.log("Defend");
};
```

### 함수 선언식 구조

```
function
  ↓
키워드: 함수 생성을 명령

attack
  ↓
함수 이름: 식별자로 등록

()
  ↓
매개변수 공간: 입력값을 받는 곳

{}
  ↓
함수 바디: 실행할 코드
```

### 호이스팅 차이

```javascript
// 함수 선언식 - 호이스팅됨
console.log(add(5, 3));  // 8 (정상 작동)

function add(a, b) {
  return a + b;
}

// 함수 표현식 - 호이스팅 안 됨
console.log(multiply(5, 3));  // TypeError: multiply is not a function

const multiply = function(a, b) {
  return a * b;
};
```

### 이유: 함수 선언식은 선언 단계에서 전체 함수가 메모리에 올라감

```javascript
// 실제 실행 순서:
// 1단계: 함수 선언식은 스코프 최상단으로 이동
function add(a, b) {
  return a + b;
}

// 2단계: 코드 실행
console.log(add(5, 3));
```

---

## 4. 화살표 함수 분석 - this 바인딩

### 코드

```javascript
// 기존 함수 표현식
const add1 = function(a, b) {
  return a + b;
};

// 화살표 함수
const add2 = (a, b) => {
  return a + b;
};

// 간단한 표현
const add3 = (a, b) => a + b;
```

### this 바인딩의 차이

```javascript
const user = {
  name: "John",
  
  // 일반 함수 - this는 user 객체를 가리킴
  greet: function() {
    console.log(this.name);  // "John"
  },
  
  // 화살표 함수 - this는 외부 스코프를 가리킴
  greetArrow: () => {
    console.log(this.name);  // undefined (window 또는 global)
  }
};

user.greet();       // "John"
user.greetArrow();  // undefined
```

### this 바인딩 상황별 분석

```javascript
// 상황 1: 일반 함수 - 메서드 호출
const obj1 = {
  value: 10,
  getValue: function() {
    return this.value;
  }
};

console.log(obj1.getValue());  // 10 (this = obj1)

// 상황 2: 일반 함수 - 단독 호출
const getValue = obj1.getValue;
console.log(getValue());       // undefined (this = window/global)

// 상황 3: 콜백으로 전달
setTimeout(obj1.getValue, 100);  // undefined (this 손실)

// 상황 4: bind/call/apply로 명시적 바인딩
console.log(getValue.call(obj1));  // 10 (this = obj1로 강제)

// 상황 5: 화살표 함수는 this를 고정
const obj2 = {
  value: 20,
  getValue: () => {
    return this.value;
  }
};

const getArrow = obj2.getValue;
console.log(getArrow());  // 같은 값 (외부 스코프의 this 유지)
```

---

## 5. 객체 분석 - 객체 리터럴 vs 프로토타입

### 코드

```javascript
const player = {
  name: "Hero",
  hp: 100,
  attack: function() {
    return this.hp * 2;
  }
};
```

### 객체 구조

```
player (객체)
  ├─ name: "Hero" (문자열 프로퍼티)
  ├─ hp: 100 (숫자 프로퍼티)
  ├─ attack: function() {} (메서드)
  └─ [[Prototype]]: Object.prototype (숨겨진 프로토타입)
```

### 프로토타입 체인(Prototype Chain)

```javascript
const animal = {
  move: function() {
    return "moving";
  }
};

const dog = Object.create(animal);
dog.name = "Buddy";
dog.bark = function() {
  return "woof";
};

// 프로토타입 체인 탐색
console.log(dog.name);    // "Buddy" (dog의 프로퍼티)
console.log(dog.move());  // "moving" (animal 프로토타입에서 찾음)
console.log(dog.bark());  // "woof" (dog의 메서드)

// 체인 구조:
// dog → animal → Object.prototype → null
```

### 메모리에서의 객체

```javascript
const obj = { value: 10 };

// 메모리 구조:
// 변수: obj
// 참조: 0x2000 → { value: 10, [[Prototype]]: {...} }
//      0x2000번지 내용:
//        - value: 10
//        - [[Prototype]]: 0x3000 (Object.prototype 참조)
```

---

## 6. 배열 분석 - 배열 vs 객체

### 코드

```javascript
const items = [
  "Sword",
  "Shield",
  "Potion"
];

console.log(items[0]);        // "Sword"
console.log(items.length);    // 3
console.log(items instanceof Array);  // true
```

### 배열 구조

```
배열 인덱스:
  0 → "Sword"
  1 → "Shield"
  2 → "Potion"
  length → 3 (자동으로 관리)

배열은 실제로 특수한 객체:
  {
    "0": "Sword",
    "1": "Shield",
    "2": "Potion",
    length: 3,
    [[Prototype]]: Array.prototype
  }
```

### 0부터 시작하는 이유

```javascript
// 메모리 관점:
// 배열 시작 주소: 0x1000
// items[0] = 0x1000 + (0 * 데이터크기) = 0x1000
// items[1] = 0x1000 + (1 * 데이터크기) = 0x1008
// items[2] = 0x1000 + (2 * 데이터크기) = 0x1010

// 0부터 시작하면 계산이 더 효율적
```

### 배열 메서드와 원본 수정 여부

```javascript
const arr = [3, 1, 4, 1, 5];

// 원본을 수정하는 메서드 (Mutating)
arr.sort();                    // [1, 1, 3, 4, 5]
arr.reverse();                 // [5, 4, 3, 1, 1]
arr.splice(0, 1);              // arr: [4, 3, 1, 1]

// 원본을 유지하고 새 배열 반환 (Non-mutating)
const sorted = [...arr].sort();
const filtered = arr.filter(x => x > 2);
const mapped = arr.map(x => x * 2);
```

---

## 7. 조건문 분석 - 조건식 평가

### 코드

```javascript
if (hp <= 0) {
  console.log("Die");
}
```

### 실행 과정

```
1단계: 조건식 평가
  hp <= 0
  ↓
  hp의 현재 값 확인

2단계: 불린값 반환
  true 또는 false 결정
  ↓

3단계: 분기 처리
  true → 블록 실행
  false → 블록 스킵
```

### Truthy vs Falsy

```javascript
// Falsy 값들 (조건식에서 false로 평가됨)
if (false) {}           // false
if (0) {}               // 0
if ("") {}              // 빈 문자열
if (null) {}            // null
if (undefined) {}       // undefined
if (NaN) {}             // NaN

// Truthy 값들 (조건식에서 true로 평가됨)
if (true) {}            // true
if (1) {}               // 0이 아닌 숫자
if ("hello") {}         // 비어있지 않은 문자열
if ({}) {}              // 객체
if ([]) {}              // 배열
if (function(){}) {}    // 함수
```

### 논리 연산자 활용

```javascript
// AND (&&) - 짧은 회로 평가
const user = { name: "John", admin: false };
user.admin && console.log("Admin");  // 실행 안 됨

// OR (||) - 짧은 회로 평가
const name = inputName || "Guest";   // inputName이 falsy면 "Guest"

// NOT (!)
const isActive = !user.disabled;
```

---

## 8. 반복문 분석 - for 루프의 동작

### 코드

```javascript
for (let i = 0; i < 5; i++) {
  console.log(i);
}
```

### 단계별 동작

```
초기화 단계:
  let i = 0
  메모리에 i 할당, 값 0으로 초기화
  ↓

첫 번째 반복:
  [조건 확인] i < 5 → true
  [본체 실행] console.log(0) → 0 출력
  [증감식]    i++ (i = 1)
  ↓

두 번째 반복:
  [조건 확인] i < 5 → true
  [본체 실행] console.log(1) → 1 출력
  [증감식]    i++ (i = 2)
  ↓
  
... 반복 ...

다섯 번째 반복:
  [조건 확인] i < 5 → true
  [본체 실행] console.log(4) → 4 출력
  [증감식]    i++ (i = 5)
  ↓

루프 종료:
  [조건 확인] i < 5 → false
  루프 탈출
```

### 다양한 반복 방식

```javascript
// for - 전통적, 모든 단계 명시적
for (let i = 0; i < arr.length; i++) {
  console.log(arr[i]);
}

// for...of - 값 기반 순회
for (const value of arr) {
  console.log(value);
}

// for...in - 키 기반 순회
for (const key in obj) {
  console.log(key, obj[key]);
}

// forEach - 콜백 함수 활용
arr.forEach((value, index) => {
  console.log(index, value);
});

// while - 조건 기반
let i = 0;
while (i < arr.length) {
  console.log(arr[i]);
  i++;
}
```

### 반복문 제어

```javascript
const items = [1, 2, 3, 4, 5];

// break - 루프 종료
for (const item of items) {
  if (item === 3) break;  // 3에서 루프 탈출
  console.log(item);      // 1, 2 출력
}

// continue - 다음 반복으로
for (const item of items) {
  if (item === 3) continue;  // 3 건너뜀
  console.log(item);         // 1, 2, 4, 5 출력
}
```

---

## 9. 이벤트 분석 - 이벤트 루프

### 코드

```javascript
button.onclick = function() {
  alert("Click");
};
```

### 이벤트 처리 흐름

```
사용자 행동
  ↓
[이벤트 발생] "click"
  ↓
[이벤트 큐에 등록]
  ↓
[이벤트 루프 확인]
  ↓
[콜스택이 비어있나?]
  yes → 콜스택으로 이동
  no → 대기
  ↓
[함수 실행]
  onclick 함수 호출
  alert("Click") 실행
  ↓
[결과 표시]
```

### 이벤트 종류

```javascript
// 마우스 이벤트
element.onclick = function() {};
element.addEventListener("mouseover", function() {});
element.addEventListener("mouseout", function() {});

// 키보드 이벤트
document.addEventListener("keydown", function(event) {
  console.log(event.key);
});

// 입력 이벤트
input.addEventListener("change", function() {});
input.addEventListener("input", function() {});

// 문서 이벤트
window.addEventListener("load", function() {});
document.addEventListener("DOMContentLoaded", function() {});
```

### 이벤트 위임(Event Delegation)

```javascript
// 비효율적: 각 항목마다 리스너 추가
items.forEach(item => {
  item.addEventListener("click", function() {
    console.log(this.textContent);
  });
});

// 효율적: 부모에서 한 번에 처리
list.addEventListener("click", function(event) {
  if (event.target.tagName === "LI") {
    console.log(event.target.textContent);
  }
});
```

---

## 10. 비동기 처리 분석 - 콜스택과 이벤트 루프

### 코드

```javascript
console.log("A");

setTimeout(() => {
  console.log("B");
}, 1000);

console.log("C");

// 결과:
// A
// C
// B (1초 후)
```

### 실행 순서 분석 - 이벤트 루프

```
[콜 스택]        [웹 API]          [콜백 큐]
─────────────────────────────────────────

console.log("A")
   ↓ 실행
콜스택에서 제거
   ↓

setTimeout 호출
   ↓ 웹 API로 전달
타이머 시작 (1000ms)
   ↓

console.log("C")
   ↓ 실행
콜스택에서 제거
   ↓

메인 코드 끝
콜스택이 비어짐
   ↓

1000ms 후 타이머 완료
콜백이 콜백 큐로 이동
   ↓

[이벤트 루프 확인]
콜스택이 비어있나? YES
콜백 큐에 작업이 있나? YES
   ↓

콜백을 콜스택으로 이동
console.log("B") 실행
   ↓
결과: A, C, B
```

### Promise와 비동기

```javascript
console.log("1");

// Promise - 마이크로태스크 큐 사용
Promise.resolve()
  .then(() => console.log("2"))
  .then(() => console.log("3"));

// setTimeout - 매크로태스크 큐 사용
setTimeout(() => {
  console.log("4");
}, 0);

console.log("5");

// 결과:
// 1
// 5
// 2
// 3
// 4

// 이유: 마이크로태스크가 매크로태스크보다 우선순위가 높음
```

### async/await 분석

```javascript
async function fetchData() {
  console.log("시작");
  
  // await에서 일시 정지
  const data = await fetch("/api/data");
  
  // 응답 후 재개
  console.log("완료");
  return data;
}

// async 함수는 항상 Promise를 반환
const promise = fetchData();

// 흐름:
// 1. fetchData() 호출
// 2. "시작" 출력
// 3. fetch() 호출 및 await에서 대기
// 4. 응답 대기 (논블로킹)
// 5. 응답 받으면 재개
// 6. "완료" 출력
```

---

## 11. 클로저(Closure) 분석 - 함수와 렉시컬 환경

### 개념

```javascript
function outer(x) {
  // outer의 렉시컬 환경: x
  
  function inner() {
    // inner에서 outer의 변수 x에 접근
    return x + 1;
  }
  
  return inner;
}

const result = outer(5);
console.log(result());  // 6

// 클로저: inner 함수가 outer의 렉시컬 환경을 "기억"
```

### 메모리 관점

```javascript
function makeCounter() {
  let count = 0;  // 외부 함수의 변수
  
  return function() {
    count++;
    return count;
  };
}

const counter = makeCounter();

console.log(counter());  // 1
console.log(counter());  // 2
console.log(counter());  // 3

// 메모리:
// - count는 클로저 때문에 메모리에 유지됨
// - 함수가 존재하는 한 count는 살아있음
```

### 클로저의 실제 활용

```javascript
// 1. 데이터 캡슐화 (private 변수)
function createBankAccount(initialBalance) {
  let balance = initialBalance;  // private
  
  return {
    deposit: function(amount) {
      balance += amount;
      return balance;
    },
    withdraw: function(amount) {
      if (balance >= amount) {
        balance -= amount;
        return balance;
      }
      return "잔액 부족";
    },
    getBalance: function() {
      return balance;
    }
  };
}

const account = createBankAccount(1000);
console.log(account.deposit(500));   // 1500
console.log(account.withdraw(200));  // 1300
console.log(account.balance);        // undefined (접근 불가)

// 2. 함수 팩토리
function multiplier(factor) {
  return function(number) {
    return number * factor;
  };
}

const double = multiplier(2);
const triple = multiplier(3);

console.log(double(5));  // 10
console.log(triple(5));  // 15
```

---

## 12. this 바인딩 완벽 분석

### 4가지 this 바인딩 규칙

```javascript
// 1. 메서드 호출: this는 호출한 객체
const obj = {
  name: "Object",
  getName: function() {
    return this.name;
  }
};

console.log(obj.getName());  // "Object"

// 2. 함수 호출: this는 undefined (strict mode) 또는 window/global
function getName() {
  return this.name;
}

console.log(getName());  // undefined 또는 undefined (strict mode)

// 3. 생성자 호출: this는 새로 생성된 객체
function Player(name) {
  this.name = name;
}

const player = new Player("Hero");
console.log(player.name);  // "Hero"

// 4. call/apply/bind: this를 명시적으로 지정
const user1 = { name: "John" };
const user2 = { name: "Jane" };

function introduce() {
  return this.name + " says hello";
}

console.log(introduce.call(user1));      // "John says hello"
console.log(introduce.apply(user2));     // "Jane says hello"
const boundFunc = introduce.bind(user1);
console.log(boundFunc());                // "John says hello"
```

### this 바인딩 우선순위

```
우선순위 (높음 → 낮음):

1순위: new 바인딩 (new 연산자)
2순위: 명시적 바인딩 (call, apply, bind)
3순위: 암시적 바인딩 (메서드 호출)
4순위: 기본 바인딩 (함수 호출)

예시:
function greet() {
  console.log(this.name);
}

const obj = { name: "Object" };

// 1순위: new
new greet();  // this = 새 객체

// 2순위: bind
greet.bind(obj)();  // this = obj

// 3순위: 메서드
obj.greet = greet;
obj.greet();  // this = obj

// 4순위: 단순 호출
greet();  // this = undefined (strict mode)
```

---

## 결론

JavaScript를 깊이 있게 이해하려면:

1. **메모리 구조**: 변수와 값이 어떻게 저장되는가
2. **스코프와 클로저**: 변수의 생명주기와 접근 범위
3. **호이스팅**: 코드 실행 전 어떤 일이 일어나는가
4. **this 바인딩**: 문맥에 따른 this의 변화
5. **비동기 처리**: 이벤트 루프와 콜 스택
6. **프로토타입**: 객체 간의 상속 구조

이러한 개념들을 이해하면 JavaScript의 "왜?"를 답할 수 있게 된다.
