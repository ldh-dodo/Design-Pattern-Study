# 07. 자바스크립트 디자인 패턴

개발자는 어느 프로젝트에나 어울리는 최고의 패턴을 찾고자 하지만, ‘최고의 패턴’의 정답은 없다. 각 프로젝트에 요구되는 부분이 다르기에, 여러 디자인 패턴을 공부하고, 올바른 디자인 패턴을 채택하는 것이 중요하다.

# 7.1 생성 패턴

다음 패턴들에 대해 알아보자.

- 생성자 패턴
- 모듈 패턴
- 노출 모듈 패턴
- 싱글톤 패턴
- 프로토타입 패턴
- 팩토리 패턴

# 7.2 생성자 패턴

- 생성자 : 객체가 새로 만들어진 뒤, 초기화하는데 사용하는 특별한 메서드

## 객체 생성

일반적으로, 자바스크립트에서 새로운 객체를 만들 때 사용하는 세 가지 방법이 있다.

```jsx
// 1. 리터럴 표기법
const newObject = {};
// 2. Object.create() 메서드 사용
const newObject = Object.create(Object.prototype);
// 3. new 키워드 사용
const newObject = new Object();
```

3번째 방법은, Object 클래스의 생성자가 객체를 생성하는 역할을 하게 된다.

클래스는 새 객체를 초기화하는 constructor() 라는 이름의 메서드를 가지고 있어야 한다.

```jsx
class Car{
	constructor(model, year, miles){
		this.model = model;
		this.year = year;
		this.miles = miles;
}

toString(){
	return ...;
	}
}
```

와 같이 말이다.

하지만, 이런 생성자 패턴은 문제가 있는데

1. 상속이 어려워짐
2. Car 생성자로 객체를 생성할 때마다 toString()과 같은 함수를 새로 정의함

Car 유형의 인스턴스는 모두 동일한 함수를 공유해야 하므로, 이 방법이 효과적이지 않다.

## 프로토타입을 가진 생성자

이를 해결하기 위해, **프로토타입**을 이용할 수 있다.

자바스크립트의 프로토타입 객체는 특정 객체의 모든 인스턴스 내에 공통 메서드를 쉽게 정의할 수 있도록 해준다.

```jsx
class Car {
  constructor(model) {
    this.model = model;
  }
}
Car.prototype.toString = function () {
  return `{this.model} is model.`;
};

let civic = new Car('Honda Civi');
let mondeo = new Car('Ford Mondeo');

console.log(civic.toString());
console.log(mondeo.toString());
```

프로토타입 객체를 통해, Car 생성자로 생성된 모든 객체들이 toString 메소드를 재정의하지 않고도 사용할 수 있게 되었다.

즉, 모든 Car 객체는 toString() 메서드를 공유한다.

# 7.3 모듈 패턴

모듈은 프로젝트를 구성하는 코드 단위를 체계적으로 분리하고 관리하는데 효과적으로 활용된다.

초기 자바스크립트에서는 아래와 같은 방법들로 모듈을 구현했다.

- 객체 리터럴 표기법
- 모듈 패턴
- AMD 모듈
- CommonJS모듈

## 객체 리터럴

객체 리터럴 표기법은 중괄호({})안에서 키와 값을 쉼표로 구분하여 객체를 정의하는 방법이다. 오류 방지를 위해 마지막 줄 끝에는 쉼표 사용을 권장하지 않는다.

```jsx
const myObjectLiteral = {
  variableKey: variableValue,
  functionKey() {
    // ...
  },
};
```

객체 리터럴은 선언시 **new** 연산자를 필요로 하지 않는다.

그리고 다음과 같이 할당 연산자를 사용할 수 있다.

```jsx
myModule.property = 'someValue';
```

## 모듈 패턴 심화

### 비공개

모듈 패턴은 **클로저(closure)**를 활용해 ‘비공개’상태와 구성을 캡슐화한다. 이는 공개 및 비공개 메서드와 변수를 묶어 전역 스코프로의 유출을 방지하고, 다른 개발자의 인터페이스와의 충돌을 방지한다.

즉, 애플리케이션이 사용해야하는 부분만 노출하고, 핵심 작업은 보호할 수 있게된다.

ES2019 이전의 자바스크립트에서는 접근 제한자(#)를 지원하지 않았고, ‘비공개’라는 개념이 희미했다. 비공개를 구현할 수 있는 방법이 따로 존재하지 않았기에, 함수 스코프를 이용해 구현했다.

반환된 객체에 포함된 변수를 비공개하기 위해, **WeakMap()**을 사용할 수 있는데, 객체만 키로 설정할 수 있고, 순회가 불가능한 자료구조이다. 즉, 모듈 내부에 접근하는 유일한 방법은 해당 객체의 참조를 통해서만 가능하기에, 객체의 비공개를 보장한다.

## 모듈 패턴 구현

```jsx
let counter = 0;

const testModule = {
	incrementCounter(){
		return counter++;
	},
	resetCounter(){
		console.log(`counter value prior to reset: ${counter}`);
		counter = 0;
	},
};

// 변수명을 정하지 않고 디폴트 default로서 내보내는 방법
export default testModule;

// 사용방법

import testModule from ./testModule;

// 카운터 증가
testModule.incrementCounter();

// 카운터 값을 확인하고 리셋
testModule.resetCounter();
```

여기서 다른 파일들은 incrementCounter()나 resetCounter()을 직접 읽지 못한다.

counter 변수는 전역 스코프로부터 완전히 보호되어 비공개 변수로서 작동한다.

## 모듈 패턴의 변형

시간이 지나면서, 각자의 입맛에 맞는 모듈 패턴의 변형들이 등장하기 시작했다.

### 믹스인(Mixin)가져오기 변형

유틸 함수나 외부 라이브러리 같은 전역 스코프에 있는 요소를 모듈 내부의 고차 함수에 인자로 전달할 수 있게 한다. 이를 통해 전역 스코프 요소를 가져와, 맘대로 이름을 지정할 수 있다.

```jsx
// utils.js
export const min = arr => Math.min(...arr);

// privateMethods.js
import { min } from './utils';

export const privateMethod = () => {
  console.log(min([10, 5, 100, 2]));
};

// myModule.js
import { privateMethod } from './privateMethods';

const myModule = () => ({
  publicMethod() {
    privateMethod();
  },
});

export default myModule;

// main.js
import myModule from './myModule';

const moduleInstance = myModule();
moduleInstance.publicMethod();
```

이렇게 요리조리 이름을 자신의 입맛대로 바꿔서 사용할 수 있게된다.

### 내보내기 변형

따로 이름을 지정해주지 않고, 전역 스코프로 변수를 내보낸다.

```jsx
// module.js
const privateVariable = 'Hello World';

const privateMethod = () => {
  // ...
};

const module = {
  publicProperty: 'Foobar',
  publicMethod: () => {
    console.log(privateVariable);
  },
};

export default module;
```

## 장점

생성자 패턴도 좋은데, 모듈 패턴을 왜 사용해야할까?

- 모듈 패턴은 캡슐화 개념보다 초보 개발자가 이해하기 쉬움
- 전역 요소를 원하는 만큼 넘겨주어 코드의 유지보수를 쉽게하고 독립적으로 만들어줌.
- 비공개 지원(export로 노출한 값만 접근 가능)
- 전역 스코프 오염 방지(같은 이름을 가진 값을 덮어씌울 걱정 X)

## 단점

- 공개와 비공개 멤버를 서로 다르게 접근해야 함
- 나중에 추가한 메서드에서는 비공개 멤버에 접근할 수 없음.
- 비공개 멤버 수정과 관련한 오류를 고칠 때, 복잡도가 높아짐.

# WeakMap을 사용하는 최신 모듈 패턴

WeakMap 객체는 약한 참조를 가진 키-값의 쌍으로 이루어진 집합체이다.

키는 객체여야만 하지만, 값은 뭐든지 넣을 수 있다.

기본적으로 키가 약하게 유지되는 map이므로, 참조되지 않는 키는 가비지 컬렉션의 대상이 된다.

## WeakMap 모듈 패턴 예제

```jsx
let _counter = new WeakMap();

class Module(){
	constructor(){
		_counter.set(this, 0);
	}
	incrementCounter(){
		let counter = _counter.get(this);
		counter++;
		_counter.set(this, counter);

		return _counter.get(this);
	}
	resetCounter(){
		console.log(`counter value prior to reset : ${_counter.get(this)}`);
		_counter.set(this, 0);
		}
	}

	const testModule = new Module();

	// 사용법

	testModule.incrementCounter();
	testModule.resetCounter();
```
