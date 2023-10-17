# 목표

> Understand the nature of TypeScript's Structural Type System and practice type compatibility and type checking.

Did you know that type systems in statically typed languages have two main approaches?

1. Nominal Type System (or Name-Based Type System).
2. Structural Type System (or Duck Typing. or Property-Name-Based Type System)

What's the difference between the two?

![SmartSelect_20231013-233257_Samsung Notes](https://github.com/hamelln/typescript-dive-notes/assets/39308313/6061de9f-003b-4164-ac8f-f05d98bf560b)

- Nominal typing compares 'name, unit'. \*\*We determine that `L` and `m` are different because they have different units.
- Structural typing compares 'property, structure'. \*\*They both have the property `number`, so they are the same.

What do you think? Structured types already look "silly". What about object types?

```typescript
// Nominal Type System: Java, Swift...
type Game = { name: string };
type Animal = { name: string };
let zelda: Game = { name: "zelda" };
let duck: Animal = { name: "gooose" };
zelda = duck; // ❌ Error: zelda is a Game type! Don't assign it an Animal type!
```

```typescript
// Structural Type System: TypeScript...
type Game = { name: string };
type Animal = { name: string };
let zelda: Game = { name: "zelda" };
let duck: Animal = { name: "gooose" };
zelda = duck; // ✅ OK: It satisfies the "name" props, so it can be assigned!
```

A "game" and an "animal" are very different. However, they are interchangeable simply because they both have the `name` propertie. Because of this, structured typing is often referred to as "duck typing".

Q) \*\*"What is a duck?"

1. has a beak like a duck
2. has duck-like eyes.

Let's assume that an animal association recognizes a duck as long as it meets the above two conditions. Then the photo below is recognized as a "real duck, not a synthetic one".

![rp0g39w789nqe8uzge8p](https://github.com/hamelln/typescript-textbook/assets/39308313/1b280fe5-0bc6-4c4c-bd15-2b34dd8baeaa)

### Why does Typescript use a structured type system?

Is structured typing just clumsy? Consider the following situation.

```typescript
type Food = { carbohydrates: number; protein: number; fat: number };
type Burger = Food & { burgerBrand: string };

// In the nominal type system, this function only handles the Food type.
function calculateCalorie({ carbohydrates, protein, fat }: Food) {
  return carbohydrates * 4 + protein * 4 + fat * 9;
}

const thighBurger: Burger = {
  carbohydrates: 60,
  protein: 28,
  fat: 27,
  burgerBrand: "McDonald",
};

// Swift does this by extending the `Foodable protocol` with an `extension`.
calculateCalorie(thighBurger); // What about Typescript? Should we stop functions from calculating burger calories?
```

Let's think about it intuitively. Even though a hamburger isn't a `Food` type, it has carbohydrate, protein, and fat information, so wouldn't it be cool if we could **count calories right away**?  
Fortunately, `TypeScript` allows for calorie counting without any extra processing! This is because the hamburger has all of the Food's `properties` of carbohydrates, proteins, and fats - it even generously allows for an **extra property** of burger brand. What seemed like a rigid system works as flexibility. This characteristic goes hand in hand with polymorphism.

I'm sure you've guessed the pros and cons of nominal vs. structured type systems, so let's summarize.

- Nominal typing: precise type checking to prevent errors. Lack of flexibility, compatibility
- Structured typing: loose type checking, but that makes it flexible.

As you can see, structured typing is a double-edged sword: flexible and fragile. There are a few techniques in `TypeScript` to avoid this, one of which is object literals. See below.

```typescript
calculateCalorie({
  carbohydrates: 20,
  fat: 22,
  protein: 11,
  burgerBrand: "McDonald", // ❌ Error: A surplus property exists.
});
```

The code we saw earlier passed the calorie calculation even though it had a surplus property called burger brand. Why does it throw an error this time?  
**Because TypeScript only allows surplus properties for objects that have been typed(assigned to a variable or asserted as a type)**.  
TypeScript classifies object literals as $\textcolor{#3498DB}{\textsf{fresh object}}$ and throws an `Error` when it encounters a surplus properties.  
This means that we only need to write down three properties: `carbohydrates`, `fat`, and `protein`.

Why this specialization for object literals? Because **structured typing is more prone to developer mistakes**. Consider the following example.

```typescript
// 📒 What if we also allowed surplus properties in object literals?
// 😡 Side effect 1: Other developers might think that burgerBrand is a required property.
const calorie1 = calculateCalorie({
  protein: 29,
  carbohydrates: 48,
  fat: 13,
  burgerBrand: "McDonald", // ❌ Error: A surplus property exists.
});

// 🤬 Side effect 2: If there's a typo in a surplus property, TypeScript won't find it!
const calorie2 = calculateCalorie({
  protein: 29,
  carbohydrates: 48,
  fat: 13,
  birgerBrand: "McDonald", // ❌ Error: A surplus property exists.
});
```

### What if we wanted to compare the L and m we saw earlier?

Because `1000L` and `1000m` are both `number` types, structured typing, which only compares properties, has no answer!  
This is where developers turn to branding. Branding is a trick that **implements a nominal type system** - see the [branding](https://github.com/hamelln/typescript-dive-notes/blob/main/branding.md) for more details!

## Conclusion.

> TypeScript's structured type system supports flexible code, but it requires developer attention. Use it a lot and get used to it!
