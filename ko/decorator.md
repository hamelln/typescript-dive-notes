# Goals

> _타입스크립트 5.0에 추가된 데코레이터에 알아보고 연습합니다._

_Editor's note: 이 글은 객체 지향 프로그래밍을 아는 독자를 대상으로 작성했습니다. 이 글에서 '객체'는. 모듈, class, 컴포넌트, 혹은 함수일 수도 있습니다. 구체적인 개념보단 넓은 관점으로 봐주세요._

### 데코레이터 패턴

타입스크립트의 데코레이터는 [데코레이터 패턴](https://en.wikipedia.org/wiki/Decorator_pattern)을 일부 지원하는 기능으로 **클래스에서만 사용할 수 있습니다(2023/10 기준).**  
데코레이터 패턴은 1994년에 Gang of Four의 [디자인패턴: 재사용 가능한 객체 지향 프로그래밍의 요소](https://en.wikipedia.org/wiki/Design_Patterns)에서 제시한 디자인 패턴으로 [SOLID 원칙](https://en.wikipedia.org/wiki/SOLID)의 S(SRP)와 O(OCP)를 지키기 좋은 패턴입니다.

- 단일 책임 원칙 (Single Responsibility Principle, SRP)  
객체는 책임을 한 가지만 가져야 합니다. 한 객체가 여러 책임을 가지면 코드는 번잡해집니다.

- 개방 폐쇄 원칙 (Open-Closed Principle, OCP)  
객체는 확장에 대해선 개방적이되, 수정에는 폐쇄적이어야 합니다.  
즉, 새 기능을 도입할 땐 객체 코드를 수정하지 말고 새 코드를 추가합니다.

### 유연성과 확장성

서비스에서 객체들은 처음엔 간단한 기능만 지원합니다. 그러다 서비스가 커질수록 기능은 많아지고 경우의 수는 여러 갈래로 늘어납니다.
커피 머신을 예로 들겠습니다. 옛날엔 커피 자판기가 많았습니다. 디자인은 다들 다르지만 전국 어느 자판기를 이용해도 메뉴와 맛과 양은 똑같았습니다. 개발 코드로 생각해보면 다들 구현이 똑같은 수준이었을 겁니다.  

![SmartSelect_20231023-203237_Samsung Notes](https://github.com/hamelln/typescript-dive-notes/assets/39308313/709b4402-dd57-40a6-abfa-c26f0fd020c7)

오늘날엔 고급 원두 머신이 보편화됐고 **메뉴가 많습니다.** 메소드를 대폭 바꿔야겠군요.

![SmartSelect_20231023-203818_Samsung Notes](https://github.com/hamelln/typescript-dive-notes/assets/39308313/0254564b-d3eb-43e4-b64f-15156ccb7878)

**브랜드와 머신 종류가 많습니다**. 인터페이스로 틀을 만들고 각 객체마다 구현을 맡깁시다.  

![SmartSelect_20231023-204140_Samsung Notes](https://github.com/hamelln/typescript-dive-notes/assets/39308313/f418b0f4-627b-48b1-b02f-77f44fbb60a1)

**브랜드마다 레시피가 다릅니다.** 카페라떼만 해도 어디는 시나몬, 혹은 시럽을 넣을 수도 있습니다. 그러나 적어도 **원두를 분쇄하고 에스프레소를 만들고 데운 우유를 사용한다**는 로직은 모두 동일합니다.  
이렇게 재사용성 가능성이 많아 보이는 로직은 데코레이터로 분리하는 방법이 있습니다.

### 데코레이터

> _객체 고유의 기본 메소드를 만들고, 그 외에는 필요에 따라서 추가 기능을 덧붙입니다._

![SmartSelect_20231023-205352_Samsung Notes](https://github.com/hamelln/typescript-dive-notes/assets/39308313/bec103af-956c-4e27-a57e-cde36bb44045)

```typescript
class StarbucksMachine extends VarietyMachine {
  coffeeBeans = 1000; // 머신에 있는 원두의 양
  water = 1000; // 머신에 있는 물의 양
  desiredMilkTemperature = 75; // 우유를 데우는 온도

  @grindCoffeeBeans(20) // 원두를 20g 분쇄합니다.
  @extract(50) // 에스프레소를 50ml 추출합니다.
  @heatMilk(140) // 우유를 140ml 데웁니다.
  caffelatte() {
    console.log(`Starbucks만의 고유 재료와 비법 레시피를 이용한 카페라떼를 만듭니다...`);
    console.log(`Starbucks 카페라떼가 나왔습니다.`);
  }

  @grindCoffeeBeans(20) 
  @extract(40) 
  @heatMilk(80) 
  cappuccino() {
    console.log(`Starbucks만의 고유 재료와 비법 레시피를 이용한 카푸치노를 만듭니다...`);
    console.log(`Starbucks 카푸치노가 나왔습니다.`);
  }
}
```

우유를 사용하는 메뉴엔 heatMilk를 쓰고, 우유를 안 쓰는 메뉴엔 heatMilk를 안 씁니다. 이렇듯 필요한 때에 필요한 것만 가져와서 base를 꾸밉니다(decorate).  
어떤가요? 수많은 케이스에 유연한 대응이 가능합니다.  

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

const grindCoffeeBeans = (beanGrams: number) => <Class extends VarietyMachine>(
    target: Function,
    context: ClassMethodDecoratorContext
  ) =>
    function <Args extends any[]>(this: Class, ...args: Args) {
      this.coffeeBeans -= beanGrams;
      console.log(`원두를 ${beanGrams}g 분쇄합니다.`);
      target.apply(this, args);
      console.log(`머신에 원두가 ${this.coffeeBeans}g 남아있습니다.`);
    };

const extract = (water: number) => <Class extends VarietyMachine>(
    target: Function,
    context: ClassMethodDecoratorContext
  ) =>
    function <Args extends any[]>(this: Class, ...args: Args) {
      this.water -= water;
      console.log(`물을 ${water}ml 사용해서 에스프레소를 추출합니다...`);
      target.apply(this, args);
      console.log(`머신에 물이 ${this.water}ml 남아있습니다.`);
    };

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
  @extract(50)
  espresso() {
    console.log("에스프레소가 나왔습니다.");
  }

  @grindCoffeeBeans(20)
  @extract(30)
  ristretto() {
    console.log("리스트레토가 나왔습니다.");
  }

  @grindCoffeeBeans(20)
  @extract(50)
  @heatMilk(140)
  caffelatte() {
    console.log(`Starbucks만의 고유 재료와 비법 레시피를 이용한 카페라떼를 만듭니다...`);
    console.log(`Starbucks 카페라떼가 나왔습니다.`);
  }
}

class EdiyaMachine extends VarietyMachine {
  coffeeBeans = 1000;
  water = 1000;
  milkTemperature = 30;
  desiredMilkTemperature = 72;

  @grindCoffeeBeans(16)
  @extract(40)
  espresso() {
    console.log("에스프레소를 추출합니다.");
  }

  @grindCoffeeBeans(16)
  @extract(40)
  @heatMilk(120)
  caffelatte() {
    console.log(`Ediya만의 고유 재료와 비법 레시피를 이용한 카페라떼를 만듭니다...`);
    console.log(`Ediya 카페라떼가 나왔습니다.`);
  }

  @grindCoffeeBeans(16)
  @extract(40)
  @heatMilk(80)
  cappuccino() {
    console.log(`Ediya만의 고유 재료와 비법 레시피를 이용한 카푸치노를 만듭니다...`);
    console.log(`Ediya 카푸치노가 나왔습니다.`);
  }
}

const ediyaMachine = new EdiyaMachine();
const starbucksMachine = new StarbucksMachine();
starbucksMachine.caffelatte();
ediyaMachine.caffelatte();
ediyaMachine.cappuccino();
```

## 마무리

> _이 외에도 타입스크립트에서 추천한 포스팅에선 데코레이터를 메소드 전후처리, 직렬화 및 역직렬화의 자동 처리, 의존성 주입, 런타입 타입 평가 등 다용도로 사용한다고 서술합니다.  
디자인 패턴은 **그때의 문제를 해결하기 위해 탄생한 개념이지, 모든 프로그램에 적용하기 좋은 정석이 아님**을 명심하세요. 데코레이터 역시 무조건적인 개선을 보장하지는 않습니다. 따라서 팀원들, 혹은 스스로에게 다시금 물어보세요.  
"지금 문제를 개선하기 위해서는 데코레이터가 적절한가?"_
