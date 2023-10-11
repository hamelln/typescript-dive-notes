# 목표

> Branding 기법을 통해 TypeScript가 구조적 타입 시스템을 어떻게 보완하는지 이해합니다.

### 명목적 타입 시스템 vs 구조적 타입 시스템

타입 시스템은 크게 Nominal(or name-based) Type System과 Structural(or property-name-based) Type System이 있습니다. 코드로 차이를 알아봅시다.  

```typescript
// Nominal Type System(명목적 타입 시스템)
type Person = { name: string };
type Animal = { name: string };

const person: Person = { name: "현" };
const duck: Animal = person; // ❌ Error: Person과 Animal 타입은 다릅니다!

// Structural Type System(구조적 타입 시스템)
type Person = { name: string };
type Animal = { name: string };

const person: Person = { name: "현" };
const duck: Animal = person; // ✅ OK: "name"라는 속성을 가지고 있으니 허용합니다.
```

TypeScript는 구조적 타입 시스템을 지원합니다. JS와 호환성을 이루기도 하고 객체지향 프로그래밍 관점에서 보자면 "다형성"을 허용함으로서 유연해지기 때문입니다.  
그렇다면 명목적(이름 기반) 타입 시스템은 그저 융통성없는 고집쟁이일까요? 그렇지 않습니다. 아래 코드를 봅시다.

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

모든 국가의 통화는 속성 이름을 통일하고, 값의 타입도 number로 하되, USD와 EUR처럼 서로 다른 통화는 구분돼야만 합니다.  
이와 같은 상황은 속성 기반 타입 시스템의 한계를 잘 나타냅니다. 이름 기반 타입 시스템은 이런 문제를 겪을 일이 없지만 속성 기반 타입 시스템은 이 처리가 불가능합니다.  

### TypeScript의 명목적 타입

상술했듯이 TypeScript는 기본적으로 구조적 타입만 지원합니다. 하지만 Branding이라는 특정 조건 하에서는 **TypeScript에서도 명목적 타입을 생성**할 수 있습니다.

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

as로 타입을 단언함으로서 그 타입일 때에만 코드를 정상 실행합니다.  

이 예제 코드만으론 아직 branding이 왜 유용한지 공감이 덜 될 거라고 생각합니다. branding은 이보다 더 특정한 상황에서 강력한 힘을 발휘합니다.

[TypeScript 5.2.2 playground](https://www.typescriptlang.org/play?ssl=56&ssc=18&pln=42&pc=1#code/PTAEEFQOwewWwJZQIYBtQBcCeAHApqAM5aEZ5yhx7JSGYAWyGo1AxvZrgQnQK5QIAjrzwAoEKBoATFgDc8UUAgBmnfHUbyGBQsiqgpTZKCwxeoVjVjNkhQggDmUcWGSsATjDtq8hAHSiLqAAKlwAyh4IOBgA5HTY+EQkZBQ8RBjuvKwYvO5oADSgAO70COyU1LRBKto+SnSEjPgyqAgA1gTGUllthQixdF09fqAAkqrIQQ4wXgSMg6joGPQ6ep0YGQgARrxkg4MGPYXLCko2qIQw1QfdrG0jAJpmFjSgqNTuinAw7nN4vwAuFgADz0OHeANImWyuTQAFoEkgHIEJMFSnRLIpNAQpHkils3G1CIVlD8QWD3tpfpJqZZCL4giV-p10u4kaAyVBeHAtv8XljkFpCPhWAg0BYYFAyMCbFAZKZeEEpJKYswijRmBgYHI+XBkB1arI0CJ4nlaMp-nktu8RgAxH5BPCguDgvAAlxBOGgUAAVXp7jGUBwu1AYU2UAcdAAFPxdBaAJSe73BM2EVBMBCS0PhyNJ33+sZSBQYFRlDNZgBy3N57kIefArFYvjowRgHSqQQA6gRMaAHHhmN9SBzVCdQEbUCJQMpPBRjLBECgllwgsRSORiv0OMZWhtKVt+iOQRljKwYEWAl28DFqdN2VrQLx6ZJFEgyLW8NlM0ufIUilvJEfARhDECQz1oE83yURQx1JdwKBgCZQBwTx8HcbAXkWPAZAAfRwrYzRkKNlh4a4JSgeQpW-eNilKco9Q6OhDwQF0vHsa0CAfWx7CcSQglgeDxShe9tWMAA1NAEEMMgpFGINdjDNkI0vBICAk1ppOwuTgwwRT2QAXlZdkADJQAAb1APCCOkIEACI-T5bSQwABS8Zh1Kk8soFs0AAF8AG4UTAbtN0WR9n2MZR+C-LMHxPc0fjnIyI0wK4JHEyTNNk+TdJzUAvR2TUVmQmAoKKH5lmgGASwjMjliYYpr2pAArJ9mBwshFiRHCQnCSJogYBr+jiTBMjwS9wOHCdPLIBz3Cc5hDKjJAdKBYSIxo-SAD5zNEb1JuYewXXeDysoW0BDJW3Y-F+cE3DwKNgAAHQAHmABxClswATIls+NAu9X4ck+IgWNdU6mC0nLJDocGZIWvSI0CgKgtACsYCKRr+QsX4IenaKS0lOgSjKDh-zCyVUCwSRGzwaIgjMAMoDwDGFyQcVVMKaQqqKgh+yZvJ0HWhwfAmwnmBQt8KzWC7QCjFAqCBWHIZ0hGHE2nazL28jLhtVAYAcOW1j+0RkaCe0AydCk8EKFZfhGy59FjZALWgnTp1nQCn3+Qo71qiRlk8XgHA4MdpukskuZORReXZNA9aKbDUtAXlkKUmT3VEA7XZDQzbLQf4MCjGIthgLYtipjBkA4wgYnjWzAqzsOIeyt3DKb2b-QW5acuNiWpSlqgo3b5XdmNoIAHkYOK6rbdARg5UKHBuPvYr+DhYeZCF1Kgj7jAB4IMn0DyHgWTPF0EHeC33E8dwM93-fu50seJCecxexxmRT3gFC8BWWgEC0DAeQ7hZAIGZkeE4SoVAWl+FKYoyASBJw8NQMgVVFzsy4MSF8MgTgIHcEEE8RY4SIWUExKeaQAAsAAGKhEo4BUDgXrFKABxfoAAJXgWx6iEBEO6EAQR6AbBwIQAEIAHBbk4X4M+wAACyZRPCXGUBgYAoR8ARDZNEYAPAeG+GAAAJioXoj06U5QMDSDgNy9RAIOBxodbkep3BYD4cAARQiREgEQOwNAAAvfOdwEBSPgMAVmS4ES4CRHCJAYT1D9WUaIIAA)에서는 Access Tokens, User Identification Numbers, Translation Strings, 안전성이 확보되지 않은 User Input Strings 등 "특별한 string, number에 대해서 일반적인 string, number로 처리하면 안 될 때" 사용한다고 서술합니다.  

아래는 브랜딩 기법을 활용해서 XSS(Cross-Site Scripting) 처리를 하는 간단한 예시입니다.(이 코드만으론 XSS 대응이 완벽하지 않으니 주의하세요.)

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
