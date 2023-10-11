# 목표

> Branding 기법을 통해 TypeScript가 구조적 타입 시스템을 어떻게 보완하는지 이해합니다.

타이핑 시스템은 크게 Nominal(or name-based) Type System과 Structural(or property-name-based) Type System이 있습니다.  
코드로 확인해봅시다.  

```typescript
// Nominal Type System
type Person = { name: string };
type Animal = { name: string };

const person: Person = { name: "현" };
const duck: Animal = person; // ❌ Error: Person과 Animal 타입은 다릅니다!

// Structural Type System
type Person = { name: string };
type Animal = { name: string };

const person: Person = { name: "현" };
const duck: Animal = person; // ✅ OK: "name"라는 속성을 가지고 있으니 허용합니다.
```

객체 지향프로그래밍 관점에서 보자면 이와 같은 구조적(속성 기반) 타입 시스템은 프로그램의 "다형성"을 허용합니다.  
그렇다면 명목적(이름 기반) 타입 시스템은 그저 융통성없는 고집쟁이일까요? 아래 코드를 봅시다.

```typescript
// Nominal Type System
type USD = { value: number };
type EUR = { value: number };

let dollors: USD = { value: 10; }
let euros: EUR = { value: 15; }
dollors = euros; // ❌ Error: USD와 EUR 타입은 다릅니다!

// Structural Type System
type USD = { value: number };
type EUR = { value: number };

let dollors: USD = { value: 10; }
let euros: EUR = { value: 15; }
dollors = euros; // ✅ OK: "value"라는 속성을 가지고 있으니 허용합니다...이러면 안 되는데?
```

이와 같은 상황은 속성 기반 타입 시스템의 한계를 잘 나타냅니다. 속성의 이름을 통일하고, 값의 타입도 number로 해야 하지만 USD와 EUR은 서로 구분돼야만 합니다.  
이름 기반 타입 시스템은 이런 문제를 겪을 일이 없지만 속성 기반 타입 시스템은 이 처리가 불가능합니다.  
TypeScript는 Branding 기법을 통해 이런 한계점을 보완합니다.

```typescript
type Brand<K, T> = K & { __brand: T };
type USD = Brand<number, "dollors">;
type EUR = Brand<number, "euros">;

let dollors = 10 as USD; // number가 아닌 USD 타입으로 취급합니다.
dollors.__brand; // undefined. 숫자면서 __brand 속성도 가진 특이한 타입.
dollors = 20; // ❌ Error: 일반 숫자는 할당 불가능.
dollors = 1 as EUR; // ❌ Error: 이름이 다른 타입을 USD 타입에 할당할 수 없습니다.
dollors = 100 as USD; // ✅ OK: USD 타입만 재할당 가능
```

TypeScript 5.2.2 playground에서는 Access Tokens, User Identification Numbers, Translation Strings, 안전성이 확보되지 않은 User Input Strings 등 특별한 string, number에 대해서 일반적인 string, number로 처리하기는 곤란할 때 브랜딩 기법을 사용한다고 서술합니다. 
아래 코드는 XSS(Cross-Site Scripting)에 대한 처리를 하는 간단한 예시입니다.(이 코드만으론 XSS 대응이 완벽하지 않으니 주의하세요.)

```typescript
type ValidatedInputString = string & { __brand: "User Input Post Validation" };

// 입력 글자에 코드 실행문이 있다면 필터링하고, branding 처리한 string을 return합니다.
const validateUserInput = (input: string) => {
  const simpleValidatedInput = input.replace(/\</g, "≤");
  return simpleValidatedInput as ValidatedInputString;
};

// 검증된 branding 문자열만 받아서 출력합니다.
const printName = (name: ValidatedInputString) => {
  console.log(name);
};

// 코드 실행하는 문자열 
const input = "alert('bobby tables')";

// 검증을 거친 문자열
const validatedInput = validateUserInput(input);

printName(validatedInput); // ✅ OK: 검증 후 브랜딩 처리된 string이므로 정상 처리
printName(input); // ❌ Error: 검증 안 된(브랜딩이 안 된) 일반 string은 실행 거부
```

## 마무리

Branding 기법은 제한적으로 명목적(이름 기반) 타입 시스템을 구현하는 묘기입니다. 이름 단위로 엄격하게 타입 체크할 필요가 있을 때 사용하도록 연습해봅시다!
