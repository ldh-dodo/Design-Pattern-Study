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

# 노출 모듈 패턴

- 객체의 이름을 반복해서 사용해야 한다는 점에 답답함을 느끼면서 창안
- 객체 리터럴 표기법을 사용해, 요소를 공개하는 것도 꺼려함

그 결과..

- 모든 함수와 변수를 비공개 스코프에 정의
- 공개하고 싶은 부분만 포인터를 통해 비공개 요소에 접근할 수 있게 해주는 익명 객체를 반환하는 패턴이 창안됨

```jsx
let privateVar = 'Rob Dobson';
const publicVar = 'Hey there!';

const privateFunc = () => {
  console.log(`Name : ${privateVar}`);
};

const publicSetName = strName => {
  privateVar = strName;
};

const publicGetName = () => {
  privateFunc();
};

// 비공개 함수와 속성에 접근하는 공개 포인터
const myRevealingModule = {
  setName: publicSetName,
  greeting: publicVar,
  getName: publicGetName,
};

export default myRevealingModule;

// 사용법

import myRevealingModule from './myRevealingModule';

myRevealingModule.setName('Matt Gaunt');
```

public 메서드를 통해, 비공개 변수인 privateVar에 접근한다.

노출 모듈 패턴을 사용하면 좀 더 구체적인 이름을 붙여 비공개 요소를 공개로 내보낼 수도 있다.

## 장점

- 코드의 일관성이 유지된다
- 모듈의 가장 아래에 위치한 공개 객체를 더 알아보기 쉽게 바꾸어 가독성을 향상

## 단점

- 비공개 함수를 참조하는 공개 함수를 수정할 수 없음
  - 위의 예시를 들어서, 비공개 함수를 참조하는 publicGetName을 아무리 수정해봤자, 함수가 변경될 뿐이지 참조된 구현(privateFunc)가 변경되는 것이 아님을 확인할 수 있다.
- 비공개 변수를 참조하는 공개 객체 멤버 또한 수정이 불가능
- 이러한 이유로 유지보수의 복잡도가 높음

# 싱글톤 패턴

- 클래스의 인스턴스가 오직 하나만 존재하도록 제한하는 패턴
- 전역에서 공유해야 하는 단 하나의 객체가 필요할 때 유용

```jsx
// 싱글톤에 대한 참조를 가지는 인스턴스
let instance;

const privateMethod = () => {
  console.log('I am private');
};
const privateVariable = 'Im also private';
const randomNumber = Math.random();

// 싱글톤
class MySingleton {
  // 싱글톤 인스턴스가 이미 존재한다면 참조를 반ㅂ환하고
  // 존재하지 않으면 생성
  constructor() {
    if (!instance) {
      this.publicProperty = 'I am also public';
      instance = this;
    }
    return instance;
  }
  publicMethod() {
    console.log('The public can see me!');
  }

  getRandomNumber() {
    return randomNumber;
  }
}

// 이름없이 기본값으로 내보내기
export default MySingleton;
```

## 싱글톤 패턴의 적합성

- 클래스의 인스턴스는 정확히 하나만 있어야 함
- 눈에 잘 보이는 곳에 위치시켜 접근을 용이하게 해야 함
- 싱글톤의 인스턴스는 서브클래싱을 통해서만 확장할 수 있어야 하고, 코드의 수정 없이 확장된 인스턴스를 사용할 수 있어야 함

## 단점

- 큰 모듈을 가져오는 경우, 어떤 클래스가 싱글톤 클래스인지 알아내기 어려움
- 위와 같은 이유로, 싱글톤 클래스를 일반 클래스로 착각하여 여러 객체를 인스턴스화하거나 부적절한 방법으로 수정할 가능성이 있음
- 테스트가 힘듬
- 신중한 조정이 필요함
  - 싱글톤의 지연된 실행 특성 때문에, 데이터가 유효하게 된 후에 사용할 수 있도록 올바른 실행순서를 구축할 수 있도록 해야함

## 리액트의 상태 관리

Redux나 Context API 같은 전역 상태 관리 도구를 이용해, 읽기 전용 상태를 활용할 수 있음. 전역 상태가 가지는 여러 단점이 상쇄되는 것은 아니지만, 적어도 컴포넌트가 전역 상태를 직접 변경할 수 없게 만들 수 있다는 장점이 있음.

# 프로토타입 패턴

- 이미 존재하는 객체를 복제해 만든 템플릿을 기반으로 새 객체를 생성하는 패턴
- 프로토타입의 상속을 기반으로 함
  - 이 때, 프로토타입 상속과 클래스는 별개로 사용됨.
  - 프로토타입 상속은 클래스처럼 따로 정의되는 것이 아니라, 이미 존재하는 다른 객체를 복제하여 새로운 객체를 만들어내는 개념
- 프로토타입 역할을 할 전용 객체를 생성하고, 해당 객체는 설계도 역할을 함

### 장점

- 다른 언어의 기능을 따라하지 않고, 자바스크립트만이 가진 고유의 방식으로 작업할 수 있음
- 객체 내에 함수를 정의할 때 복사본이 아닌 참조로 생성되어, 모든 자식 객체가 동일한 함수를 가리키게 함 → 성능상 이점

```jsx
const myCar = {
  name: 'Ford Escord',

  drive() {
    console.log("Weeee. i'm driving!");
  },
  panic() {
    console.log('Wait. How do you stop this  thing?');
  },
};

// 새로운 car를 인스턴스화하기 위해 Object.create를 사용
const yourCar = Object.create(myCar);

// 프로토타입이 제대로 들어왔음을 알 수 있습니다.
console.log(yourCar.name);
```

# 팩토리 패턴

- 객체를 생성하는 생성 패턴의 하나
- 생성자를 필요로 하지 않으나, 필요한 타입의 팩토리 객체를 생성하는 다른 방법을 제공

```jsx
// 자동차를 정의하는 클래스
class Car {
	constructor({doors = 4, state = 'brand new', color = 'silver'} = {}){
		this.doors = doors;
		this.state = state;
		this.color = color;
}

// 트럭을 정의하는 클래스
class Truck {
	constructor({state = 'used', wheelSize = 'large', color = 'blue'} = {}){
		this.state = state;
		this.wheelSize = wheelSize;
		this.color = color;
	}
}

// 차량 팩토리를 정의

class VehicleFactory {
	constructor(){
		this.vehicleClass = Car;
	}

	createVehicle(options){
		const { vehicleType, ...rest } = options;

		switch(vehicleType) {
			class 'car':
				this.vehicleClass = Car;
				breakk;
			}
			case 'truck':
				this.vehicleClass = Truck;
				break;
				// 해당되지 않으면 VehicleFactory.prototype.vehicleClass에 Car을 할당
		}
		return new this.vehicleClass(rest);
	}
}

// 자동차를 만드는 팩토리의 인스턴스 생성
const carFactory = new VehicleFactory();
const car = carFactory.createVehicle({
	vehicleType: 'car',
	color: 'yellow',
	doors: 6,
});

// 자동차가 vehicleClass/prototype Car로 생성되었는지 확인
console.log(car instanceof Car);
```

### 팩토리 패턴을 사용하면 좋은 상황

- 객체나 컴포넌트의 생성 과정이 높은 복잡도를 가지고 있을 때
- 상황에 맞춰 다양한 객체 인스턴스를 편리하게 생성할 수 있는 방법이 필요할 때
- 같은 속성을 공유하는 여러 개의 작은 객체 또는 컴포넌트를 다뤄야할 때
- 덕 타이밍같은 API 규칙만 충족하면 되는 다른 객체의 인스턴스와 함께 객체를 구성할 때
  - 디커플링에도 유용
