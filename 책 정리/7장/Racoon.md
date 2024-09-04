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

# 구조 패턴

- 클래스와 객체의 구성을 다룸

다음의 패턴에 대해 배울 것임

- 퍼사드 패턴
- 믹스인 패턴
- 데코레이터 패턴
- 플라이웨이트 패턴

# 퍼사드 패턴

- Facade란, 실제 모습을 숨기고 꾸며낸 겉모습만을 세상에 드러내는 것을 의미함
- 퍼사드패턴은 심층적인 복잡성을 숨기고, 사용하기 편리한 높은 수준의 인터페이스를 제공하는 패턴
- jQuery와 같은 자바스크립트 라이브러리에서 흔히 볼 수 있는 구조 패턴
- 코드의 구현 부분과 사용 부분을 분리함

## 장점

- 사용하기 쉬움
- 패턴 구현에 필요한 코드의 양이 적음

```jsx
const addMyEvent = (el, ev, fn) => {
	if(el.addEventListener){
		el.addEventListener(ev, fn, false);
		} else if(el.attachEvent){
			el.attachEvent(`on${ev}`, fn);
			} else {
			el[`on${ev}`] = fn;
		}
};
```

# 믹스인  패턴

- 믹스인은 C++나 Lisp 같은 전통적인 프로그래밍 언어에서 서브클래스가 쉽게 상속받아 기능을 재사용할 수 있도록 하는 클래스

### 서브클래싱

- 부모 클래스 객체에서 속성을 상속받아 새로운 객체를 만드는 것을 의미함
- 부모 클래스의 먼저 정의된 메서드를 오버라이드 하는 것도 가능
- 오버라이드된 부모 클래스의 메서드를 호출할 수도 있음
    - 이를 메서드 체이닝이라고 부름
- 마찬가지로, 부모 클래스의 생성자를 호출할 수도 있음
    - 이를 생성자 체이닝이라고 부름

### 자바스크립트에서의 믹스인

- 자바스크립트에서는 기능의 확장을 위해 믹스인의 상속을 이용함
- 자식 클래스는 부모 클래스로부터 메서드와 속성을 부여받고, 자신만의 속성과 메서드를 정의할 수도 있음
- 믹스인은 최소한의 복잡성으로 객체의 기능을 빌리거나 상속할 수 있게 해줌

### 믹스인 코드

```jsx
const myMixins = superclass => 
	class extends superclass {
		moveUp(){
			console.log('move up');
		}
		moveDown(){
			console.log('move down');
		}
		stop(){
			console.log('stop! in te name of love!');
		}
	};
```

```jsx
// CarAnimator 생성자의 기본 구조
class CarAnimator{
	moveLeft(){
		console.log('move left');
		}
	}
	// PersonAnimator 생성자의 기본 구조
	class PersonAnimator{
		moveRandomly(){
			/*...*/
			}
	}
	
	// MyMixins을 사용하여 CarAnimator 확장
	class MyAnimator extends MyMixins(CarAnimator){}
	
	// carAnimator의 새 인스턴스 생성
	const myAnimator = new MyAnimator();
	myAnimator.moveLeft();
	myAnimator.moveDown();
	myAnimator.stop();
```

이렇게 믹스인을 이용하면, 비슷한 기능을 클래스에 추가하는 작업이 꽤 간단해진다.

### 장점

- 함수의 중복을 줄이고 재사용성을 높인다
- 객체 인스턴스 사이에 공유되는 기능이 있다면, 중복을 피하고 고유 기능을 구현하는데에 집중할 수 있음

### 단점

- 몇몇의 개발자들은 클래스나 객체의 프로토타입에 기능을 주입하는 것을 나쁜 방법이라고 여김
    - 프로토타입 오염과 함수의 출처에 대한 불확실성을 초래하기 때문

# 데코레이터 패턴

- 데코레이터 패턴은 코드 재사용을 목표로 하는 구조패턴
- 믹스인과 마찬가지로 객체 서브클래싱의 다른 방법이라고 생각하면 됨
- 기본적으로 클래스에 동적으로 기능을 추가하기 위해 사용함
- 애플리케이션의 기능이 다양한 타입의 객체를 필요로 할 때 적합
- 객체의 생성을 신경 쓰지 않는 대신 확장에 초점을 둠

### 장점

- 기존 시스템의 내부 코드를 힘겹게 바꾸지 않고도 기능을 추가할 수 있음

```jsx
// Vehicle 생성자
class Vehicle{
	constructor(vehicleType){
		// 일부 합리적인 기본값
		this.vehicleType = vehicleType || 'car';
		this.model = 'default';
		this.license = '00000-000';
	}
}

// 기본 Vehicle에 대한 인스턴스
const testInstance = new Vehicle('car');
console.log(testInstance);

// 데코레이트 될 새로운 차량 인스턴스를 생성합니다
const truck = new Vehicle('truck');

// Vehicle에 추가하는 새로운 기능 
truck.setModel = function(modelName){
	this.model = modelName;
}

truck.setColor = function(color){
	this.color = color;
}
```

해당 코드만 보기에는 데코레이터 패턴의 이점을 보여주기에는 부족하니, 추가로 예시를 살펴보자.

```jsx
class MacBook {
	constructor(){
		this.cost = 997;
		this.screenSize = 11.6;
	}
	getCost(){
		return this.cost;
	}
	getScreenSize(){
		return this.screenSize;
	}
}

// 데코레이터 1
class Memory extends MacBook {
	constructor(macBook){
		super();
		this.macBook = macBook;
	}
	getCost() {
		return this.macBook.getCost() + 75;
	}
}

// 데코레이터 2
class Engraving extends MacBook{
	constructor(macBook){
		super();
		this.macBook = macBook;
	}
	getCost() {
		return this.macBook.getCost() + 200;
	}
}

// 데코레이터 3
class Insurance extends MacBook{
	constructor(macBook){
		super();
		this.macBook = macBook;
	}
	getCost(){
		return this.macBook.getCost() + 250;
	}
}

let mb = new MacBook();

// 데코레이터 초기화
mb = new Memory(mb);
mb = new Engraving(mb);
mb = new Insurance(mb);
```

해당 예제에서 맥북의 업그레이드에 필요한 추가 비용을 반환하기 위해 MacBook 부모클래스 객체의 .getCost() 함수를 데코레이터로 오버라이드 했다.

# 플라이웨이트 패턴

- 반복되고 느리고 비효율적으로 데이터를 공유하는 코드를 최적화하는 해결 방법
- 연관된 객체끼리 데이터를 공유하게 하면서 애플리케이션의 메모리를 ‘최소화’ 하는 목적을 가짐
    - 목표 : 메모리 공간의 경량화
- 여러 비슷한 객체나 데이터 구조에서 공통으로 사용되는 부분 만을 하나의 외부 객체로 내보내는 것으로 이루어짐
- 각 객체를 데이터에 저장하기보다, 하나의 의존 외부 데이터에 모아서 저장

## 사용법

1. 데이터 레이어에서 메모리에 저장된 수많은 비슷한 객체 사이로 데이터를 공유하는 방법
2. 비슷한 동작을 하는 이벤트 핸들러를 모든 자식 요소에 등록하기보다는 부모 요소 같은 중앙 이벤트 관리자에게 맡기는 방법

→ 전통적으로는 1번 방식이 많이 사용됨

## 데이터 공유

- 내재적 상태
    - 객체의 내부 메서드에 필요한 것. (없으면 동작 X)
- 외재적 상태
    - 제거되어 외부에 저장될 수 있는 정보
    - 해당 정보를 다룰 때는 따로 관리자를 사용함. 한 가지 방법으로는, 플라이웨이트 객체와 내재적 상태를 보관하는 중앙 데이터베이스를 관리자로 사용하는 것이 있음

# 전통적인 플라이웨이트 구현 방법

- 플라이웨이트 패턴은 지금껏 자바스크립트에서 많이 사용되지 않아서, 자바나 C++ 생태계에서 영감을 얻어 구현
- 보여줄 샘플 코드는 아래 세 가지 특징을 가짐

### 세 가지 특징

- 플라이웨이트 : 외부의 상태를 받아 작동할 수 있게 하는 인터페이스
- 구체적 플라이웨이트 : 플라이웨이트 인터페이스를 실제로 구현하고 내부 상태를 저장
    - 다양한 컨텍스트 사이에서 공유될 수 있어야 하고, 외부 상태를 조작할 수 있어야 함
- 플라이웨이트 팩토리 : 플라이웨이트 객체를 생성하고 관리함.
    - 플라이웨이트를 공유할 수 있도록 보장해줌
    - 개별 인스턴스가 필요할 때 재사용할 수 있도록 관리함

### 메소드 설명

- CoffeOrder : 플라이웨이트
- CoffeeFlavor : 구체적 플라이웨이트
- CoffeOrderContext : 헬퍼
- CoffeFlavorFactory : 플라이웨이트 팩토리
- testFlyweight : 플라이웨이트 활용

### implements 덕 펀칭하기

- 덕 펀칭 : 런타임 소스를 수정할 필요 없이 언어나 솔루션의 기능을 확장할 수 있게 해줌
- 다음 코드에서, 인터페이스를 구현하기 위해 자바의 키워드(implements)를 필요로 하지만, 자바스크립트에는 원래 없는 기능이므로 덕 펀칭을 해볼 것임
- Function.prototype.implementsFor는 객체 생성자에 작용하며 부모 클래스 또는 객체를 받아들여 일반적인 상속 또는(함수일 때) 또는 가상 상속(객체일 때)을 이용해 상속 받는다

```jsx
// 인터페이스의 구현을 시뮬레이션 하기 위한 유틸리티 크래스

class InterfaceImplementation{
	static implementsFor(superclassOrInterface){
		if(superclassOrInterface instanceof Function){
			this.prototype = Object.create(superclassOrInterface.prototype);
			this.prototype.constructor = this;
			this.prototype.parent = superclassOrInterface.prototype;
		} else {
			this.prototpye = Object.create(superclassOrInterface);
			this.prototype.constructor = this;
			this.prototype.parent = superclassOrInterface;
	}
```

해당 코드는 implements 키워드의 부재를 보완하여, 함수가 인터페이스를 상속하도록 만들어준다

이제 CoffeFlavor은 CoffeeOrder 인터페이스를 구현하며, 사용하기 위해서는 인터페이스에 명시된 메서드를 반드시 구현해야 한다.

```jsx
// CoffeeOrder 인터페이스
const CoffeeOrder = {
	serveCoffee(context) {},
	getFlavor() {},
};

class CoffeeFlavor extends InterfaceImplementation{
	constructor(newFlavor){
		super();
		this.flavor = newFlavor;
	}
	getFlavor(){
		return this.flavor;
	}
	serveCoffe(context){
		console.log(`Serving Coffee flavor ${this.flavor} to
			table ${context.getTable()}`); // 커피 제공 로그
	}
}

// CoffeeOrder 인터페이스 구현
CoffeeFlavor.implementsFor(CoffeeOrder);

const CoffeeOrderContext = (tableNumber) => ({
	getTable(){
		return tableNumber;
	},
});

class CoffeeFalvorFactory {
	constructor(){
		this.flavors = {};
		this.length = 0;
	}
	getCoffeeFlavor(flavorName){
		let flavor = this.flavors[flavorName];
		if(!flavor) {
			flavor = new CoffeeFlavor(flavorName);
			this.flavors[flavorName] = flavor;
			this.length++;
		}
		return flavor;
	}
	getTotalCoffeFlavorsMade(){
		return this.length;
	}
}

// 사용 예시

const testFlyweight = () => {
	const flavors = [];
	const tables = [];
	let ordersMade = 0;
	const flavorFactory = new CoffeeFlavorFactory();
	
	function takeOrders(flavorIn, table){
		flavors.push(flavorFactory.getCoffeeFlavor(flavorIn));
		tables.push(CoffeeOrderContext(table));
		ordersMade++;
	}
	
	// 주문 처리
	takeOrders('Cappuccino', 2);
	
	// 주문 제공
	for(let i = 0; i < ordersMade; ++i){
			flavors[i].serveCoffee(tables[i]));
	}
	
	console.log(' ');
	console.log(`total CoffeeFlavor objects made:
		${flavorFactory.getTotalCoffeeFlavorMade()}`);
	};
	
	testFlyWeight();

```

## 중앙 집중식 이벤트 핸들링 방법

- 사용자 액션(예 : 클릭, 마우스 오버)에 따라 실행되는 비슷한 동작을 가진 여러 비슷한 요소들이 있다고 가정

부모 컨테이너 내부의 각 링크 요소에 ‘클릭’ 이벤트를 바인딩하지 말고, 최상위 컨테이너에 플라이웨이트를 부착하여 하위 요소로부터 전달되는 이벤트를 감지할 수 있도록 할 수 있음

### 코드 소개 전 설명

플라이웨이트의 로직을 캡슐화하여 담아두기 위해 stateManager 네임 스페이스를 사용하고, div 컨테이너에 클릭 이벤트를 바인드하기 위해 jQuery를 사용함.

그 전에 unbind 이벤트를 통해 컨테이너에 붙은 다른 핸들러를 떼어내도록 한다.

정확히 어떤 자식 요소가 클릭되었는지 확인하기 위해서 target을 체크한다. target은 부모와 상관없이 클릭된 요소가 어떤 것인지에 대한 참조를 제공함.

중요한 것은, 페이지가 로드되고 난 후, 모든 자식 요소들에 이벤트를 바인딩할 필요 없이 click 이벤트를 다룰 수 있다는 것.

```html
<div id = "container">
	<div class = "toggle"> More Info (Address)
		<span class="info">
			This is more information
		</span>
	</div>
	<div class = "toggle">Even More Info (Map)
		<span class ="info">
			<iframe src="MAPS_URL"></iframe>
		</span>
	</div>
</div>

<script>
	<function() {
		const stateManager = {
			fly(){
				const self = this;
				$('#container')
					.off()
					.on('click', 'div.toggle', function() {
						self.handleClick(this);
					});
				},
				handleClick(elem){
					$(elem)
						.find('span')
						.toggle('slow');
					},
				};
				
				// 이벤트 리스너 초기화
				stateManager.fly();
			})();
</script>
```

해당 방식의 장점은 개별적으로 관리되는 동작을 하나의 동작으로 바꾸어 메모리 절약이 가능하게 해준다는 점이다.

# 행위 패턴

- 행위 패턴은 객체 간의 의사소통을 돕는 패턴
- 시스템 내 서로 다른 객체 간의 의사소통 방식을 개선하고 간소화하는 것이 목적

### 자바스크립트의 행위 패턴 종류

- 관찰자 패턴
- 중재자 패턴
- 커맨드 패턴

# 관찰자 패턴

- 관찰자 패턴은 한 객체가 변경될 때 다른 객체들에 변경되었음을 알릴 수 있게 해주는 패턴
- 변경된 객체는 누가 자신을 구독하는지 알 필요 없이 알림을 보낼 수 있음
- 한 객체(주체)를 관찰하는 여러 객체들(관찰자)이 존재하며, 주체의 상태가 변화하면 관찰자들에게 자동으로 알림을 보냄
- 관찰자가 더 이상 주체의 변경에 대한 알림을 받고 싶지 않을 경우, 관찰자 목록에서 제거 가능

## 관찰자 패턴의 요소

- 주체 : 관찰자 리스트를 관리하고, 추가와 삭제를 가능하게 합니다.
- 관찰자 : 주체의 상태 변화 알림을 감지하는 update 인터페이스를 제공합니다.
- 구체적 주체(ConcreteSubject) : 상태 변화에 대한 알림을 모든 관찰자에게 전달하고, ConcreteObserver의 상태를 저장합니다.
- 구체적 관찰자(ConcreteObserver) : ConcreteSubject의 참조를 저장하고, 관찰자의 update 인터페이스를 구현하여 주체의 상태 변화와 관찰자의 상태 변화가 일치할 수 있도록 합니다.

### 주체가 가질 수 있는 관찰자 목록 구현 코드

```jsx
class ObserverList {
	constructor() {
		this.observerList = [];
	}
	add(obj) {
		return this.observerList.push(obj);
	}
	count() {
		return this.observerList.length;
	}
	get(index) {
		if(index > -1 && index < this.observerList.length) {
			return this.observerList[index];
		}
	}
	indexOf(obj, startIndex) {
		let i = startIndex;
		
		while(i < this.observerList.length) {
			if(this.observerList[i] === obj) {
				return i;
			}
		}
		return -1;
	}
	
	removeAt(index){
		this.observerList.splice(index, 1);
		}
	}
```

### 관찰자 목록 추가, 제거, 알림 기능 구현 코드

```jsx
class Subject {
	constructor(){
		this.observers = new ObserverList();
	}
	
	addObserver(observer){
		this.observers.add(observer);
	}
	
	removeObserver(observer){
		this.observers.removeAt(this.observers.indexOf(observer, 0));
	}
	notify(context){
		const observerCount = this.observers.count();
		for(let i = 0; i < observerCount; i++){
			this.observers.get(i).update(context);
		}
	}
}

// 관찰자 클래스

class Observer {
	constructor() {}
	update () {
		// ...
	}
}
```

## 관찰자 패턴과 발행/구독 패턴의 차이점

- 실제 자바스크립트 환경에서는 발행/구독 패턴(Publish/Subscribe) 이라는 변형된 형태가 더 많이 사용됨
- 두 패턴은 유사하지만 주목할만한 차이점이 존재
- 발행/구독 패턴은 이벤트 알림을 원하는 구독자와 이벤트를 발생시키는 발행자 사이에 토픽/이벤트 채널을 둠
    - 이러한 이벤트 시스템을 통해 애플리케이션에 특화된 이벤트 정의가 가능
- 발행/구독 패턴의 핵심은 발행자와 구독자를 각자 독립적으로 유지한다는 것
    - 또한 시스템의 구성 요소 간에 느슨한 결합을 도모한다는 것
    - 객체(발행자)가 다른 객체(구독자)의 메서드를 직접 호출하는 대신, 구독자는 특정 작업이나 활동을 구독하고 해당 작업이나 활동이 발생했을 때 알림을 받게 됨

```jsx
// 간단한 발행/구독 패턴 구현

const events = (function () {
	const topics = {};
	const hOP = topics.hasOwnProperty;
	
	return {
		subscribe: function (topic, listener){
			if(!hOP.call(topics, topic)) topics[topic] = [];
			const index = topics[topic].push(listener) - 1;
			
			return {
				remove: function(){
					delete topics[topic][index];
				},
			};
		},
		publish : function(topic, info){
			if(!hOP.call(topics, topic)) return;
			topics[topic].forEach(function (item) {
				item(info !== undefined ? info : {});
			});
		},
	};
})();
```

### 장점

- 여러 구성 요소 간의 관계를 심도 있게 고민해볼 수 있는 기회를 마련해줌
- 각각의 요소들이 직접 연결되어 있는 곳을 파악하여, 주체와 관찰자의 관계로 대체할 수 있는 부분을 찾아낼 수 있도록 도움을 줌
    - 이를 통해 애플리케이션을 더 작고 느슨하게 연결된 부분으로 나눌 수 있음
    - 결국, 코드의 관리와 재사용성을 높임
- 한 객체가 다른 객체들이 어떤 식으로 구현되어 있는지 알 필요 없이, 알림을 보낼 수 있음
- 시스템의 구성 요소 간의 결합도를 낮추도록 도와줌

### 단점

- 구독자와 발행자의 연결을 분리함으로써, 애플리케이션의 특정 부분들이 기대하는 대로 동작하고 있다는 것을 보장하기 어렵게 함
- 구독자들이 서로의 존재에 대해 전혀 알 수 가 없고 발행자를 변경하는 데 드는 비용을 파악할 수가 없음

# 중재자 패턴

- 하나의 객체가 이벤트 발생 시 다른 객체들에게 알림을 보낼 수 있는 디자인 패턴
- 하나의 객체가 다른 객체에서 발생한 ‘특정 유형’의 이벤트에 대해 알림을 받을 수 있다는 것이 특징
    - 관찰자 패턴은 하나의 객체가 다른 객체에서 발생하는 ‘다수의 이벤트’ 를 구독하는 것이 차이
- 여러 개의 이벤트 소스를 하나의 객체로 보내는 방법을 발행/구독 또는 이벤트 집합이라고 하는데, 이러한 문제에 직면했을 때 중재자 패턴을 고려할 수 있음
- 중재자 패턴은 구성 요소 간의 관계를 관리함으로써 직접 참조를 없애고 느슨한 결합을 가능케 함
    - 이는 시스템의 결합도를 낮추고 구성 요소의 재사용성을 높여줌
- 예시로 항송 교통 관제 시스템이 있음. 관제탑(중재자)은 항공기의 모든 통신(이벤트 알림)이 관제탑을 거쳐 이루어지고, 항공기끼리의 직접 통신을 하지 않게 관리함

### 발행/구독 패턴과 중재자 패턴의 유사점과 차이점

- 유사점
    - ‘이벤트’ 와 ‘서드 파티 객체’ 라는 핵심 요소
- 차이점

발행/구독 패턴(이벤트 집합 패턴)과 중재자 패턴 모두 이벤트를 사용함.

이벤트 집합 패턴이 이벤트를 다루는 패턴이라는 것은 이름에서 알 수 있듯 명백함. 반면에 중재자 패턴은 웹 애플리케이션 프레임워크에서 구현을 단순화하기 위해 이벤트를 그저 활용할 뿐임.

따라서, 중재자 패턴이 반드시 이벤트를 다룰 필요는 없음.

즉, 이벤트 집합 패턴은 설계된 목적 자체가 이벤트를 처리하기 위함이고, 중재자 패턴은 편리한 구현의 도구로 활용하는 것 뿐임.

### 서드 파티 객체

두 패턴 모두 서드 파티 객체를 사용함.

이벤트 집합 패턴 자체는 이벤트 발행자와 구독자에 대해 서드 파티 객체이며, 모든 이벤트가 통과하는 중앙 허브의 역할을 함

중재자 패턴 또한 다른 객체에 대한 서드 파티 객체임. 그렇다면 차이점은?

→ 애플리케이션 로직과 워크플로가 어디에 구현되어 있는지에 차이가 있다.

### 이벤트 집합 패턴에서의 서드 파티 객체

- 이벤트 집합 패턴에서의 서드파티 객체는 알 수 없는 수의 소스에서 알 수 없는 수의 핸들러로 이벤트가 연결되도록 지원하는 역할만 하.
- 실행되어야 하는 모든 워크플로와 비즈니스 로직은 이벤트를 발생시키는 객체(소스)와 처리하는 객체(핸들러)에 직접 구현됨

### 중재자 패턴에서의 서드 파티 객체

- 중재자 패턴에서 비즈니스 로직과 워크플로는 중재자 내부에 집중됨
- 중재자는 자신이 보유한 정보를 바탕으로 각 객체의 메서드 호출 시점과 속성 업데이트의 필요성을 판단함
    - 이를 통해 워크플로와 프로세스를 캡슐화 함
    - 또한 여러 객체 사이를 조율해 시스템이 원하는 대로 동작하도록 함
- 워크플로의 각 객체는 각자의 역할을 수행하는 방법을 알고 있지만, 중재자는 보다 거시적인 차원에서의 결정을 통해 객체들에 적절한 작업 시기를 알려줌

## 이벤트 집합 패턴(발행/구독)의 활용

- 일반적으로 이벤트 집합 패턴은 직접적인 구독 관계가 많아질 경우 또는 전혀 관련 없는 객체들 간의 소통이 필요할 때 사용됨
- jQuery의 on() 메서드는, 이벤트를 발생시키는 DOM 요소가 너무 많은 경우에 활용하면 좋은 예시. 10개, 20개, 200개 이상의 DOM 요소에 각각 ‘클릭’ 이벤트를 등록하는 것은 비효율적임

## 중재자 패턴의 활용

- 중재자 패턴은 두 개 이상의 객체가 간접적인 관계를 가지고 있고 비즈니스 로직이나 워크플로에 따라 상호작용 및 조정이 필요한 경우에 유용함

## 발행/구독 패턴과 중재자 패턴 결합하기

- 두 패턴이 왜 혼용되면 안 되는지에 대한 이해를 돕기 위해, 두 패턴을 함께 사용해보자

```jsx
const MenuItem = MyFrameworkView.extend({
	events: {
		'click .thatThing' : 'clickedIt',
	},
	
	clickedIt(e){
		e.preventDefault();
		
		// "menu:click:foo"를 실행한다고 가정
		MyFramework.trigger(`menu:click:${this.model.get('name')}`);
	},
});

// 애플리케이션의 다른 곳에서 구현

class MyWorkflow {
	constructor() {
		MyFramework.on('menu:click:foo', this.doStuff, this);
	}
	
	static doStuff(){
		// 이곳에 여러 객체를 인스턴스화
		// 객체의 이벤트 핸들러 설정
		// 모든 객체를 의미 있는 워크플로로 조정
	}
}
```

이 예제에서는 MenuItem이 클릭될 때 menu:click:foo 이벤트가 발생한다. MyWorkflow 클래스의 인스턴스가 이 이벤트를 처리하고 모든 객체를 조율하여 이상적인 사용자 경험과 워크플로를 구현한다.

이벤트 집합 패턴을 통해 메뉴와 워크플로 사이의 명확한 분리를 구현할 수 있고, 중재자 패틀 통해 워크플로의 관리 및 유지보수성을 강화할 수 있는 코드이다.

# 중재자 패턴 vs 퍼사드 패턴

- 두 패턴 모두 기존 모듈의 기능을 추상화하지만 미묘한 차이점이 존재
- 중재자 패턴은 모듈이 명시적으로 중재자를 참조함으로써, 모듈 간의 상호작용을 중앙집중화함.
    - 이는 본질적으로 다방향성을 지님
- 반면에 퍼사드 패턴은 모듈 또는 시스템에 직관적인 인터페이스를 제공하지만 추가 기능을 구현하지는 않음
    - 시스템 내 다른 모듈은 퍼사드의 개념을 직접적으로 인지하지 못하므로 단방향성을 지님

# 커맨드 패턴

- 커맨드 패턴은 메서드 호출, 요청 또는 작업을 단일 객체로 캡슐화하여 추후에 실행할 수 있도록 해줌
- 이를 통해 실행 시점을 유연하게 조정하고 호출을 매개변수화할 수 있음
- 또한 커맨드 패턴은 명령을 실행하는 객체와 명령을 호출하는 객체 간의 결합을 느슨하게 하여 구체적인 클래스(객체)의 변경에 대한 유연성을 향상시킴
- 커맨드 패턴의 기본 원칙은 명령을 내리는 객체와 명령을 실행하는 객체의 책임을 분리한다는 점
    - 커맨드 패턴은 이러한 책임을 다른 객체에 위임함으로써 역할 분리를 실현함
- 구현 측면에서 단순 커맨드 개체는 ‘실행할 동작’ 과 ‘해당 동작을 호출할 객체’를 연결
- 주요 장점은 인터페이가 동일한 모든 커맨드 객체를 쉽게 교체할 수 있다는 점

```jsx
const CarManager = {
	// 정보 조회
	requestInfo(model, id) {
		return `The information for ${model} with ID ${id} is foobar`;
	},
	
	// 자동차 구매
	buyVehicle(model, id){
		return `You have successfully purchased Item ${id}, a ${model}`;
	},
	
	// 시승 신청
	arrangeViewing(model, id){
		return `You have booked a viewing of ${model} ( ${id} ) `;
	},
};
```

- CarManager 객체는 자동차 정보 조회, 구매, 시승 신청 명령을 실행하는 커맨드 객체
- 이 객체의 메서드를 직접 호출하는 것도 가능하지만, 특정 상황에서는 이러한 직접 호출이 문제가 될 수 있음
    - 예를 들어, CarManager 객체 내부의 핵심 API가 변경된다고 가정했을 때, 이러한 경우에 메서드를 직접 호출하는 애플리케이션 내 모든 객체를 수정해야 하는 문제가 발생함.
    - 이러한 종류의 강한 결합은 OOP에서 지향하는 ‘느슨한 결합’ 원칙에 위배됨
    - 이 문제는 API를 보다 추상화함으로써 해결할 수 있음

커맨드 패턴의 이점을 살리기 위해 CarManager 객체를 확장해보자.

CarManager 객체에서 실행할 수 있는 메서드의 이름과 데이터(차량 모델, ID 등)를 매개변수로 받아 처리하는 구조로 개선할 것이다.

```jsx
CarManager.execute('buyVehicle', 'Ford Escort', '453543');
```

위 구조에 맞춰, carManager.execute 메서드의 정의를 추가한다.

```jsx
carManager.execute = function(name){
	return(
		carManager[name] &&
		carManager[name].apply(carManager, [].slice.call(arguments, 1))
	);
};
```

최종 코드는 다음과 같다.

```jsx
carManager.execute('arrangeViewing', 'Ferrari', '14523');
carManager.execute('requestInfo', 'Ford Mondeo', '54323');
carManager.execute('requestInfo', 'Ford Escort', '34232');
carManager.execute('buyVehicle', 'Ford Escort', '34232');
```