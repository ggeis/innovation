# What is functional programming?

Functional programming is a coding technique, and there are languages more suited for it than others. This technique is directly linked to some of the most advanced branches of mathematics, but we are not bringing any theoretical maths to this practical guide.

A good way of starting functional programming is by analyzing each of its interconnected principles:
- Declarative
- Purity
- Referential transparency
- Immutability
## Declarative
We will start with the only functional programming principle that is subjective. Declarative programming could well be the inclination to separate further and further the description of a program from its evaluation. Declaring *what we want* (and progressively isolating away *how we'll get it*) inevitably ends using more expressions and functions and less control flow and state. Declarative programming fits better when we have a very restricted and clear model for the problem to solve. And even those mature solutions will still be having irreducible or just more desirable imperative parts.

![A image](https://ggeis.static.observableusercontent.com/files/f6b54228ec064ffbb740ec742a2fb1496f1220af2188c48786df4dcdd841a239514e766bf93962b77b38cf10ace51ce2d1dcb317780230095cefbba1d640474a?response-content-disposition=attachment%3Bfilename*%3DUTF-8%27%27imperativevsdeclarative.png&Expires=1616155200000&Key-Pair-Id=APKAJCHFJLLLU4Y2WVSQ&Signature=fG1odyXROhkytVr8IGnslVkP8syKZNqVgKJ8ex4EIn3VonlGDf5zcCi3nih2yAVEaaIOjOv6AsPdMDfDQIljdkCKbD0ThsIyK8kmndPmPsRXhN~HgUqKAoXPxxBydja6SzN711rmwi0bcmmMnEEjvEi0i8zNUW6BEmQ9EyfgGGX0fD6gJDK~VDxZAYZnzKYjlCOj6ERGHyf0uK6saZPxf7j~64iKKgh1HMwQpIrcToKWulgkY9NcSNVFuya0oNwoz5GOBbvtuVz9cCZGqQHuJI0laRyG4QvDvmN8bNvlE~D4NPWtytQQzhf-AExc~tVPEhtS4fBhhLNkUSHOlX7MrQ__)
*Comparison between imperative (left) vs declarative (right) code techniques.*

A familiar example of declarative programming is GraphQL, a language that lets us compose a restricted set of expressions to shape the desired result, abstracting itself completely away from all the mechanisms and internal processes that the GraphQL resolvers will use to obtain this declared output. Other examples of declarative programs are the Webpack configuration and plugin system, or the fluent APIs of some React-based libraries and React itself.

Another example is the `map` function of the Array `prototype`. It's a high-order (second-order) function, we'll see later why and how implementing `map` makes the Array built-in data type a functor. But for now, let's see how abstracting away the loop to the more specialized `map` has not just freed us from the repetitive responsibility of properly managing the index access state, but has brought a more declarative, decomposed, and type-checkable syntax.

```typescript
function imperative_statements () {
  let inputArray = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
  let newArray = [];
  for (let index = 0; index < inputArray.length; index++) {
    newArray[index] = Math.pow(inputArray[index], 2);
  }
  return newArray;
};
```
```typescript
function declarative_statements () {
  return [0, 1, 2, 3, 4, 5, 6, 7, 8, 9].map(num => Math.pow(num,2));
}
```
Having demostrated the declarative edge, it's fair to say that the imperative way of coding also have an important place in Javascript, as it tends to be faster and more optimized when done properly and in small scale, or when we temporary or permanently prioritize low level synergies rather than collaboration and mantainability.
### A first perspective on types
We can find another very explicit example of declarative programming in Typescript and its, rightly named so, `declare` keyword
```typescript 
declare function declarative_statements(): void;
declare function imperative_statements(): void;
```
We must reckon that these particular Typescript declarations are not very interesting and far from functional, but we're effectively declaring *what* we expect from the `declarative_statements` and `imperative_statements` functions, and enable compilers and other static analysis tools to complain or act after this declaration (for example automatically writing documentation). The next step could well be moving this declarations out to a .d.ts specialized file. Or even extract this .d.ts to a different npm package that might be installed as a external development dependency, etc. As we said keep widening the distance between the *what* from the *how* is what declarative programming emphasizes.
## Purity
Purity is a characteristic of some functions. A pure function is a function that is predictable (given the same input always returns the same output) and that has no side effects. Pure functions make code easier to read, understand, test, debug, refactor, decompose and compose. 

*Pure functions are closer to mathematical functions, but with one possible result*

Objects are the structures our model is shaped on, and functions are the machinery that moves the state of this model forward. The assembly of small functions into bigger ones allow for rewarding and extendible behaviors. As functions tend to be more versatile than the data that makes up our business model, function *composition* is the cornerstone of modern software: we can develop complex computation without renouncing to decoupling, flexibility and reusability. Pure functions are specially convenient on this regard because *pure functions always combine into pure functions*, and pure functions bring topmost excellence on correctness, parallelization and program flow no matter how many pieces they are composed from.
```typescript
const raiseTo = (pow) => (num) => Math.pow(num, pow);
const squareRootOf = raiseTo(0.5);
const squareRootOfEach = (array) => array.map(squareRootOf);
return squareRootOfEach([0, 1, 4, 9, 16, 25, 36, 49, 64, 81])
```
*Pure functions glued into pure functions using basic calls*

We should favor purity whenever is possible, but a lot of what we do (I/O, exception control, time) entail side-effects.
What we are meaning by side-effects is *effectful computations*, which gloom our codebases because handling them often involves to detour from abstract golden paths. Real-world implementations certanly carry integration, external agency, time handling, latencies, delays, failure... and  lot of defects come from a unappropiate engineering of those material operations.

Effectful computations cannot be resolved in a pure manner and there is 6 categories of them:
* * *
### Effectful computations

- **Non-terminating** 

 Some functions defy *run-to-completion* rule. It may be a defect or something deliberate like the infinite cycle.
 ```javascript 
while (true)
```

  *Generators*, a special kind of functions which can be interrupted and resumed, also belong to this category, even if they provide patterns to cordon other impure effects within itselves. More on that later.

- **Non-deterministic**

 Nondeterministic computations, given the same input and state, return different results between different calls. When purposeful they introduce RNG or a time-bound constant that varies according to the moment of evaluation.
 We have to ensure *superposition*, validation of every possible outcome, mostly with types but sometimes a serialization should be set in place to acknowlege for it.

- **Exceptions**

 This is similar to the non-terminating effect, the difference is that it's the code itself which *imperatively* throws the exception that ends the program in a controlled yet impure manner.
 ```javascript
throw new Error('Error message');
```

- **Continuations**
 
 This type of effects impact more on the `goto`-featured programming languages. As Javascript is not one of those, continuation effects often refer to callback origin non-terminating effects.

- **I/O**

 Both input and output are impure by being engaged in a continuous mutating intercourse with physical devices that can have blocking states, permission denial, network down, damage and other occurrences independent of our business logic. But this is not the full story.

 Input extra impurity might also come from the non-determinism of its interactivity. 

 Output's potential source of impurity will serve us to introduce another concept of functional programming: *lazy evaluation*, let's see an example:

  ```javascript 
const raiseTo = (pow) => (num) => {
    console.log(`calculating ${num}^${pow}...`);
    return pow === 0 ? 1 : Math.pow(num, pow);
}
```

 In normal conditions this `console.log` is inoffensive. `raiseTo` could pretend it´s a pure, even if a bit wasteful, function. But let's write a slightly different version of it to prove the point:

```javascript 
const IntensiveHundred = {
  base: 1
};

Object.defineProperty(IntensiveHundred, 'calculated', {
  get() {
    let num = 0;
    for (let i = 0; i < 100; i++) {
      num = num + this.base;
    }
    return num;
  }
});

const raiseTo = (pow) => (numObj) => {
  console.log(`calculating ${numObj.calculated\}^${pow} ...`);
  return pow === 0 ? 1 : Math.pow(numObj.calculated, pow);
};

const raiseToZero = raiseTo(0);
raiseToZero(IntensiveHundred);
```

   We can see how after adding some composition (getter), resources are allocated and spared just to feed the `console.log` call even if lazy evaluation is discarding the calculation for the actual returning result. This summarizes the doppelgänger problem with the output side-effects, it is something that our program definitely does but is not really a computation, it might look like a neutral operation but it is not, it stacks effects, and breaks referential transparency.
   
   - **Shared State** 

  Shared state is any variable, object, or memory space that exists in a shared scope, also the property of any object being passed or referenced between scopes. 

  Shared state carries the curse of being potentially changed by any piece of code in the application, and detecting what part of the code did the change involves to trace all the history and check all the code that uses that variable and guess how it happened. This is specially painful when the shared state is intermittent and you have to have luck trying to catch it red-handed.

  A classic example of Javascript shared state is the problematic \`this\` keyword, which gives you free access to instance, global or bind level data outside of the method scope.

  Every program holds state. The real question is whether it qualifies as functional state, there is different ways of handling state in a functional way: there is subtle differences between the different types of state access (read-only, write-only, read/write), and strategies to do it safely (change detection, types, reactivity...), there is also advanced functional structures that camouflage shared state and other effects in boxes called monads that are compatible with pure functions, we'll arrive there at some point, but for now let's say the first functional measure against shared state is immutability.
  
  * * *
  ### An unexpected plot twist
  
  There is two sides of purity: internal and external. It's convenient that a function is pure both from the inside and the outside, but by convention purity qualification is decided on the external.
In this regard is important to treat input values as immutable. Ideally, input values should be immutable values and the only thing its changed in the function. It's also good that the return value is immutable too.
  ```javascript 
// impure
function applySort(arr) {
  return arr.sort(); 
}

// externally pure
function sort(arr) {
  return applySort([...arr]);
}

// externally pure
function createMapById(list) {
  const map = Object.create(null);
  list.forEach(function addToMap(item) {
    map[item.id] = item;
  });
  return Object.freeze(map);
}

const vatTax = Object.freeze({ es: 21 }); // free variable: scalar or immutable

// pure
function getPrice(price, country) {
  return price * (1 + vatTax[country] / 100);
}

```

A more appropiate name for external purity is referential transparency.

## Referential transparency 

In the previous section we have defined what a pure function is by negation, mostly identifing the side-effects that prevents a function from being pure. But the formal and affirmative definition of a pure function comes from the referential transparency. A pure function is a function that has referential transparency.

Referencial transparency, also called idempotence, exists when there exists a pure mapping between the function arguments and its return value. 

This implies all the majesty of *sustituability*: we can replace a pure function with its output value and the application will keeping doing exactly the same. 

With substituability comes easier equational reasoning, rewrittability and memoization.
These are the causes of side-effects but most of the times they concur as a mix of them. Javascript deliberated flexibility and ubiquitous vocation makes effectful computations and impurity a great deal of our codebases.
