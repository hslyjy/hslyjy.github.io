---
layout: post
title: 코어 자바스크립트 - 04. 콜백 함수 (1)
description: 콜백 지옥을 벗어나기 위해선 어떻게 해야하는지, 콜백 함수에 대해 깊이 알아본다.
tags: 
    - javascript
    - frontend
    - 코어 자바스크립트
author: sol
image: /assets/img/corejavascript.PNG
optimized_image: /assets/img/opt_corejavascript.png
category: code
---


# 04. 콜백 함수

## 4-1. 콜백 함수란?

다른 코드(함수 또는 메서드)에게 인자로 넘겨줌으로써 그 제어권도 함께 위임한 함수

## 4-2. 제어권

### 4-2-1. this

이전 포스팅에서 "콜백 함수도 함수이기 때문에 기본적으로는 this가 전역객체를 참조하지만, 제어권을 넘겨받을 코드에서 콜백 함수에 별도로 this가 될 대상을 지정한 경우에는 그 대상을 참조하게 된다"라는 것을 얘기했다.

별도의 this를 지정하는 방식 및 제어권에 대한 이해를 높이기 위해 map 메서드를 직접 구현한 예제를 살펴보자.

예제 4-1
```javascript
Array.prototype.map = function (callback, thisArgs) {
        var mappedArr = [];
        for (var i = 0; i < this.length; i ++) {
                var mappedValue = callback.call(thisArgs || window, this[i], i, this);
                mappedArr[i] = mappedValue;
        }
        return mappedArr
};
```

핵심은 call/apply 메서드에 있다. 예제 4-1에서 call 메서드를 활용하기 때문에 this에 다른 값이 담기는 이유를 정확히 알 수 있다. 바로 제어권을 넘겨받을 코드에서 call/apply 메서드의 첫 번째 인자에 콜백 함수 내부에서의 this가 될 대상을 명시적으로 바인딩하기 때문이다.

예제 4-2
```javascript
setTimeout(function () {console.log(this);}, 300);                          // (1) 

[1,2,3,4,5].forEach(function (x) {
        console.log(this);                                                  // (2)
});

document.body.innerHTML += '<button id="a">클릭</button>';
document.body.querySelector('#a').addEventListener('click', function(e){    
        console.log(this,e);                                                // (3)
});
```

위의 예제 4-2의 (1), (2), (3)의 값을 이제 정확히 예측할 수 있을 것이다.

(1)은 call 메서드의 첫 번쨰 인자에 전역객체를 넘기기 때문에 콜백 함수 내부에서의 this가 전역객체를 가리킨다.
(2)의 forEach는 '별도의 인자로 this를 받는 경우'에 해당하지만 별도의 인자로 this를 넘겨주지 않았기 때문에 전역객체를 가리킨다.
(3)의 addEventListener은 내부에서 콜백 함수를 호출할 때 call 메서드의 첫 번째 인ㄴ자에 addEventListener 메서드의 this를 그대로 넘기도록 정의되어 있기 때문에 콜백 함수 내부에서의 this가 addEventListener를 호출한 주체인 HTML 엘리먼트를 가리키게 된다.

## 4-3. 콜백 함수는 함수다

콜백 함수는 함수다. 당연한 소리라 생각될 수 있지만 이 의미를 곰곰이 생각해 볼 필요가 있다.

예제 4-3
```javascript
var obj = {
        vals: [1,2,3],
        logValues: function(v,i) {
                console.log(this, v, i);
        }
};
obj.logValues(1,2);                 // (1)
[4,5,6].forEach(obj.logValues);     // (2)
```

(1)의 결과는 obj를 가리키고, (2)의 결과는 전역객체를 가리킨다. logValues는 메서드이지만 (2)에서 obj를 this로 하는 메서드를 그대로 전달한 것이 아니라 함수만 전달한 것이다.

결국 어떤 함수의 인자에 객체의 메서드를 전달하더라도 이는 결국 메서드가 아닌 함수일 뿐이다. 이 차이를 정확히 이해하는 것이 중요하다.

## 4-4.  콜백 함수 내부의 this에 다른 값 바인딩하기

객체의 메서드를 콜백 함수로 전달하면 해당 객체를 this로 바라볼 수 없다. 그럼에도 콜백 함수 내부에서 this가 객체를 바라보게 하고싶다면 어떻게 해야할까?

예제 4-4
```javascript
var obj1 = {
    name: 'obj1',
    func: function () {
        var self = this;
        return function() {
            console.log(self.name);
        };
    }
};

var callback = obj1.func();
setTimeout(callback, 1000);
```

예제 4-4와 같이 전통적으로는 this를 다른 변수에 담아 콜백 함수로 활용할 함수에서는 this 대신 그 변수를 사용하게 하고 이를 클로저로 만드는 방식이 많이 쓰였다.

이 방식은 실제로 this를 사용하지도 않을뿐더러 번거롭기 그지 없다. 다음 예제를 살펴보자

예제 4-5
```javascript
var obj1 = {
        name: 'obj1',
        func: function() {
                console.log(obj1.name);
        }
    };
    
    
setTimeout(obj1.func, 1000);
```

this를 사용하지 않았을 때의 결과이다. 훨씬 간결하고 직관적이지만 this를 이용해 다양한 상황에서 재활용할 수 없게 되어버렸다.

예제 4-6
```javascript
var obj1 = {
        name: 'obj1',
        func: function () {
            var self = this;
            return function() {
                console.log(self.name);
            };
        }
};
    
var callback = obj1.func();
setTimeout(callback, 1000);

var obj2 = {
        name: 'obj2',
        func: obj1.func
};
var callback2 = obj2.func();
setTimeout(callback2, 1500);

var obj3 = { name: 'obj3'};
var callback3 = obj1.func.call(obj3);
setTimeout(callback3, 2000);
```

예제 4-6은 예제 4-4의 ob1의 func을 재활용하는 방법이다. 위처럼 예제 4-4가 번거롭긴 하지만 this를 우회적으로나마 활용함으로써 다양한 상황에서 원하는 객체를 바라ㅏ보는 콜백 함수를 만들 수 있다.

다행히 이제는 전통적인 방식의 아쉬움을 보완하는 훌륭한 방법이 있다. 바로 ES5에서 등장한 bind 메서드를 이용한 방법이다.

간단히 예제 코드만 소개하고 넘어가겠다.

예제 4-7
```javascript
var obj1 = {
        name: 'obj1',
        func: function () {
            console.log(this.name);
        }
};
setTimeout(obj1.func.bind(obj1), 1000);

var obj2 = {name: 'obj2'};
setTimeout(obj1.func.bind(obj2), 1500);
```

이번 포스팅을 정리하자면,
> - 콜백 함수는 다른 코드에 인자로 넘겨줌으로써 그 제어권도 함께 위임한 함수이다.
> - 제어권을 넘겨받은 코드는 다음과 같은 제어권을 가진다.
>> 1) 콜백 함수를 호출하는 시점을 스스로 판단해서 실행한다.<br>
>> 2) 콜백 함수의 this가 무엇을 바라보도록 할지가 정해져 있는 경우도 있다. 정하지 않는 경우 전역객체를 바라본다. 사용자 임의로 this를 바꾸고 싶은 경우, bind 메서드를 활용한다.
> - 어떤 함수에 인자로 메서드를 전달하더라도 이는 결국 함수로서 실행된다.

다음 포스팅에서는 콜백 지옥과 비동기 처리에 대해 알아보겠다.


 [코어 자바스크립트 (정재남 저)](http://www.yes24.com/Product/Goods/78586788) 도서를 참고하였습니다.
<br><br>  
