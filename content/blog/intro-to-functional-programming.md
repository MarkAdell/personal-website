---
title: Yet Another Introduction to Functional Programming
date: 2023-07-13
tags: ["programming", "functional-programming"]
ShowToc: true
---

In this article, we will explore the fundamental principles of functional programming that you can utilize in your day-to-day work, regardless of the programming language you use.

There is a common misconception among beginners that one can only write pure functional programming code, or only stick to other paradigms like OOP. This is not true. One of the beauties of functional programming is that you can make use of it in any programming paradigm you use, and it will not fail to make your code more maintainable. And towards the end of the article, we will show an example of how we can utilize these concepts within object-oriented programming code.

I decided to use JavaScript for most of the code examples because of its concise syntax and native support for some of the functional programming building blocks, such as treating functions as first-class citizens. However, it is likely that you can apply everything we will cover here in your preferred programming language.

## What is Functional Programming

Functional Programming is a programming paradigm that promotes writing code in a **declarative** rather than imperative manner and utilizes concepts such as **pure functions**, **immutability**, and **higher-order functions**, leading to code that is more maintainable and less prone to bugs.

We will explore each of these concepts, providing lots of code examples. Let's dive in!

## Imperative VS Declarative code

In imperative code, you write specific instructions to the compiler on **how** to compute a result, while in declarative code, you describe **what** is the result that you want to compute.

Let's see an example of imperative code that populates a new array with the even numbers from another array:

```javascript
// Imperative approach

function isEven(number) {
    return number % 2 === 0;
}

const numbers = [1, 2, 3, 4, 5, 6, 7, 8];

const evenNumbers = [];

for (let i = 0; i < numbers.length; i++) {
    if (isEven(numbers[i])) {
        evenNumbers.push(numbers[i]);
    }
}

console.log(evenNumbers); // [2, 4, 6, 8]
```

As you can see, we are giving specific instructions on how to compute the desired result. We manually implemented the filtering logic by looping through the array and populating the `evenNumbers` array based on a condition.

Now, let's see how to achieve the same result using declarative code:

```javascript
// Declarative approach

function isEven(number) {
    return number % 2 === 0;
}

function filter(array, predicate) {
  // implementation of the filtering logic
}

const numbers = [1, 2, 3, 4, 5, 6, 7, 8];

const evenNumbers = filter(numbers, isEven); // <-- the declarative code

console.log(evenNumbers); // [2, 4, 6, 8]
```

In the declarative approach, we described the desired outcome. We were like "Hey compiler, we want the `evenNumbers` array to hold the values of the `numbers` array after only including the even numbers".

Even if you are unaware of the inner implementation of the `filter` function, you will likely understand what is happening as the code is self-descriptive and reads like English. And this is the beauty of declarative code.

Please note that many programming languages provide built-in functions like `filter`. We will cover this later in the article when we discuss higher-order functions.

This is unrelated to functional programming, but we can make the code more concise using [arrow functions](https://www.w3schools.com/js/js_arrow_function.asp). Arrow functions provide a shorter syntax for creating function expressions and are supported in many programming languages, also known as *Lambda Expressions*.

```javascript
// Declarative approach using arrow functions

const numbers = [1, 2, 3, 4, 5, 6, 7, 8];

function filter(array, predicate) {
  // implementation of the filtering logic
}

const evenNumbers = filter(numbers, number => number % 2 === 0);

console.log(evenNumbers); // [2, 4, 6, 8]
```

Moving forward, we will use arrow functions to pass functions around in the code examples.

Let's see another example:

```javascript
// Imperative approach

const numbers = [1, 2, 3, 4, 5];

let max = numbers[0];

for (let i = 1; i < numbers.length; i++) {
    if (numbers[i] > max) {
        max = numbers[i];
    }
}

console.log(max); // 5

// Declarative approach

const numbers = [1, 2, 3, 4, 5];

const max = Math.max(...numbers);

console.log(max); // 5
```

## Pure functions

Pure functions are the heart of functional programming, and they play a fundamental role in applying other functional programming concepts.

Pure functions are functions that have the following properties:
1. They don't cause side effects to inputs or the global state.
    - Making the code **Less prone to bugs**.
2. When called with the same input, they always return the same output (Referential transparency).
    - Making the code **Predictable**, **Testable**, **Cacheable**.
3. They only depend on their input to produce the output.
    - Making the code **Less prone to bugs**, **Testable**, **Composable**.

Let's see examples of impure functions violating each of these properties and then how to rewrite them to be pure.

### Causing side effects #1:

```javascript
// Impure function #1

let counter = 0;

function incrementCounter() {
    counter++;
}

incrementCounter();

console.log(counter); // 1
```

The previous function is not pure because it causes side effects by modifying the `counter` variable (global state).

Refactored into a pure function:

```javascript
// Pure function #1

let counter = 0;

function incrementCounter(counter) {
    return counter + 1;
}

let incrementedCounter = incrementCounter(counter);

console.log(incrementedCounter); // 1

console.log(counter); // 0
```

Instead of modifying the global variable, we passed the counter as an input and returned the incremented value.

### Causing side effects #2:

```javascript
// Impure function #2

function sum(a, b) {
    console.log('Hello world!');
    return a + b;
}
```

The previous function is not pure as it causes side effects by logging something to the console.

```javascript
// Pure function #2

console.log('Hello world!');

function sum(a, b) {
    return a + b;
}
```

By removing the `console.log`, the function no longer causes side effects. If you are so eager to greet the world, you can do it outside of this function.

### Causing side effects #3:

```javascript
// Impure function #3

const numbers = [1, 2, 3, 4, 5];

function doubleNumbers(numbers) {
    for (let i = 0; i < numbers.length; i++) {
        numbers[i] = 2 * numbers[i];
    }
}

doubleNumbers(numbers);

console.log(numbers) // [2, 4, 6, 8, 10]
```

The previous function is not pure because it causes side effects by modifying its input.

Refactored into a pure function:

```javascript
// Pure function #3

const numbers = [1, 2, 3, 4, 5];

function doubleNumbers(numbers) {
    const result = [];

    for (let i = 0; i < numbers.length; i++) {
        result.push(2 * numbers[i]);
    }

    return result;
}

const doubledNumbers = doubleNumbers(numbers);

console.log(doubledNumbers) // [2, 4, 6, 8, 10]

console.log(numbers) // [1, 2, 3, 4, 5]
```

Instead of modifying the input, we created a new array `result` to store the updated values and returned it.

Please note that if arrays are passed by value in the programming language you use, it will be acceptable to modify the passed `numbers` array directly and return it, the function will be pure as it doesn't affect the original array.

### Not the same output for the same input:

```javascript
// Impure function #4

function isWeekend(weekendDays) {
    const currentDay = new Date().getDay();
    return weekendDays.includes(currentDay);
}

const weekendDays = [0, 6]; // Sunday and Saturday.

console.log(isWeekend(weekendDays)); // Might return true or false for the same input.
```

The previous function is not pure because it doesn't always return the same output for the same input, this happened because it doesn't depend fully on its input to produce the output, instead, it also depends on the internally generated current day.

Refactored into a pure function:

```javascript
// Pure function #4

function isWeekend(day, weekendDays) {
    return weekendDays.includes(day);
}

const currentDay = new Date().getDay();
const weekendDays = [0, 6]; // Sunday and Saturday.

console.log(isWeekend(currentDay, weekendDays)); // true if the given day is Sunday or Saturday, otherwise false.
```

Instead of making the function generate the day internally, we passed it as a parameter. By making the output fully depend on the input, we are now sure that the function will always return the same output for the same input.

### Not fully depending on the input to produce output:

```javascript
// Impure function #5

let taxRate = 0.1;

function calculateTotalAmount(itemPrice) {
    return itemPrice + (itemPrice * taxRate);
}

const itemPrice = 100;

const totalPrice = calculateTotalAmount(itemPrice);

console.log(totalPrice); // 110
```

The previous function is not pure because it doesn't depend fully on the input to produce the output, instead, it also depends on an external variable `taxRate`.

There is a catch here, the function is not impure just because it depends on an external variable. It is impure because this external variable can be modified at anytime, which means the function is not guaranteed to always return the same output for the same input.

Refactored into a pure function:

```javascript
// Pure function #5

function calculateTotalAmount(price, taxRate) {
    return price + (price * taxRate);
}

const itemPrice = 100;
const taxRate = 0.1;

const totalPrice = calculateTotalAmount(itemPrice, taxRate);

console.log(totalPrice); // 110
```

Instead of depending on an external variable, we passed all the needed variables to the function so that it depends fully on its input to produce the output.

Another approach:

```javascript
// Pure function #5 second approach

const taxRate = 0.1;

function calculateTotalAmount(itemPrice) {
    return itemPrice + (itemPrice * taxRate);
}

const itemPrice = 100;

const totalPrice = calculateTotalAmount(itemPrice);

console.log(totalPrice); // 110
```

By declaring `taxRate` with `const` instead of `let`, making it immutable, it is guaranteed that the function always returns the same output for the same input. However, it is advisable to use the first approach, which is passing the `taxRate` as a parameter, as this makes the function more self-contained.

### When to use Pure Functions

Whenever possible! It is important to recognize that not all functions in real-world applications can be pure. Sometimes the whole purpose of a function is to do side effects or rely on external dependencies to return or compute a value, think of I/O operations. So, when should you use Pure Functions? Whenever you find yourself writing a function whose purpose is to perform a computation or transformation based on a value that you have, then it is a good candidate for a pure function.

## Immutability

Immutability is the practice of not modifying data entities after they are created. When a change is required, a new data entity is created to hold the modified data, while the original data entity remains unchanged.

Immutability helps prevent bugs that arise when a developer assumes a data entity holds a specific value, when in reality, it has been changed elsewhere in the code.

Depending on the programming language you use, there may or may not be a way to enforce immutability, by having support for immutable variables and data structures. This is why I referred to immutability as a "practice", as in some cases, it will be your responsibility to avoid direct mutation (modification) of data entities.

To achieve immutability, we can follow these general principles, regardless of the programming language we use:
- Using constant variables if supported.
- Using immutable data structures if supported.
- Avoiding direct mutation to data entities.

Let's see some examples:

```javascript
// Bad: Variable is mutated.
let name = 'Mark';

name = 'DenMark';

// Better: New variable (data entity) is created to hold the updated value.
let name = 'Mark';

let newName = 'DenMark';

// Even better: Using constant variables to enforce immutability (A TypeError will be thrown if we reassigned name).
const name = 'Mark';

const newName = 'DenMark';
```

Please note that in JavaScript, the `const` keyword ensures that the variable cannot be reassigned. However, it does not make the value it holds immutable. For example:

```javascript
const numbers = [1, 2, 3];

numbers.push(4); // This is allowed.

console.log(numbers); // [1, 2, 3, 4]

numbers = [1, 2, 3, 4, 5]; // TypeError: Assignment to constant variable.
```

This is a case where it will be your responsibility to follow the "practice" of creating a new data entity to hold the new value, and avoid mutating the original one.

```javascript
const numbers = [1, 2, 3];

// Creating a new array by copying the old array and adding the new element 4.
const newNumbers = [...numbers, 4];

console.log(numbers); // [1, 2, 3]
console.log(newNumbers); // [1, 2, 3, 4]
```

The same principle applies to JavaScript objects:

```javascript
const person = {
    name: 'Mark'
};

// Creating a new object by copying the old object and adding the new age property.
const newPerson = {
    ...person,
    age: 13
};

console.log(person); // { name: 'Mark' }
console.log(newPerson); // { name: 'Mark', age: 13 }

// Even better: Enforcing Object immutability using Object.freeze()
const person = Object.freeze({
    name: 'Mark'
});

person.age = 13; // Fails silently.

console.log(person); // { name: 'Mark' }

const newPerson = {
    ...person,
    age: 13
};

console.log(newPerson); // { name: 'Mark', age: 13 }
```

I don't want to dive into more language-specific details as it is not the purpose of this article. The takeaway is, **enforce immutability when you can, otherwise, do your best following the practice of not mutating data entities**.

For many programming languages, there are either internal or external libraries that provide immutable data structures and utilities to achieve immutability. Some examples are:
- [immutable](https://www.npmjs.com/package/immutable) for JavaScript.
- [pyrsistent](https://pypi.org/project/pyrsistent/) for Python.
- [eclipse-collections](https://github.com/eclipse/eclipse-collections) for Java.
- [System.Collections.Immutable](https://www.nuget.org/packages/System.Collections.Immutable/) for C#.

Immutability doesn't stop at variables and data structures. Sometimes, it's also useful to make our classes immutable, such as Data Transfer Objects (DTOs), and it comes with its own benefits. As a final example (pun intended), let's see how we can build an immutable class in Java:

```java
final public class Person {
    private final String name;
    private final int age;
    private final List<String> hobbies;

    public Person(String name, int age, List<String> hobbies) {
        this.name = name;
        this.age = age;
        this.hobbies = Collections.unmodifiableList(hobbies);
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public List<String> getHobbies() {
        return hobbies;
    }
}
```

Note the following class properties:
- Class is declared as `final` to prevent inheritance.
- Class fields are declared as `final` to make them immutable.
- The `hobbies` list is made immutable by using `Collections.unmodifiableList()`.
- Class doesn't have any setter methods.

Immutability is a broad topic. We covered its basic fundamentals, and I hope I have made you curious to dive deeper.

## First-class functions

First-class functions refer to the ability of functions to be treated as values. This means you can assign functions to variables, pass them as arguments to other functions, and return them as values from other functions.

First-class functions serve as the foundation for techniques like higher-order functions and function composition in functional programming.

Let's see some examples:

```javascript
// Assigning functions to variables.
const sum = function(a, b) {
    return a + b;
}

console.log(sum(3, 4)); // 7

// Passing functions as arguments to other functions.
function calculate(operation, a, b) {
    return operation(a, b);
}

console.log(calculate(sum, 3, 4)); // 7

console.log(
    calculate(function(a, b) {
        return a * b
    }, 3, 4)
); // 12

console.log(
    calculate((a, b) => a - b, 3, 4)
); // -1

// Returning functions from other functions.
function getOperation(type) {
    if (type === 'add') {
        return function(a, b) {
            return a + b;
        }
    }

    return function(a, b) {
        return a - b;
    }
}

const operation = getOperation('add');

console.log(operation(3, 4)); // 7
```

We are ready now to move on to the last concept we'll explore today: Higher-order functions.

## Higher-order functions

Higher-order functions are functions that can take one or more functions as arguments or return functions as their results.

One of several benefits of higher-order functions is that they enable code reusability. Instead of writing similar logic, such as filtering, multiple times, you can encapsulate it in a higher-order function once and pass different functions as arguments to achieve different behaviors.

Many programming languages provide built-in higher-order functions for manipulating lists or collections without the need for loops. Making the code more concise, reusable, and readable. Some of the common higher-order functions for list manipulation are:
- **Map**: Applies a given function to each list element, returning a new list with the transformed values.
- **Filter**: Filters list elements based on a given predicate function (a function that returns a bool), returning a new list with the filtered elements.
- **Reduce**: Combines list elements into a single value using a given accumulator function, simplifying computations like summing or finding the maximum.
- **Find**: Searches for a specific element in a list based on a given predicate function.

And the list goes on.

Let's see a code example for filtering an array and explain the benefits of using higher-order functions:

```javascript
// Using the iterative approach (imperative).

const numbers = [1, 2, 3, 4, 5, 6, 7, 8];

const evenNumbers = [];

for (let i = 0; i < numbers.length; i++) {
    if (number[i] % 2 == 0) {
        evenNumbers.push(numbers[i]);
    }
}

console.log(evenNumbers); // [2, 4, 6, 8]

// Using a higher-order function (declarative).

const numbers = [1, 2, 3, 4, 5, 6, 7, 8];

const evenNumbers = numbers.filter(number => number % 2 === 0);

console.log(evenNumbers); // [2, 4, 6, 8]
```

Benefits of writing this code using the `filter` higher-order function:
- It's more concise because we achieved the same task using fewer lines of code.
- It's more readable because it's declarative, we describe **what** we want, not **how** to do it.
- It's more reusable because the main filtering logic is implemented once inside the `filter` function, allowing it to be used for different filtering scenarios based on the filtering criteria we provide.
- It's pure. Higher-order functions are often implemented to be pure, so we get all the nice benefits of pure functions.

So every time we use the iterative, imperative approach, we lose most of these benefits, along with some added drawbacks, such as:
- The risk of introducing bugs.
- The risk of writing unreadable code.
- The risk of introducing performance issues.

### Examples using higher-order functions

Let's see examples of using some of the built-in JavaScript higher-order functions:

```javascript
const numbers = [1, 2, 3, 4, 5, 6];

const squaredNumbers = numbers.map(number => number * number); // [1, 4, 9, 16, 25, 36]

const numbersLessThanSix = numbers.filter(number => number < 6); // [1, 2, 3, 4, 5]

const firstEvenNumber = numbers.find(number => number % 2 === 0); // 2

const hasNegativeNumbers = numbers.some(number => number < 0); // false

const initialValue = 0;
const sum =  numbers.reduce((accumulator, currentValue) => accumulator + currentValue, initialValue); // 21
```

### Composing higher-order functions

Remember when we said that higher-order functions are pure and this gives us the benefits of pure functions? One of these benefits is composability, it means that we can safely compose these functions together, by letting the output of one function be the input of another function. This is made possible because pure functions always get an input and produce an output, and output only depends on the input.

Let's write code that gets us the sum of odd numbers after multiplying each odd number by three:

```javascript
const numbers = [1, 2, 3, 4, 5, 6];

const result = numbers
    .filter(number => number % 2 != 0)
    .map(number => number * 3)
    .reduce((a, b) => a + b, 0);

console.log(result); // 27
```

We chained the three functions to compute our desired result. The output of the `filter` becomes the input of the `map`, and the output of the `map` becomes the input of the `reduce`.

Please note that chaining is one form of function composition.

### Implementing higher-order functions

Now to the fun part, let's see how we can implement these higher-order function ourselves. We will implement `map`, `filter`, `some`, and a bonus function called `countOccurrences` which doesn't have a built-in equivalent in JavaScript.

```javascript
function map(array, mapper) {
    const result = [];

    for (let i = 0; i < array.length; i++) {
        result.push(mapper(array[i]));
    }

    return result;
}

const squaredNumbers = map([1, 2, 3, 4], number => number * number);

console.log(squaredNumbers); // [1, 4, 9, 16]

// ------------------------

function filter(array, predicate) {
    const result = [];

    for (let i = 0; i < array.length; i++) {
        if (predicate(array[i])) {
            result.push(array[i]);
        }
    }

    return result;
}

const evenNumbers = filter([1, 2, 3, 4], number => number % 2 === 0);

console.log(evenNumbers); // [2, 4]

// ------------------------

function some(array, predicate) {
    for (let i = 0; i < array.length; i++) {
        if (predicate(array[i])) {
            return true;
        }
    }

    return false;
}

const hasNegativeNumbers = some([-1, 0, 1, 2], number => number < 0);

console.log(hasNegativeNumbers); // true

// ------------------------

function countOccurrences(array, predicate) {
    let counter = 0;

    for (let i = 0; i < array.length; i++) {
        if (predicate(array[i])) {
            counter += 1;
        }
    }

    return counter;
}

const evenNumbersCount = countOccurrences([1, 2, 3, 4], number => number % 2 == 0);

console.log(evenNumbersCount); // 2
```

Please note how they are all implemented to be pure.

### Higher-order functions exercise

Exercise time! Try to implement a higher-order function called `findLast` that accepts an array and a predicate function and returns the last element in the array that satisfies the given predicate, or `undefined` otherwise. This is how it will be used:

```javascript
const lastNumberBiggerThanOne = findLast([4, 9, 5, 6, 2], number => number > 1);

console.log(lastNumberBiggerThanOne); // 2
```

[Here](https://gist.github.com/MarkAdell/05afa2f6f873694c21eff85f49a9552d) is a possible solution to the exercise.

### Higher-order functions real life example

Let's see a more real life example for using higher-order functions, applying different discounts to a shopping cart:

```javascript
const cartItems = [
    { name: "Item 1", price: 10 },
    { name: "Item 2", price: 20 },
    { name: "Item 3", price: 30 },
];

function applyDiscount(item, discount) {
    const discountedPrice = item.price - (item.price * discount);
    return { ...item, price: discountedPrice }; // Copying the item object to another object and overriding the price value.
}

const regularDiscount = 0.1; // 10% discount.
const specialDiscount = 0.5; // 50% discount.

const regularCustomersCart = cartItems.map(item => applyDiscount(item, regularDiscount));
const specialCustomersCart = cartItems.map(item => applyDiscount(item, specialDiscount));

console.log(regularCustomersCart); // [{ name: "Item 1", price: 9 }, { name: "Item 2", price: 18 }, { name: "Item 3", price: 27 }]
console.log(specialCustomersCart); // [{ name: "Item 1", price: 5 }, { name: "Item 2", price: 10 }, { name: "Item 3", price: 15 }]
```

**It is important** to note that if `applyDiscount` mutates its input by modifying the item price directly, the `map` function will mutate the entire `cartItems` array, which we don't want. Therefore, it is crucial that the callbacks we pass to our higher-order functions are implemented to be pure.

As a final note, you may initially find yourself somewhat resistant to using higher-order functions in your code. It is understandable because all of us got introduced to loops first and became accustomed to using them. However, I can assure you that with little practice and commitment to using higher-order functions whenever possible, it will become second nature and you will start to appreciate their power and elegance.

## Can functional programming be used within OOP code?

Definitely! You can make use of several functional programming concepts while writing object-oriented code, such as immutability, pure functions, and higher-order functions.

Let's see a Java example where **all** the concepts we talked about in this article are applied:

```java
import java.util.List;
import java.util.stream.Collectors;

public class BranchService {
    private BranchRepository branchRepository;

    public BranchService(BranchRepository branchRepository) {
        this.branchRepository = branchRepository;
    }

    public List<String> getActiveBranches(int storeId) {
        List<Branch> branches = branchRepository.getBranchesByStoreId(storeId);
        List<String> formattedBranches = formatBranches(branches);
        return formattedBranches;
    }

    private List<String> formatBranches(List<Branch> branches) {
        return branches.stream()
                .filter(branch -> branch.isActive())
                .map(branch -> branch.getName().toUpperCase())
                .collect(Collectors.toList());
    }
}
```

This is a simple service class that has a public method `getActiveBranches`, which retrieves data from some data store, then applies a series of manipulations to the retrieved data before it returns it.

Functional programming concepts applied in this class:
- Preserving the **Immutability** of the `branches` list retrieved from the data store.
- Creating a **Pure Function** `formatBranches` that accepts the list of branches and returns a new list with the desired transformations.
- The `formatBranches` function is implemented in a **Declarative** manner, making use of **Higher-order functions**.

# Conclusion

There is still much more to learn about functional programming. Feel free to dive deeper into this beautiful paradigm. The aim of this article was to explain the basic principles that you can start applying today, making your code more maintainable and resistant to bugs.

Please feel free to suggest updates to the article by opening issues in [this](https://github.com/MarkAdell/functional_programming_article/issues) repository.

# References
- https://github.com/xgrommx/awesome-functional-programming
- https://dzone.com/articles/about-immutability-in-object-oriented-programming
- https://www.youtube.com/watch?v=0if71HOyVjY
- https://www.youtube.com/watch?v=e-5obm1G_FY
