---
layout: post
title: 코어 자바스크립트 - 03.this (2)
description: 앞에서 this 값이 어떻게 바인딩 되는지 살펴봤다. 이러한 규칙을 꺠고 this에 별도로 바인딩하는 방법도 있다.
tags: 
    - javascript
    - frontend
    - 코어 자바스크립트
author: sol
image: /assets/img/corejavascript.PNG
optimized_image: /assets/img/opt_corejavascript.png
category: code
---

앞에서 this 값이 어떻게 바인딩 되는지 살펴봤다. 이러한 규칙을 꺠고 this에 별도로 바인딩하는 방법도 있다.


# 03. this (2)

## 3-2. 명시적으로 this를 바인딩하는 방법

### 3-2-1. call 메서드
```javascript
Function.prototype.call(thisArg[, arg1[, arg2[, ...]]])
```
call 메서드는 메서드의 호출 주체인 함수를 즉시 실행하도록 한다.

메서드의 첫 번째 인자를 this로 바인딩하고, 이후의 인자들을 호출할 함수의 매개변수로 한다.

함수를 그냥 실행하면 this는 전역객체를 참조하지만 call 메서드를 이용하면 임의의 객체를 this로 지정할 수 있다.

예제 3-2-1
```javascript
var obj = {
    a: 1,
    method: function (x, y) {
        console.log(this.a, x, y);
    }
};

obj.method(2, 3);               // 1 2 3
obj.method.call({a:4}, 5, 6);   // 4 5 6
```


### 3-2-2 apply 메서드
```javascript
Function.prototype.call(thisArg[, argsArray])
```

apply 메서드는 call 메서드와 기능적으로 완전 동일하다.

call 메서드와 달리 두 번째 인자를 배열로 받아 그 배열의 요소들을 호출할 함수의 매개변수로 지정한다.

예제 3-2-2
```javascript
var func = function(a,b,c) {*
    console.log(this,a,b,c);
};
func.apply({x:2}, [4,5,6]);         // { x: 2 } 4 5 6

var obj = {
    a: 1,
    method: function (x,y) {
        console.log(this.a, x,y);
    }
};
obj.method.apply({a:1},[5,6]);      // 1 5 6
```

### 3-2-3. bind 메서드
```javascript
Function.prototype.bind(thisArg[, arg1[,arg2[,...]]])
```

bind 메서드는 ES5에서 추가된 기능이다. call과 비슷하지만 즉시 호출하지 않고 넘겨받은 this 및 인수들을 바탕으로 새로운 함수를 반환하기만 하는 메서드이다.

Reactjs를 다루는 개발자들은 자주 봤을 메서드이다. 실제로 리액트에서 중요하게 쓰이는 메서드로 class 안의 함수를 바인딩할 때 자주 사용된다.

이 메서드를 이용할 때 '굳이 왜 바인딩 해줄까'라고 의문을 품었지만 이번 공부를 통해 확실하게 알게 되었다.

몇가지 예제를 통해 bind 메서드에 자세히 알아보자.

예제 3-2-3
```javascript
var func = function(a,b,c,d) {
    console.log(this, a, b, c, d);
};
func(1,2,3,4);

var bindFunc1 = func.bind({x:1});
bindFunc1(5, 6, 7, 8);                      // Window{ ... } 1 2 3 4

var bindFunc2 = func.bind({x:1}, 4, 5);
bindFunc2(6, 7);                            // { x: 1 } 4 5 6 7
bindFunc2(8, 9);                            // { x: 1 } 4 5 8 9
```

위 예제에서 알 수 있듯, bindFunc1과 bindFunc2 변수에는 func에 this를 {x:1}로 지정한 새로운 함수가 담긴다.
그리고 bindFunc2를 보면 매개변수 또한 지정하여 쓸 수 있다는 것을 알 수 있다.

#### name 프로퍼티

bind 메서드는 독특한 특징이 있는데 name 프로퍼티에 'bound'라는 접두어가 붙는다는 점이다. 이 떄문에 call이나 apply 메서드보다 코드를 추적하기에 더 수월하다.

예제 3-2-4
```javascript
var func = function (a, b, c, d) {
    console.log(this, a, b, c, d);
};
var bindFunc = func.bind({x:1},4,5);

console.log(func.name);                 // func
console.log(bindFunc.name);             // bound func
```

#### 상위 컨텍스트의 this를 내부함수나 콜백 함수에 전달하기

이전 포스팅에서 this를 그대로 바라보게 하기 위한 방법을 우회법을 소개했는데, call, aply 또는 bind 메서드를 이용하면 더 깔끔하게 처리할 수 있다. 개인적으로 bind가 가장 깔끔해보인다.

예제 3-2-5
```javascript
var obj = {
    solfunc: function () {
        console.log(this);
        var jiminFunc = function() {
            console.log(this);
        };
        jiminFunc.call(this);
    }
};
obj.solfunc();
```
```javascript
var obj = {
    solfunc: function () {
        console.log(this);
        var jiminFunc = function() {
            console.log(this);
        }.bind(this);
        jiminFunc(this);
    }
};
obj.solfunc();
```

### 3-2-4. 화살표 함수의 예외사항

ES6에 새롭게 도입된 화살표 함수는 실행 컨텍스트 생성 시 this를 바인딩하는 과정이 사라졌다.

이 함수 내부에는 this가 아예 없으며, 접근하고자 하면 스코프체인상 가장 가까운 this에 접근하게 된다. this를 우회하거나 call/apply/bind 메서드를 적용할 필요가 없어 더욱 간결하고 편리하다.

그러나 이 함수는 ES6에만 적용된다는 사실을 명심하자.

예제 3-2-6
```javascript
var obj = {
    solfunc: function() {
        console.log(this);
        var jiminFunc = () => {
            console.log(this);
        };
        innerFunc();
    }
};
obj.outer();
```

### 3-2-5. 별도의 인자로 this를 받는 경우(콜백 함수 내에서의 this)

콜백 함수를 인자로 받는 메서드 중 일부는 추가로 this를 지정할 객체(thisArgs)를 인자로 지정할 수 있는 경우가 있다.

이런 형태는 여러 내부 요소에 대해 같은 동작을 반복수행하는 **배열 메서드**가 많이 가지고 있으며, ES6에서 새로 등장한 메서드들(Set, Map)에도 일부 존재한다.

thisArgs를 인자로 받는 메서드 몇가지를 나열하고 넘어가겠다.
```javascript
Array.prototype.forEach(callback[, thisArg])
Array.prototype.forEach(callback[, thisArg])
Array.prototype.forEach(callback[, thisArg])
Array.prototype.forEach(callback[, thisArg])
Array.prototype.forEach(callback[, thisArg])
Array.prototype.forEach(callback[, thisArg])
```

정리하자면,
> - call, apply 메서드는 this를 명시적으로 지정하면서 함수나 메서드를 호출한다.
> - bind 메서드는 this 및 함수에 넘길 인수를 일부 지정하여 새로운 함수를 만든다.
> - 요소를 순회하며 콜백 함수를 반복 호출하는 내용의 일부 메서드는 별도의 인자로 this를 받기도 한다.



 [코어 자바스크립트 (정재남 저)](http://www.yes24.com/Product/Goods/78586788) 도서를 참고하였습니다.
<br><br>  
