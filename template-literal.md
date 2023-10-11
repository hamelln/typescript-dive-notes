# 목표

> TypeScript 4.1에 도입된 Template Literal Types을 알아보고 대수적 타입(Algebraic Data Types, 이후 ADTs)이 왜 정적 타입 언어에 필요한지 이해합니다.

정적 타입 언어는 런타임 전에 타입을 체크합니다. 그 덕분에 번거로운 실행 시간, 빌드 시간 없이 에러를 발견합니다.  
하지만 완벽한 언어는 없습니다. 대부분의 정적 타입 언어들이 가지는 한계점은 ad-hoc types system입니다.  
ad-hoc은 "임시적인, 즉흥적인"이라는 뜻입니다. 예를 들면 저희는 클래스, 객체를 만들 때마다 그에 맞는 타입을 만듭니다. 모든 상황에 완벽하게 호환되는 만능형 타입이 없으니까요.  

당연한 듯 보입니다. 이게 왜 문제였을까요? 여러 문제 중 두 가지만 꼽아보자면 **고도의 추상화(high-level-abstraction)와 범위 모델링(domain modeling)이 쉽지 않습니다.**  
예를 들어 **Generic, Union Types, Tuple Types가 없는 TypeScript**를 생각해보세요. 끔찍합니다.  
이를 보완하는 여러 연구와 타입 개선이 이뤄졌습니다. Martin-Löf Type Theory, Generic, C++의 Dependent Types, Rust의 ADTs가 좋은 예시입니다.  

TypeScript에서 ADTs는 타입을 대수적으로 바라보는 휴리스틱입니다. 타입을 대수학처럼 더하고(+), 곱해서(x) 문제해결에 필요한 타입을 만들어내자는 아이디어입니다.  
ADTs는 크게 Sum Types(+)와 Product Types(x) 두 가지로 나눕니다. 간단한 예시를 보죠.  

```typescript
// Sum Types
type Direction = "up" | "down" | "right" | "left"
type Params = string | number
// Product Types
type Person = { name: string, age: number }
type Tuple = readonly [boolean, number, string]
```

보다시피 ADTs는 거창하고 어려운 개념이 아닙니다!
**Sum Types는 여러 선택지가 있는 타입이고, Product Types는 하나 안에 여러 타입이 공존합니다.** 그게 전부입니다.(Rust처럼 시스템적으로 ADTs를 지원할 경우 더 강력한 기능 구현이 가능합니다.)  
본론으로 돌아가보죠. Template Literal Types는 ADTs의 domain modeling을 강력하게 구현합니다. 

Template Literal Types가 도입되기 전의 코드를 봅시다.  

```typescript
// case 1: 버튼 타입이 추가될 때마다...
type Buttons = "a" | "b" | "x" | "y" | "home" | "zl" | "zr";

// 💔 버튼에 대한 메소드 타입도 일일이 추가해야 한다.
type ButtonsController = {
  onA: () => void;
  onB: () => void;
  onX: () => void;
  onY: () => void;
  onHome: () => void;
  onZl: () => void;
  onZr: () => void;
};

class Controller implements ButtonsController {
  // ... 메소드 구현
}

// case 2: 포커용 카드 타입을 어떻게 만들지?
type CardRank = 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | "J" | "Q" | "K" | "A";
type CardSuit = "♥" | "♠" | "♣" | "◆";
type Card = [CardSuit, CardRank] | "JOKER"; // 🙇‍♂️ 흠...뭔가 별로다.
```

case 1은 타입이 추가되거나 타입 이름이 변경될 때마다 메소드 타입을 정의하는 코드도 매번 고쳐야 합니다. 이는 번거로울 뿐더러 개발자가 실수할 수도 있습니다.  
case 2는 타입이 튜플이거나 "JOKER" 리터럴 타입입니다. 의도한 바가 아닌 이상 통일감이 떨어지고 보기 안 좋네요.  

이제 Template Literal Types로 위 코드를 개선해봅시다.  

```typescript
// case 1: 버튼에 대한 타입만 추가하거나 변경합니다.
type Buttons = "a" | "b" | "x" | "y" | "home" | "zl" | "zr" 
type CapitalizedButtons = Capitalize<Buttons> // 📘 Capitalize: TS 4.1에 추가된 타입. string 리터럴 타입의 첫 글자를 대문자로 바꿉니다.
type ButtonHandlers = `on${CapitalizedButtons}` // "onA" | "onB" | "onX" | "onY" | "onHome" | "onZl" | "onZr"

// 🎉 메소드 타입은 버튼 타입에 맞춰서 자동으로 업데이트됩니다!
type ButtonsController = {
  [BH in ButtonHandlers]: () => void;
}

class Controller implements ButtonsController {
  // ... 메소드 구현
}

// case 2: 포커 카드
type CardRank = 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | "J" | "Q" | "K" | "A";
type CardSuit = "♥" | "♠" | "♣" | "◆";
type Card = `${CardSuit}-${CardRank}` | "JOKER"; // 53가지의 문자열 리터럴 타입 생성
```

이처럼 대수적 타입을 통한 사고방식과 방법은 문제해결에 도움을 줍니다. 연습을 통해 다양하게 응용해보세요!
