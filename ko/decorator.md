# Goals

> 타입스크립트 5.0에 추가된 데코레이터에 알아보고 연습합니다.

Editor's note: 이 글은 객체 지향 프로그래밍을 아는 독자를 대상으로 작성했습니다. 이 글에서 '객체'는. 모듈, class, 컴포넌트, 혹은 함수일 수도 있습니다. 구체적인 개념보단 넓은 관점으로 봐주세요.  

타입스크립트의 데코레이터는 [데코레이터 패턴](https://en.wikipedia.org/wiki/Decorator_pattern)을 일부 지원하는 기능입니다.  
데코레이터 패턴은 1994년에 Gang of Four의 [디자인패턴: 재사용 가능한 객체 지향 프로그래밍의 요소](https://en.wikipedia.org/wiki/Design_Patterns)에서 제시한 디자인 패턴입니다.  
해당 책에서는 디자인 패턴을 세 종류로 분류합니다.  

1. 생성 패턴(피자 레시피처럼 스마트하고 체계적으로 객체를 생성)
2. 구조적 패턴(다양한 객체를 조합해서 큰 구조를 짓는 것. 쉽게 변경, 확장, 결합함)
3. 행동형 패턴(퍼즐처럼 복잡하고 많은 객체들이 서로 어떻게 소통하고 동작할지에 대한 지침)

데코레이터 패턴은 [SOLID 원칙](https://en.wikipedia.org/wiki/SOLID)에서 S(SRP)와 O(OCP)를 담당합니다. 

- 단일 책임 원칙 (Single Responsibility Principle, SRP)  
객체는 한 종류에 대해서만 책임을 가져야 합니다. 한 객체가 여러 책임을 가지면 코드는 번잡해집니다. 모듈화와 가독성을 개선합니다.

- 개방 폐쇄 원칙 (Open-Closed Principle, OCP)  
객체는 확장에 대해선 개방적이되, 수정에는 폐쇄적이어야 합니다.  
즉, 새 기능을 도입할 땐 객체 코드를 수정하지 말고 새 코드를 추가합니다. 유연성을 보장합니다.

서비스에서 객체들은 처음엔 간단한 기능만 지원합니다. 그러다 서비스가 커질수록 기능은 많아지고 경우의 수는 여러 갈래로 늘어납니다.
커피 머신을 예로 들겠습니다. 옛날엔 커피 머신이 할 일은 단순했습니다. 커피 가루를 뜨거운 물에 섞어서 내놓기만 하면 됐습니다.  
그러나 이제는 다양한 고급 원두 머신이 있습니다. 각 머신은 레시피도 제각각입니다. 어떻게 하면 편리하게 구현할 수 있을까요?

```typescript
abstract class VarietyMachine {
  abstract coffeeBeans: number;
  abstract milkTemperature: number;
  abstract water: number;
  desiredMilkTemperature: number = 70;
  espresso(): void {}
  americano(water: number): void {}
  caffelatte(): void {}
  cappuccino(): void {}
}

function grindCoffeeBeans(beanGrams: number) {
  return <Class extends VarietyMachine>(
    target: Function,
    context: ClassMethodDecoratorContext
  ) =>
    function <Args extends any[]>(this: Class, ...args: Args) {
      this.coffeeBeans -= beanGrams;
      console.log(`원두를 ${beanGrams}g 분쇄합니다.`);
      target.apply(this, args);
      console.log(`머신에 원두가 ${this.coffeeBeans}g 남아있습니다.`);
    };
}

function extractEspresso(water: number) {
  return <Class extends VarietyMachine>(
    target: Function,
    context: ClassMethodDecoratorContext
  ) =>
    function <Args extends any[]>(this: Class, ...args: Args) {
      this.water -= water;
      console.log(`물을 ${water}ml 사용해서 에스프레소를 추출합니다...`);
      target.apply(this, args);
      console.log(`머신에 물이 ${this.water}ml 남아있습니다.`);
    };
}

function heatMilk(milk: number) {
  return function <Class extends VarietyMachine>(
    target: Function,
    context: ClassMethodDecoratorContext
  ) {
    return function <Args extends any[]>(this: Class, ...args: Args) {
      while (this.milkTemperature < this.desiredMilkTemperature) {
        this.milkTemperature++;
      }
      console.log(`${milk}ml의 우유를 ${this.milkTemperature}도로 데웠습니다.`);
      target.apply(this, args);
    };
  };
}

class StarbucksMachine extends VarietyMachine {
  coffeeBeans = 1000;
  water = 1000;
  milkTemperature = 30;
  desiredMilkTemperature = 75;

  @grindCoffeeBeans(20)
  @extractEspresso(50)
  espresso() {
    console.log("에스프레소가 나왔습니다.");
  }

  @grindCoffeeBeans(20)
  @extractEspresso(30)
  ristretto() {
    console.log("리스트레토가 나왔습니다.");
  }

  @grindCoffeeBeans(20)
  @extractEspresso(50)
  @heatMilk(140)
  caffelatte() {
    console.log(`Starbucks 카페라떼를 만듭니다...`);
    console.log(`Starbucks 카페라떼가 나왔습니다.`);
  }
}

class EdiyaMachine extends VarietyMachine {
  coffeeBeans = 1000;
  water = 1000;
  milkTemperature = 30;
  desiredMilkTemperature = 72;

  @grindCoffeeBeans(16)
  @extractEspresso(40)
  espresso() {
    console.log("에스프레소를 추출합니다.");
  }

  @grindCoffeeBeans(16)
  @extractEspresso(40)
  @heatMilk(120)
  caffelatte() {
    console.log(`Ediya 카페라떼를 만듭니다...`);
    console.log(`Ediya 카페라떼가 나왔습니다.`);
  }

  @grindCoffeeBeans(16)
  @extractEspresso(40)
  @heatMilk(80)
  cappuccino() {
    console.log(`Ediya 카푸치노를 만듭니다...`);
    console.log(`Ediya 카푸치노가 나왔습니다.`);
  }
}

const ediyaMachine = new EdiyaMachine();
const starbucksMachine = new StarbucksMachine();
starbucksMachine.caffelatte();
ediyaMachine.caffelatte();
ediyaMachine.cappuccino();
```
