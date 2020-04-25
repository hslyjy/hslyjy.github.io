---
layout: post
title: 코어 자바스크립트 - 02.실행 컨텍스트
tags: [javascript, frontend, 코어 자바스크립트]
author: sol
---


# 02. 실행 컨텍스트

### 2-1. 실행 컨텍스트

실행 컨텍스트는 실행할 코드에 제공할 환경 정보들을 모아놓은 객체로, 동일한 환경에 있는 코드들을 실행할 때 필요한 환경 정보들을 모아 컨텍스트를 구성하고 
이를 콜스택에 쌓아 올렸다가 가장 위에 쌓여있는 컨텍스트와 관련 있는 코드들을 실행하는 식으로 전체 코드의 환경과 순서를 보장한다. 
실행 컨텍스트를 구성하는 방법 전역공간, eval(), 함수 등이 있다. 

어떤 실행 컨텍스트가 활성화될 때 자바스크립트 엔진은 해당 컨텍스트에 관련된 코드들을 실행하는 데 필요한 환경 정보들을 수집해서 실행 컨텍스트 객체에 저장한다. 이 객체는 자바스크립트 엔진이 활용할 목적으로 생성할 뿐 개발자가 코드를 통해 확인할 수는 없다. 실행 컨텍스트는 다음과 같이 구성된다.

> - VariableEnvironment: 현재 컨텍스트 내의 식별자들에 대한 정보 + 외부 환경 정보. 선언 시점의 LexicalEnvironment의 스냅샷으로, 변경 사항 반영 안됨.
> - LexicalEnvironment: 처음에는 VariableEnvironment와 같지만 변경 사항이 실시간으로 반영됨.
> - ThisBinding: this 식별자가 바라봐야 할 대상 객체.

### 2-2. VariableEnvironment
VariableEnvironment에 담기는 내용은 LexicalEnvironment와 같지만 최초 실행시의 스냅샷을 유지한다. 실행 컨텍스트를 생성할 때 VariableEnvironment에 정보를 먼저 담고, 그대로 복사하여 LexicalEnvironment를 만들고, 이후에는 LexicalEnvironment를 주로 활용한다.
VariableEnvironment와 LexicalEnvironment 모두 내부에 environmentRecord와 outerEnvironmentReference로 구성되어 있다.

### 2-3. LexicalEnvrionment
#### 2-3-1. environmentRecord와 호이스팅


예제 2-1.
{% highlight js %}
function a(x) {
    console.log(x); // (1)
    var x;
    console.log(x); // (2)
    var x = 2;
    console.log(x); // (3)
}
a(1);
{% endhighlight %}

예제 2-1의 결과를 예상해보자. 회사에서 해당 내용을 스터디하면서 동료들에게 위 코드의 결과를 예상해보라고 했을 때 대부분 정확한 결과를 예상하지 못했다. 보통 (1) 1, (2) undefined, (3) 2로 예상을 많이 한다. 실행 결과는 (1) 1, (2) 1, (3) 2 로 나온다.

왜 이런 결과가 나왔을까? 호이스팅 개념을 정확히 이해한다면 앞으로는 헷갈리지 않을 것이다.

LexicalEnvrionment의 environmentRecord에는 현재 컨텍스트와 관련된 코드의 식별자 정보들이 저장된다. 함수 자체, var로 선언된 변수의 식별자 등을 컨텍스트 내부 전체를 처음부터 끝까지 쭉 훑어나가며 순서대로 수집한다. 변수 정보를 수집하는 과정이 끝나도 아직 실행 컨텍스트가 관여할 코드들은 실행되기 전의 상태이다. 즉, 코드가 실행되기 전에 자바스크립트 엔진은 이미 해당 환경에 속한 코드의 변수명들을 모두 알고 있다.

쉽게 말해, **식별자들을 모두 최상단으로 끌어올린 다음 코드를 실행한다** 라고 이해해도 무방하다. 이것을 ***호이스팅***이라고 한다.

그렇다면 위 예제 2-1을 호이스팅 개념을 적용해 해석해보자.

아래 예제는 예제 2-1 코드를 호이스팅 개념을 설명하기 위해 바꾼 것이다. 실제로 코드가 변하진 않는다는 것을 명심하자.

예제 2-2
{% highlight js %}
function a() {
    var x;
    var x;
    var x;

    x = 1;
    console.log(x); // (1)
    console.log(x); // (2) 
    x = 2;
    console.log(x); // (3)
}
a(1);
{% endhighlight %}

위와 같이 선언된 변수들을 먼저 수집한 후 순서대로 코드를 실행하기 때문에 (1) 1, (2) 1, (3) 2 와 같은 결과가 나온 것이다.


#### 2-3-1. 함수 선언문과 함수 표현식 
함수 호이스팅의 경우에는 더 세심한 이해를 필요로 한다.
먼저 함수 선언문과 함수 표현식의 차이를 알아야한다.

예제 2-3
{% highlight js %}
function a(){}          // 함수 선언문. 함수명 a가 곧 변수명
a(); // 실행 OK

var b = function (){}   // (익명) 함수 표현식. 변수명 b가 곧 함수명
b(); // 실행 OK

var c = function d(){}  // 기명 함수 표현식. 변수명은 c, 함수명은 d
c(); 
{% endhighlight %}

함수 선언문의 경우에는 함수 자체를 모두 끌어올리지만, 함수 표현식의 경우에는 변수를 선언했기 때문에 변수 선언부가 먼저 끌어올려지고 함수는 코드를 실행될 때 함수를 변수에 할당한다.

예제 2-4의 결과를 확인해보면 바로 이해할 수 있을 것이다.

예제 2-4
{% highlight js %}
function a() {
        console.log(b);
        var b = 'bbb';
        console.log(b);
        function b() {
        }
        console.log(b);
    }
a();

function c() {
        console.log(d);
        var d = 'ddd';
        console.log(d);
        var d = function () {
        }
        console.log(d);
    }
c();
{% endhighlight %}

결과는 예상대로 아래와 같다.<br>
[Function: b]<br>
bbb<br>
bbb<br>
undefined<br>
ddd<br>
[Function: d]<br>

#### 2-3-2. 스코프, 스코프 체인, outerEnvironmentReference

전역 공간을 제외하면 **오직 함수에 의해서만** 스코프가 생성된다. 이러한 식별자의 유효범위를 안에서부터 바깥으로 차례로 검색해나가는 것을 스코프 체인이라고 한다. 이를 가능케 하는 것이 LexicalEnvironment의 두변째 수집자료인 outerEnvironmentReference이다.

A 함수 내부에 B 함수를 선언하고 다시 B 함수 내부에 C 함수를 선언한 경우, 함수 C의 outerEnvironmentReference는 함수 B의 LexicalEnvironment를 참조하고, 함수 B의 LexicalEnvironment에 있는 outerEnvironmentReference는 다시 함수 B가 선언되던 때(A)의 LexicalEnvrionment를 참조한다. 각 outerEnvironmentReference는 오직 자신이 선언된 시점의 LexicalEnvironment만 참조하고 있으므로 가장 가까운 요소부터 차례대로만 접근할 수 있고 다른 순서로 접근하는 것은 불가능하다.

다시 말해, **무조건 스코프 체인 상에서 가장 먼저 발견된 식별자에만 접근 가능**하다. 

예제 2-5.
{% highlight js %}
var a = 1;
var outer = function() {
    var inner = function () {
        console.log(a); // (1)
        var a=3;
    };
    inner();
    console.log(a); // (2)
}
outer();
console.log(a); // (3)
{% endhighlight %}

위 예제의 결과를 예상해보자. 어떻게 예상했을지는 모르지만 정답은 (1) undefined, (2) 1, (3) 1 이다.

천천히 따라가보자.
> - 전역 컨텍스트가 활성화 되고 전역 컨텍스트의 environmentRecord에 {a, outer} 식별자를 지정한다.
> - 전역 스코프에 있는 변수 a에 1을 outer에 함수를 할당한다.
> - outer 함수를 호출하고 이에 따라 outer 실행 컨텍스트가 활성화 된다.
> - outer 실행 컨텍스트 environmentRecord에 {inner} 식별자를 저장하고 outerEnvironmentReference에는 outer 함수가 선언될 당시의 LexicalEnvrionment가 담긴다.
> - outer 스코프에 있는 변수 inner에 함수를 할당한다.
> - inner 함수를 호출하고 inner 실행 컨텍스트가 활성화 된다.
> - inner environmentRecord에 {a}를 저장하고 outerEnvironmentReference에는 inner 함수가 선언될 당시의 LexicalEnvironment가 담긴다.
> - 식별자 a에 접근하고자 한다. 현재 활성화 상태인 inner 컨텍스트의 environmentRecord에서 a를 검색하고 a를 발견했지만 여기에는 아직 할당된 값이 없어 undefined를 출력한다.
> - inner 스코프에 있는 변수 a에 3을 할당한다.
> - inner 함수 실행이 종료되고 inner 실행 컨텍스트가 콜 스택에서 제거되며 바로 아래의 outer 실행 컨텍스트가 다시 활성화 된다.
> - 식별자 a에 접근을 시도하고 environmentRecord에 a가 있는지 찾아보고 없으면 outerEnvironmentReference에 있는 environmentRecord로 넘어가는 식으로 계속해서 검색한다. 전역 LexicalEnvrionment에 a가 있으니 그 a에 저장된 값 1을 반환한다.
> - outer 함수 실행이 종료되고 outer 실행 컨텍스트가 콜 스택에서 제거되며 바로 아래의 전역 컨텍스트가 활성화 된다.
> - 식별자 a에 접근을 시도하고 전역 컨텍스트의 environmentRecord에서 a를 검색하고 바로 a를 찾을 수 있다. 이로써 모든 코드의 실행이 완료되고 전역 컨텍스트가 콜 스택에서 제거되고 종료한다.


여기까지 하면 실행 컨텍스트의 전반적인 설명이 끝났다. 중요한 부분만을 요약하다보니 다소 설명이 부족할 수도 있다. 더욱 디테일한 내용이 궁금하면 위에 소개한 책을 읽어보는 것을 추천한다. 아직 thisBinding이 남았는데, thisBinding에 대해서는 다음 포스팅에서 구체적으로 다룰 예정이다.

 [코어 자바스크립트 (정재남 저)](http://www.yes24.com/Product/Goods/78586788) 도서를 참고하였습니다.
<br><br>  


