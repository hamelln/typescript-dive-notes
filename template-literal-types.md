# Goal

> Learn about Template Literal Types introduced in TypeScript 4.1 and understand why Algebraic Data Types (ADTs) are necessary for a statically typed language.

Statically typed languages check types before runtime, catching errors without the hassle of runtime and buildtime.  
There is no perfect language. The limitation of most statically typed languages is an ad-hoc types system. Ad-hoc means "temporary, for this". For example, whenever we create a class or object, we create a type for it. There is no one-size-fits-all type that works perfectly in every situation.

### Why was this a problem??

To name just two, **high-level-abstraction and domain modeling are not easy**.  
For example, think of TypeScript without **Generics, Union Types, and Tuple Types** - it's terrible.  
There's a lot of research and improvements to make up for this. Martin-Löf Type Theory, Generics, Dependent Types in C++, and ADTs in Rust are good examples.

In TypeScript, ADTs are a heuristic for looking at types algebraically. The idea is to add (+) and multiply (x) types like algebra to create the types you need to solve a problem. There are two main types of ADTs: Sum Types (+) and Product Types (x). Let's look at a simple example.

```typescript
// Sum Types
type Direction = "up" | "down" | "right" | "left";
type Params = string | number;
// Product Types
type Person = { name: string; age: number };
type Tuple = readonly [boolean, number, string];
```

As you can see, ADTs are not a complicated concept!  
**Sum Types are types with multiple choices, and Product Types are multiple types coexisting inside of one** That's about it (and when you have systematic support for ADTs, like Rust, you can implement more powerful features).

### Template Literal Types?

Let's cut to the chase: Template Literal Types provide a powerful implementation of domain modeling for ADTs. Let's look at the code before Template Literal Types were introduced.

```typescript
// case 1: Each time a button type is added...
type Buttons = "a" | "b" | "x" | "y" | "home" | "zl" | "zr";

// 💔 You'll also need to add a method type for the button individually.
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
  // ... method
}

// case 2: How would you create this card type for poker?
type CardRank = 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | "J" | "Q" | "K" | "A";
type CardSuit = "♥" | "♠" | "♣" | "◆";
type Card = [CardSuit, CardRank] | "JOKER"; // 🙇‍♂️ Hmmm...not cool
```

In case 1, the code defining the method type must also be fixed every time a type is added or the type name is changed. This is cumbersome and can lead to developer mistakes.  
In case 2, the type is a tuple or the "JOKER" literal type. Unless that's what you're going for, it's inconsistent and ugly.

So let's improve the above code with Template Literal Types.

```typescript
// case 1: Add or change the type for the button only.
type Buttons = "a" | "b" | "x" | "y" | "home" | "zl" | "zr";
type CapitalizedButtons = Capitalize<Buttons>;
type ButtonHandlers = `on${CapitalizedButtons}`; // "onA" | "onB" | "onX" | "onY" | "onHome" | "onZl" | "onZr"

// 🎉 The method type is automatically updated to match the button type!
type ButtonsController = {
  [BH in ButtonHandlers]: () => void;
};

class Controller implements ButtonsController {
  // ... method
}

// case 2: Cards
type CardRank = 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | "J" | "Q" | "K" | "A";
type CardSuit = "♥" | "♠" | "♣" | "◆";
type Card = `${CardSuit}-${CardRank}` | "JOKER"; // Cool!
```

## Conclusion

ADTs help you think and solve problems. Enjoy more of TypeScript with Template Literal Types!
