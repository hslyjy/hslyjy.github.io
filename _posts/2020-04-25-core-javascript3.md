---
layout: post
title: 코어 자바스크립트 - 03.this (1)
tags: [javascript, frontend, 코어 자바스크립트]
author: sol
---

자바스크립트로 코드를 짜다보면 많은 사람들이 this에 대해 혼란을 느낀다.<br> 
대부분의 객체지향 언어의 경우 this는 클래스로 생성한 인스턴스를 의미하지만 자바스크립트에서는 상황에 따라 this가 바라보는 대상이 달라진다.<br>
이번 포스팅은 javascript의 this에 대해서 다룬다.

# 03. this (1)

## 3-1. 상황에 따라 달라지는 this 

this는 기본적으로 실행 컨텍스트가 생성될 때 함께 결정되며 실행 컨텍스트는 함수를 호출할 때 생성되므로,<br> 
this는 **함수를 호출할 때 결정된다**.

### 3-1-1. 전역 공간에서의 this
전역 공간에서의 this는 전역 객체
> - 브라우저 환경 : window
> - Nodejs 환경 : global

### 3-1. 함수 vs 메서드

this를 설명하기 전에 먼저 함수와 메서드의 이해부터 해야한다.
> - 함수 : 그 자체로 독립적인 기능 수행
> - 메서드 : 미리 정의한 동작을 수행하는 코드 뭉치. **함수 앞에 .이 있으면 메서드임**

예제 3-1
```js
var func = function (x) {
    console.log(this, x);
};
func(1); // Window {...} 1

var obj = {
    method: func
};
obj.method(2); // { method: f } 2
```

### 3-1-2. 메서드 내부에서의 this
this에는 호출한 주체에 대한 정보가 담긴다. 호출 주체는 함수명 앞의 객체이다. 점 표기법의 경우 마지막 점 앞에 명시된 객체가 this가 된다.

#### 함수 내부에서의 this
실행 컨텍스트를 활성화할 당시에 this가 지정되지 않은 경우, this는 전역 객체를 바라본다. 따라서 함수에서 this는 전역 객체를 가리킨다.

#### 메서드의 내부함수에서의 this
개발자들이 혼란을 자주 느끼는 지점 중 하나...<br>
예제 3-2
```js
var obj1 = {
    outer: function() {
        console.log(this);          // (1)
        var innerFunc = function () {
            console.log(this);      // (2), (3)
        }
        innerFunc();

        var obj2 = {
            innerMethod: innerFunc
        };
        obj2.innerMethod();
    }
};

obj1.outer();
```

위 예제3-2의 (1), (2), (3)을 예측해보자.

정답은 (1): obj1, (2): 전역객체, (3): obj2이다. 이유는 단순하다.<br>
(1)의 경우 obj1.outer()가 실행되면서 실행 컨텍스트가 생성되고 호이스팅하고, 스코프 체인 정보를 수집하고, this를 바인딩한다. outer 앞에 점(.)이 있었으므로 메서드로 호출한 것이며 점 앞의 객체인 obj1이 바인딩된다.<br>

(2)의 경우에는 outer 블록 안에서 innerFunc()을 호출하였지만, 앞에 점이 없으므로 메서드가 아닌 함수로 호출된 것이다. 때문에 this가 지정되지 않았고 자동으로 전역객체가 바인딩된다.<br>

(3)의 경우에는 obj2 객체 안에 innerMethod 변수에 innerFunc을 할당했다. 그리고 obj2.innerMethod()를 호출할 때 실행 컨텍스트가 생성되고 호이스팅하고, 스코프 체인 정보를 수집하고, this를 바인딩한다. innerMethod() 앞에 점(.)이 있었으므로 메서드로 호출한 것이며 점 앞의 객체인 obj2가 바인딩된다.

여기까지 이해했다면 간단하게 이렇게 이해하는 사람도 있을 것이다. 메서드는 '객체의 프로퍼티에 할당된 함수'구나!<br>
하지만 반은 맞고 반은 틀렸다고 책에서는 얘기한다. 

어떤 함수를 객체의 프로퍼티에 할당한다고 해서 그 자체로서 무조건 메서드가 되는 것이 아니고 <br>
객체의 메서드로서 호출할 경우에만 메서드로 동작하는 것이다. 

예제 3-1에서 볼 수 있듯이 func는 함수로 동작하기도 하고 메서드로 동작하기도 한다.

#### 메서드 내부 함수에서의 this 우회 방법
그러면 이제 this의 인상이 달라졌다. 하지만 호출 대상이 없을 때 전역 객체를 바인딩하지 않고 호출 당시 주변 환경의 this를 상속받아 사용하고 싶어진다.
예제 3-3
```js
var obj1 = {
    outer: function() {
        console.log(this);          // (1)
        var innerFunc1 = function () {
            console.log(this);      // (2), (3)
        }
        innerFunc1();

        var self = this;
        var innerFunc2 = function () {
            console.log(self);
        }
        innerFunc2();
    }
};

obj1.outer();
```
ES5까지는 자체적으로 내부함수에 this를 상속할 방법이 없다. 위의 예제 3-3처럼 self라는 변수에 this를 할당하여 self를 출력하게 하면 self가 참조하고 있는 this를 출력할 수 있게 된다.<br>
이런 방식을 사용할 경우 _this, that 혹은 _ 등 다른 변수명을 쓰는데 self가 가장 많이 쓰인다고 한다.(나는 현업에서 _this나 _를 많이 봐왔다.)

#### this를 바인딩하지 않는 함수

ES6에서는 함수 내부에서 this가 전역객체를 바라보는 문제를 보완하려고 화살표 함수(arrow function)을 도입했다.<br>
화살표 함수는 실행 컨텍스트를 생성할 때 this 바인딩 과정 자체가 빠져 상위 스코프의 this를 그대로 활용 가능하다.
예제 3-4를 실행해보면 알 수 있다.

예제 3-4
```js
var obj = {
    outer: function() {
        console.log(this);
        var innerFunc = () => {
            console.log(this);
        };
        innerFunc();
    }
};
obj.outer();
```
### 3-1-3. 콜백 함수 호출 시 그 함수 내부에서의 this

다음 예제를 보면서 이해를 해보자.

예제 3-5
```js
setTimeout(function() {console.log(this);}, 300);               //(1)

[1,2,3,4,5].forEach(function(x) {                               //(2)
    console.log(this, x);
});

document.body.innerHTML += '<button id = "a">클릭</button>';
document.body.querySelector('#a')
        .addEventListener('click', function(e){
            console.log(this, e);                               //(3)
        });
```
(1)의 setTimeout 함수와 (2)의 forEach 메서드는 그 내부에서 콜백 함수를 호출할 때 대상이 될 this를 지정하지 않는다.<br>
(3)의 addEventListener 메서드는 콜백 함수를 호출할 때 자신의 this를 상속하도록 정의되어 있다.

결론적으로 **콜백 함수에서의 this는 어떻다라고 따로 정의할 수 없다**. 콜백 함수의 제어권을 가지는 함수가 콜백 함수에서의 this를 무엇이로 할지 결정하며, 특별히 정의하지 않은 경우, 전역객체를 바라본다.

### 3-1-4. 생성자 함수 내부에서의 this
이 또한 다음 예제와 함께 확인해보자.
예제 3-6
```js
var Cat = function(name, age) {
    this.bark = '야옹';
    this.name = name;
    this.age = age;
};
var roger = new Cat('roger', 6);
var mong = new Cat('mong', 7);

console.log(roger, mong);

/* 결과
Cat { bark: '야옹', name: 'roger', age: 6 } Cat { bark: '야옹', name: 'mong', age: 7 }
*/
```

6번째 생성자 함수 내부에서의 this는 roger 인스턴스를,<br>
7번째 생성자 함수 내부에서의 this는 mong 인스턴스를 가리키는 것을 알 수 있다. 

 [코어 자바스크립트 (정재남 저)](http://www.yes24.com/Product/Goods/78586788) 도서를 참고하였습니다.
<br><br>  
