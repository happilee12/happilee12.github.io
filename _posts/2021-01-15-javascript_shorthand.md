---
title: "[Javascript] 간결하고 읽기쉬운 자바스크립트 코드를 만드는 20가지 단축표기법(Shorthand Technique)✨"
date: 2021-01-15 00:00:00 -0400
categories: javascript translation
---


> 이 포스트는 다음 포스트를 한글로 번역하여 작성하였습니다.
원글 : [https://medium.com/javascript-in-plain-english/20-javascript-shorthand-techniques-that-will-save-your-time-f1671aab405f](https://medium.com/javascript-in-plain-english/20-javascript-shorthand-techniques-that-will-save-your-time-f1671aab405f)

### ⚡ 알게될 내용
아래 코드를 이해하고 적절히 활용할 수 있다.  
1. `isLoggedin && goToHomepage();`
2. `let total = +'453'`
3. `const floor = ~~6.8;`
4. `let obj = {x: 20, y: 'hello'};
const cloneObj = {...obj};`


--------------------------------------
<br>

![20javascript shorthand techniques](/assets/20javascript%20shorthand%20techniques.jpeg)


프로그래밍 언어에서의 shorthand technique를 활용하면 더 간결하고 최적화된 코드를 작성할 수 있으며 코드 양을 줄여줄일 수 있다. Javascript에서의 `단축표기법(shorthand technique)`을 알아보자.


<br>
### 1. 변수 선언
```javascript
//Longhand
let x;
let y = 20;

//Shorthand
let x, y = 20;
```
<br>
### 2. 여러 변수에 값 할당
array destructuring을 활용하면 여러 변수에 값을 쉽게 할당할 수 있다.

```javascript
//Longhand
let a, b, c;
a = 5;
b = 8;
c = 12;

//Shorthand
let [a, b, c] = [5, 8, 12];
```
<br>
### 3. 3항연산자
이렇게 3항 연산자를 활용하면 5줄을 아낄 수 있다.
```javascript
//Longhand
let marks = 26;
let result;
if(marks >= 30){
 result = 'Pass';
}else{
 result = 'Fail';
}
//Shorthand
let result = marks >= 30 ? 'Pass' : 'Fail';
```
<br>
### 4. 디폴트 값 할당
`OR(||)` 연산을 사용하면, 예상되는 값이 `falsy`일 때의 기본값을 할당 할 수 있다.
```javascript
//Longhand
let imagePath;
let path = getImagePath();
if(path !== null && path !== undefined && path !== '') {
  imagePath = path;
} else {
  imagePath = 'default.jpg';
}

//Shorthand
let imagePath = getImagePath() || 'default.jpg';
```
<br>
### 5. `And(&&)` 연산자 활용
어떤 값이 true일 때만 함수를 실행하고 싶다면, `&&`연산자를 활용할 수 있다.
```javascript
//Longhand
if (isLoggedin) {
 goToHomepage();
}

//Shorthand
isLoggedin && goToHomepage();
```
이런 방식은 **React**에서 특정 조건일 때 컴포넌트를 랜더링 하는 등에 활용하면 유용하다
```javascript
<div> { this.state.isLoading && <Loading /> } </div>
```
<br>
### 6. 두 변수의 값 swap
두 변수의 값을 사용할 때에도, array destructuring을 활용할 수 있다.
```javascript
let x = 'Hello', y = 55;
//Longhand
const temp = x;
x = y;
y = temp;

//Shorthand
[x, y] = [y, x];
```
<br>
### 7. 화살표함수(Arrow Function)
```JavaScript
//Longhand
function add(num1, num2) {
   return num1 + num2;
}

//Shorthand
const add = (num1, num2) => num1 + num2;
```
참고: [JavaScript Arrow function](https://jscurious.com/javascript-arrow-function/)

<br>
### 8. 템플릿 리터럴(Template literal)
ES6부터는 템플릿 리터럴을 활용해서 문자열을 손쉽게 연결할 수 있다.
``` JavaScript
//Longhand
console.log('You got a missed call from ' + number + ' at ' + time);

//Shorthand
console.log(`You got a missed call from ${number} at ${time}`);
```
<br>
### 9. 여러 줄에 걸친 문자열
여러줄에 걸친 문자열을 표현할 때 일반적으로는 `+` 연산자와 `\n`문자 대신 백틱(`) 을 사용하면 편하다.

```javascript
//Longhand
console.log('JavaScript, often abbreviated as JS, is a\n' +             'programming language that conforms to the \n' +
'ECMAScript specification. JavaScript is high-level,\n' +
'often just-in-time compiled, and multi-paradigm.' );
//Shorthand
console.log(`JavaScript, often abbreviated as JS, is a programming language that conforms to the ECMAScript specification. JavaScript is high-level, often just-in-time compiled, and multi-paradigm.`);
```
<br>
### 10. 여러 조건 확인
여러개 중 하나의 값과 매칭하는지 여부를 볼 때, 매칭하는 값들을 array에 넣고 `indexOf()`나 `includes()`를 사용할 수 있다.
```javascript
//Longhand
if (value === 1 || value === 'one' || value === 2 || value === 'two') {
     // Execute some code
}

// Shorthand 1
if ([1, 'one', 2, 'two'].indexOf(value) >= 0) {
    // Execute some code
}
// Shorthand 2
if ([1, 'one', 2, 'two'].includes(value)) {
    // Execute some code
}
```
<br>
### 11. 객체 프로퍼티 할당
변수명과 객체의 key 이름이 같다면, 변수명만 적어도 된다. 자바스크립트가 자동으로 키 이름을 변수명과 같게 설정하고, 값을 할당해준다.
```javascript
let firstname = 'Amitav';
let lastname = 'Mishra';
//Longhand
let obj = {firstname: firstname, lastname: lastname};

//Shorthand
let obj = {firstname, lastname};
```
<br>
### 12. String => Number
`parseInt`나 `parseFloat`등 문자열을 숫자로 변환하는 빌트인 메소드가 있긴 하다.
**`+`연산자를 문자열 앞에 붙여주는 방식으로도, 문자열을 숫자로 변환**할 수 있다.
```javascript
//Longhand
let total = parseInt('453');
let average = parseFloat('42.6');

//Shorthand
let total = +'453';
let average = +'42.6';
```
<br>
### 13. 문자열 여러번 반복
문자열을 여러번 반복할 때 `for`룹 대신 `repeat()` 메소드를 사용하면 한줄에 처리할 수 있다.

```javascript
//Longhand
let str = '';
for(let i = 0; i < 5; i ++) {
  str += 'Hello ';
}
console.log(str); // Hello Hello Hello Hello Hello

// Shorthand
'Hello '.repeat(5);
```

Tip. 누군가에게 'sorry'를 100번 보내면 사과하고 싶을 때, `repeat()`메소드를 활용해보라.
```javascript
'sorry\n'.repeat(100);
```
<br>
### 14. 거듭제곱 연산 (Exponent Power)
거듭제곱 연산에 `Math.pow()` 대신, `**`를 활용하면 더 짧게 쓸 수 있다.
```javascript
//Longhand
const power = Math.pow(4, 3); // 64

// Shorthand
const power = 4**3; // 64
```
<br>
### 15. `~~`를 활용한 반올림
두개의 `~(NOT)` 비트연산자를 `Math.floor()`메소드 대신에 사용할 수 있다. 단, 32 비트 정수에만 유효하다.  (2**31)-1 = 2147483647보다 큰 수에는 비트 연산자 대신 Math.floor() 를 사용해야 한다.
```javascript
//Longhand
const floor = Math.floor(6.8); // 6

// Shorthand
const floor = ~~6.8; // 6
```


<br>
### 16. 배열에서 최대, 최솟값 찾기
loop을 통해 배열 내 값을 하나하나 확인하며 최대/최솟값을 찾는 방법도 있고, `Array.reduce()`를 활용하는 방법도 있지만, `...(spread operator)`를 사용하면 한 줄에 처리할 수 있다.

```javascript
// Shorthand
const arr = [2, 8, 15, 4];
Math.max(...arr); // 15
Math.min(...arr); // 2
```
<br>
### 17. For loop
`for`룹 대신 `for ... of` 룹이나, `for ... in` 룹을 사용하여 배열을 순회할 수도 있다.
```javascript
let arr = [10, 20, 30, 40];
//Longhand
for (let i = 0; i < arr.length; i++) {
  console.log(arr[i]);
}
//Shorthand
//for of loop
for (const val of arr) {
  console.log(val);
}
//for in loop
for (const index in arr) {
  console.log(`index: ${index} and value: ${arr[index]}`);
}
```

`for ... in `룹을 사용하면 객체의 프로퍼티를 순회할 수도 있다.
```javascript
let obj = {x: 20, y: 50};
for (const key in obj) {
  console.log(obj[key]);
}
```
참고. [Different ways to iterate through objects and arrays in JavaScript](https://jscurious.com/different-ways-to-iterate-through-objects-and-arrays-in-javascript/)
<br>
### 18. 배열 합치기
```javascript
let arr1 = [20, 30];
//Longhand
let arr2 = arr1.concat([60, 80]);
// [20, 30, 60, 80]

//Shorthand
let arr2 = [...arr1, 60, 80];
// [20, 30, 60, 80]
```
<br>
### 19. 객체의 deep cloning (깊은 복사)
여러 계층짜리 객체를 깊은복사 할때에 각 프로퍼티를 돌며 각 프로퍼티가 객체를 포함하는지 확인하고, 객체를 포함한다면 재귀적 호출을 통해 현재 값을 전달하며 값을 복사할 수 있다.

만약 복사하려는 객체가 함수, undefined, NaN, Date 등을 값으로 갖지 않는다면, `JSON.parse()`와 `JSON.stringify()`를 활용해서 복사하는 방법도 있다.

또 한 층짜리 객체일 경우엔(nested object가 없을 경우), spread operator(`...`)를 통해 deep clone을 할 수도 있다.

```javascript
let obj = {x: 20, y: {z: 30}};

//Longhand
const makeDeepClone = (obj) => {
  let newObject = {};
  Object.keys(obj).map(key => {
    if(typeof obj[key] === 'object'){
      newObject[key] = makeDeepClone(obj[key]);
    } else {
      newObject[key] = obj[key];
    }
  });
 return newObject;
}
const cloneObj = makeDeepClone(obj);

//Shorthand
const cloneObj = JSON.parse(JSON.stringify(obj));
//Shorthand for single level object
let obj = {x: 20, y: 'hello'};
const cloneObj = {...obj};
```
참고 : [JSON.parse() and JSON.stringify()](https://jscurious.com/difference-between-json-parse-and-json-stringify/)

역자 추가) nested object를 spread operator로 copy 하는 경우, **가장 상위 계층의 값들만 deep copy 되고,** nested object는 shallow copy된다.

![nest object deepcopy](/assets/2021-01-15-vue-950c95b0.png)

최상위 계층에 있는 x는 copied에서 값을 바꿨을 때 obj의 값에는 영향을 주지 않았지만, nested object인 z값을 바꾸면 원본 Obj의 값도 바뀐다.

<br>
### 20. 문자열에서 문자 가져오기
```javascript
let str = 'jscurious.com';
//Longhand
str.charAt(2); // c

//Shorthand
str[2]; // c
```
