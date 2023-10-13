# 목표

> TypeScript의 Structural Type System 특성을 이해하고 타입 호환과 타입 체크를 연습합니다.

정적 타입 언어의 타입 시스템은 크게 두 가지 방식이 있다는 점을 알고 계신가요? 

1. 명목적 타입. Nominal Type System(혹은 Name-Based Type System)
2. 구조적 타입. Structural Type System(혹은 Duck Typing. 혹은 Property-Bame-Based Type System)

둘은 어떤 차이가 있을까요? 간단하게 표현할 수 있습니다!  

![SmartSelect_20231013-233257_Samsung Notes](https://github.com/hamelln/typescript-dive-notes/assets/39308313/6061de9f-003b-4164-ac8f-f05d98bf560b)

- 구조적 타입은 '속성, 구조'를 비교합니다. **둘 다 `number`속성을 가지므로 둘은 똑같다**고 판단합니다.
- 명목적 타입은 '이름, 단위'를 비교합니다. **`L`와 `m`는 단위가 다르므로 둘은 다르다**고 판단합니다.

어떠신가요? 위 그림을 통해 얻을 수 있는 결론은 '구조적 타입은 바보' 같이 보입니다. 코드로는 어떨까요?

```typescript
// 명목적 타이핑(타입 이름 기반 타이핑): Java, Swift...
type Game = { name: string }
type Animal = { name: string }
let zelda: Game = { name: "zelda" }
let duck: Animal = { name: "gooose" }
zelda = duck; // ❌ Error: zelda는 Game 타입입니다! Animal 타입을 할당하지 마세요!
```

```typescript
// 구조적 타이핑(속성 기반 타이핑): TypeScript...
type Game = { name: string }
type Animal = { name: string }
let zelda: Game = { name: "zelda" }
let duck: Animal = { name: "gooose" }
zelda = duck; // ✅ OK: "name" 속성을 만족하니까 할당 가능합니다!
```

"게임"과 "동물"은 엄연히 다릅니다. 하지만 `name` 속성이 공통된다는 이유만으로 서로 호환됩니다.  
볼수록 허술해보입니다! 이런 특성 때문에 구조적 타이핑은 덕 타이핑이란 이름으로도 많이 불립니다.  

Q) **“오리란 무엇인가?”**

1. 오리 같은 부리를 가졌다.
2. 오리 같은 눈을 가졌다.

동물 협회에서 위 두 조건을 만족하기만 하면 오리로 공인했다고 가정해봅시다. 그럼 아래의 사진은 "합성이 아닌 진짜 오리"라고 인정됩니다.

![rp0g39w789nqe8uzge8p](https://github.com/hamelln/typescript-textbook/assets/39308313/1b280fe5-0bc6-4c4c-bd15-2b34dd8baeaa)

### 왜 타입스크립트는 구조적 타입 시스템을 사용할까요? 

구조적 타이핑은 허술하기만 할까요? 다음 상황을 생각해봅시다.

```typescript
type Food = { carbohydrates: number; protein: number; fat: number };
type Burger = Food & { burgerBrand: string };

// 명목적 타입 시스템에서 이 함수는 Food 타입만 처리합니다.
function calculateCalorie({ carbohydrates, protein, fat }: Food) {
  return carbohydrates * 4 + protein * 4 + fat * 9;
}

const thighBurger: Burger = {
  carbohydrates: 60,
  protein: 28,
  fat: 27,
  burgerBrand: "Mom's Touch",
};

calculateCalorie(thighBurger); // 버거 타입의 칼로리 계산을 막는 게 좋을까요?
```

`Swift` 같은 명목적 타입 언어는 이런 케이스에 `Foodable protocol`을 `extension`으로 확장해서 사용합니다. 당연한 처리지만 불편하게 느껴지기도 합니다. 직관적으로 생각해봅시다. 햄버거는 `Food` 타입은 아니지만 탄수화물, 단백질, 지방 정보가 있으니까 바로 칼로리 계산을 허용한다면 편하지 않을까요?  

다행히도 `TypeScript`는 구조적 타입 시스템을 지원하기 때문에 위 코드를 정상 실행합니다!  
탄수화물, 단백질, 지방 속성을 갖췄으니 버거 브랜드라는 잉여 속성에 대해서는 신경을 안 씁니다. 이러한 유연성은 객체 지향 프로그래밍에서의 다형성을 허용합니다!  

이제 명목적 타입 시스템과 구조적 타입 시스템의 장단점이 짐작됐으리라 생각합니다. 짧게 정리해보죠.

- 명목적 타이핑: 정밀한 타입 체크로 에러 방지. 유연성, 호환성이 부족
- 구조적 타이핑: 타입 체크가 유연해서 편하지만 실수할 확률이 높아짐

보다시피, 융통성에 치우치면 타입 체크는 허술해집니다. `TypeScript`에는 이를 방지하는 기술이 몇 가지 있습니다. 그 중에 하나가 객체 리터럴입니다. 아래를 봅시다.

```typescript
calculateCalorie({
  carbohydrates: 20,
  fat: 22,
  protein: 11,
  burgerBrand: "Mom's Touch", // ❌ Error: 잉여 속성이 존재함.
});
```

먼젓번 코드에선 버거 브랜드가 있어도 칼로리 계산을 허용했습니다. 이번에는 왜 에러가 발생할까요?  
**TypeScript는 타입이 결정된(변수에 할당되거나 as로 타입 단언한) 객체에 대해서만 유연하게 타입을 체크**하기 때문입니다.  
`TypeScript`는 객체 리터럴을 $\textcolor{#3498DB}{\textsf{fresh한 객체}}$라고 따로 분류합니다.  
fresh한 객체는 타입 체크가 좀 더 까다롭습니다. 요구하는 속성보다 많으면 `Error`를 발생시킵니다. `carbohydrates`, `fat`, `protein` 속성만 허용하고, 추가적인 속성은 전부 거절합니다.  
이러한 시스템이 적용된 이유는 뭘까요?  
방금도 말했듯이, **구조적 타입 시스템은 타입 체크가 유연하기 때문에 개발자가 실수할 확률도 높아지기 때문**입니다. 아래의 예시를 봅시다.

```typescript
// 📒 만약 객체 리터럴에서 잉여 속성도 허용한다면?
// 😡 부작용 1: 다른 개발자는 burgerBrand가 필수 속성이라고 오해할 수 있습니다.
const calorie1 = calculateCalorie({
  protein: 29,
  carbohydrates: 48,
  fat: 13,
  burgerBrand: "버거킹", // ❌ Error: 잉여 속성이 존재함.
});

// 🤬 부작용 2: 잉여 속성에 오타가 있어도 TypeScript는 이를 발견할 수 없습니다!
const calorie2 = calculateCalorie({
  protein: 29,
  carbohydrates: 48,
  fat: 13,
  birgerBrand: "버거킹", // ❌ Error: 잉여 속성이 존재함.
});
```

이와 같이 내장된 로직으로 구조적 타이핑의 단점을 보완하기도 하지만, 개발자의 테크닉으로 보완할 때도 있습니다. 대표적인 예가 브랜딩입니다.  
브랜딩은 `TypeScript`가 따로 신기술이라고 발표한 것은 아니지만, 공식적으로 인정하는 테크닉입니다. 한시적으로 **명목적 타입 시스템을 구현**하는 일종의 묘기인데요. 자세한 내용은 [브랜딩 문서](https://github.com/hamelln/typescript-dive-notes/blob/main/branding.md)를 참고해주세요!

## 마무리

> TypeScript의 구조적 타입 시스템은 유연한 코드를 돕지만 개발자의 주의를 전제합니다. 많이 사용하면서 익숙해집시다!
