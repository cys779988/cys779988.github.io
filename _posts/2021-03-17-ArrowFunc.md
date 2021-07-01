---
title:  "Arrow Function"
excerpt: Arrow Function
categories:
  - javascript
---


## 일반 함수와 화살표 함수 비교

```javascript
let sum = (a, b) => a + b;

let sum = function(a, b) {
  return a + b;
};

```  

## 화살표 함수의 this


```javascript
let group = {
  title: "1조",
  students: ["보라", "호진", "지민"],

  showList() {
    this.students.forEach(
      student => alert(this.title + ': ' + student)
    );
  }
};

group.showList();
```  

- 화살표 함수는 일반 함수와는 달리 ‘고유한’ this를 가지지 않음. 화살표 함수에서 this를 참조하면, 화살표 함수가 아닌 ‘평범한’ 외부 함수에서 this 값을 가져옴.
- 화살표 함수 본문에서 this에 접근하면 함수가 선언된 곳 상위의 context를 가리키므로 외부에서 값을 가져옴
- 이런 특징은 객체의   ```showList()```   안에서 동일객체의 프로퍼티(student)를 대상으로 순회를 하는 데 사용할 수 있음
- 예시의 foreach에서 화살표 함수를 사용했기 때문에 화살표 함수 본문에 있는   ```this.title```  은 화살표 함수 바깥에 있는 메서드인   ```showList()```  가 가리키는 대상과 동일해짐. 즉   ```this.title```  과   ```group.title```  은 같음


## 화살표함수 사용하면 안되는 경우

#### 메소드

  
```javascript
const person = {
    name : 'Lee',
    sayHi: () => console.log(`hi ${this.name}`)
};

person.sayHi();   // hi undefined




const person = {
    name : 'Lee',
    sayHi() { console.log(`hi ${this.name}`);
    }
}

person.sayHi();   // hi Lee
```  

- 화살표 함수 사용시 this메소드를 소유한 객체가 아닌, 상위 스코프의 컨텍스트인 전역객체 window를 가리킴


#### prototype

  
```javascript
const person = {
    name: 'Lee',
}

Object.prototype.sayHi = () => console.log(`hi $[this.name]`);

person.sayHi();   // hi undefined




const person = {
    name: 'Lee',
};

Object.prototype.sayHi = function() {
    console.log(`hi ${this.name});
}:

person.sayHi();   // hi Lee
```  

#### 생성자 함수

  
```javascript
const Foo = () => {};

console.log(Foo.hasOwnProperty('prototype'));   // false
// 화살표함수는 prototype 프로퍼티가 없음

const foo = new Foo();    // TypeError: Foo is not a constructor
```  


#### addEventListener 함수의 콜백함수

  
```javascript
var button = document.getElementById('myButton');  
button.addEventListener('click', () => {  
  console.log(this === window); // => true
  this.innerHTML = 'Clicked button';
});




var button = document.getElementById('myButton');  
button.addEventListener('click', function() {  
  console.log(this === button); // => true
  this.innerHTML = 'Clicked button';
});
```  

- addEventListener 함수의 콜백 함수를 화살표 함수로 정의하면 this가 상위 컨텍스트인 전역객체 window를 가리킴
