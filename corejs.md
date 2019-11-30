# Core Javascript

## 1. datatype

`[bool, undefined, null, symbol, string, object, number]`

ref -> object  
primitive  -> others

```txt
Q. gc 실행 전 ref count 0 인 변수 공간들도 재사용되는가?
    A. 파악이 될 방안이 있다면 가능할 것 같다고 말씀하심
  
Q. primitive 데이터 중 중복되는 값이 있음을 (v8에서) 어떤 식으로 파악하고 있는가?
    A. 내부적 원리는 잘 모르겠다고 하심. (Hash가 아닐까 라는 개인적 추측)
```

## 2. exec context

"code snipets based on same condition, env"  
condition, env == context

es5 -> function, global (traditional scope by var)  
after es5 -> add { } (by let, const)

"함수를 실행할 때 필요한 환경정보"들을 담을 객체 == exec context (js 창시자가 만듬)

1. *VariableEnvironment* ( 순간의 값 )
    - environmentRecord (snapshot)
    - outerEnvironmentReference (snapshot)
2. *LexicalEnvironment* ( 값의 변경 O )
    - environmentRecord -> 현 문맥의 식별자 정보 (*hoisting (허구의 개념))
    - outerEnvironmentReference -> 현 문맥과 관련있는 **외부 식별자 정보
3. *ThisBinding*

```txt
* hoisitng
-> 실제로 동작원리는 아니고, 같은 효과를 보이는 현상을 설명하기 위해 만든 허구의 개념.
실행 컨텍스트의 맨위로 식별자 정보를 끌어올림. (함수는 함수 전체를 끌어올림)
식별자가 모두 끌어올려진 상태의 값들이 environmentRecord 내의 정보들과 같다.


** 외부 식별자 정보
현재 context 바로 직전의 LexicalEnvironment (VariableEnvironment) - callstack 상 인접
당연히 이것은 연쇄적이며 scope chain을 만들어내는 과정.

++ environmentRecord에 outer scope에 존재하는 변수와 같은 이름으로 선언할 시 outer scope에 해당 값에 접근할 수 없게 되는 현상을 shadowing 이라 한다.
```

## 3. this

this binding은 실행컨텍스트가 활성화 될 때 한다.  
-> 실행컨텍스트 활성화는 함수가 호출되는 순간에 한다.  
-> this는 함수 호출 시에 동적으로 바인딩된다.

> es5에서 함수(Function, not Method)로써 호출된 this는 언제나 전역객체를 가리키게 된다.  
(버그? js의 특성?)

```js
var a = 10;
var obj = {
    a: 20,
    b: function() {
        console.log(this.a) // 20
        function c() {
            console.log(this.a) // 10
        }
        c(); // (this가 전역으로 binding)
    }
}

// 이러한 문제를 해결하기 위한 방안이 필요했다.

var a = 10;
var obj = {
    a: 20,
    b: function() {
        var self = this;
        console.log(this.a) // 20
        function c() {
            console.log(self.a) // 20
        }
        c(); // (this가 전역으로 binding)
    }
}

// 위처럼 this를 다른 변수로 담아 outer scope를 통해 처리할 수 있도록 함.
```

es6에선 이런 문제를 해결하기 위해 arrow function이 탄생

### callback에서의 this

> setTimeout 내에서는 this를 별도로 처리하진 않아서, this는 window가 binding
> addEventListener는 this를 target으로 binding해준다.

이처럼  
_this를 지정하지 않으면 `window`,_  
_callback에서 지정해주면 `해당 this`,_  
_개발자가 callback에 미리 binding해서 넘기면 `해당 값`_  
을 따른다.

### 생성자 함수에서의 this

new 쓰고 말고 차이 (당연하게도) new 붙이면 해당 공간, 안붙이면 전역공간 binding

## 4. Callback Function

### 제어권

1. 실행 시점  
    ex) setInterval
2. 인자  
    ex) forEach
3. this

이처럼  
_다른함수 (A)의 인자로 콜백함수(B)를 전달하면 A가 B의 `제어권` 가짐_  
_개발자의 별도 지정이 없는 한 A에 `*미리 정해놓은 방식`에 따라 B를 호출_

```txt
*미리 정해놓은 방식이란  
어떤 시점에 콜백을 호출할 지,  
인자에 어떤 값을 지정할 지,  
this를 무엇을 바인딩할지  
```

callback을 bind 등을 통해 미리 정의해 전달하면 custom 가능

## 5. Closure

MDN: "함수가 선언될 당시의 lexical env와 함수의 결합이다."

## 6. Prototype

### 6-1. prototype and \_\_proto\_\_

1. prototype  

2. constructure  

3. \_\_proto\_\_  

> proto chain

### 6-2. 메서드 상속 및 동작원리

prototype은 instance와 전혀 다른 공간임을 언제나 기억

A.\_\_proto\_\_.something() 과 A.something()은 서로 다른 결과값을 낼 가능성이 다분함. (단, something은 prototype의 func)

```js
function Person(name, age) {
    this.name = name;
    this.age = age;
}
Person.prototype.age = 100;
Person.prototype.addAge() {
    return (this.age += 1);
}

var t = new Person("A", 1);
t.addAge(); // 2
t.__proto__.addAge(); // 101
```

## 7. Class

### 7-1. class

### 7-2. inheritance

## 100. ETC

전역공간에 변수를 선언하지 말라 -> 전역 공간에 버그스러운 현상이 많다. (변수 선언으로 하면 delete 안되고 window.~~ 로 하면 delete 가능 등)

Object에서는 Prototype이 아닌 method로만 사용하는 routine들이 유난히 많다. (keys, entries ... )  
why? Object는 proto chain 및 prototype 설계에 의해 객체를 this로 사용하기가 난감하기 때문
