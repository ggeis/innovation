# Functions
A function is any callable expression that can be evaluated by applying the `()` operator to it. Functions in Javascript have the two important characteristics that make the functional style possible:

- *First class*: Functions are actual objects so they behave like "yet-to-be-executed" special values. But values after all. Functions can be even instantiated via constructors which is proof o their first-class nature and their underlying object prototype design:
```typescript   
    function() {
      const localizeDate = new Function(
        'date',
        'locale',
        'const date_ = new Date(date || Date.now()); ' +
          'const locale_ = locale || Intl.DateTimeFormat().resolvedOptions().locale; ' +
          'return date_.toLocaleDateString(locale_);'
      );
      return localizeDate();
    }
```

- *Higher-order*: The first-class feature enables that functions can not only be assigned to variables but therefore used as parameters and returning values of other functions. A higher-order function is a function that takes functions as arguments and/or returns other functions. This functions composability capacity is the foundation of functional programming.
 
 As an extra, which sometimes can be a liability, there are different implicit types of function invocations depending on where the function is declared and where it is invoked: global function, method of an object, constructor function... To ease this, `Function.prototype.apply` and `Function.prototype.call` enable the overriding of the function original runtime context (`this`) adherence. Functional programming might use those prototypes, but mostly with `null` as the first argument, to explicitly commit to purity.
```typescript   
    function() {
      const contraryFn = (fn) => (...args) => !fn.call(null, ...args);
      const isToday = (date) => date.equals(Temporal.now.plainDateISO());
      const isNotToday = contraryFn(isToday);
      const today = Temporal.now.plainDateISO();
      const tomorrow = today.add({ days: 1 });
      assert(isToday(today) === true);
      assert(isNotToday(today) === false);
      assert(isToday(tomorrow) === false);
      assert(isNotToday(tomorrow) === true);
    }
```
The higher-order function is in the first line. `contraryFn` accepts a function as an argument and returns back the same function but wrapped in a negation expression. As a generalized function the returned function accepts any list of arguments thanks to the spread operator(`...args`), it is completely agnostic regarding the number of them which makes it a *variadic* function (or *arity N*).
 
 As `isToday` is a *lambda expression*, for the case at hand we could pass any runtime context via call or apply to it and will be completely ignored anyways. 
 
 Born from functional programming, lambda expressions, also informally called arrow functions, encode anonymous functions into a shorter syntax and at the same time deflect any runtime context injection attempt. You can have multiple lines in those function's bodies, but one-liners are the most commonly used. That's because lambda expressions are especially suited as the parameters-in for higher-order functions, sometimes on the fly at the parameter declaration without further ceremony. They bring quick and local logic. That doesn't mean it's always easy to grasp, what it means is that lambda expressions always provide algebraic reasoning and referential transparency. Therefore, `contraryFn = (fn) => (...args) => !fn(...args)` would be a more succinct declaration that will work just fine for the sample.
 
 ## Function composition
 Generally speaking, composition occurs when data combines to make like-data or data of the same type; it preserves type. Objects fuse into new objects, and functions combine to create new functions. When using mixins, for example, to create new objects, this process is known as structural composition and was covered in the previous chapter. This section reviews how to assemble code at the function level, known as low-level composition. Function composition is the backbone of functional programming, and it’s the guiding principle by which we can arrange and assemble our entire code.
 ```typescript
 function() {
  const localizeDate = (date, locale) => date.toLocaleString(locale);
  const localizeUK = (date) => localizeDate(date, 'en-UK');
  const wholeHour = (date) => date.round({ smallestUnit: 'hour', roundingMode: 'floor' });

  const date = Temporal.now.plainDateTimeISO();
  return localizeUK(wholeHour(date));
}
```
So given the functions `localizeUK` and `wholeHour`, we can order them in such a way that the output of the first becomes the input of the second. The basic two function composition can be represented by `f(g(args))`. Javascript follows *eager evaluation* of expressions, so `wholeHour` is executed first and `localizeUK` after. This is the exact reverse with how we would express that in natural language: '*given a date, then round-down the hour, then format for UK*', but functional programming also provides some techniques to resemble a more fluent function chain or cascade if we prefer that. More on that later.

Let's say this two function plumb is used in different places of the codebase and we want to express it as a further simplified operation. We will use a simple composition higher-order function. `wholeHour` takes and returns a new date value, `localizeUK` takes a date value and returns a string value; but `compose` will take two function values and return a function value.
```typescript
function() {
  const compose = (f, g) => (...args) => f(g(...args));

  const localizeDate = (date, locale) => date.toLocaleString(locale);
  const localizeUK = (date) => localizeDate(date, 'en-UK');
  const wholeHour = (date) => date.round({ smallestUnit: 'hour', roundingMode: 'floor' });
  const uKHour = compose(localizeUK, wholeHour);

  const date = Temporal.now.plainDateTimeISO();
  assert(uKHour(date) === localizeUK(wholeHour(date)));
}
```
We could argue that
```typescript   
const uKHour = (date) => localizeUk(wholeHour(date));
```
would do the same and removing the `compose` dependency. And that is completely right, but we would then be unnecessarily "massaging" the data (notice how there is no reference to `date` in the original `uKHour`) and slightly defocusing from the expression of the pure behavior we are looking for. With `compose` we are separating the combined functions declaration from the strict evaluation of them. Also, as more complex use cases come for, especially working with list and graphs, or as soon as we are bringing in other functional structures, composing higher-order functions can do more for us.
```typescript   
const toUpperCase = (str) => str.toUpperCase();
const upperCaseUKHour = compose(toUpperCase, uKHour);
```
We can recompose functions too.
Or it might be the case that we want to declare `upperCaseUKHour` straight from the beginning, then the original `compose` function falls a bit short for the use case, but we can still pass it to another more complete composition function able to take any number of functions:
```typescript   
const multipleCompose = (...fns) => fns.reduce(compose);
const upperCaseUKHour = multipleCompose(toUpperCase, localizeUK, wholeHour);
```
`Array.prototype.reduce` within `multipleCompose` will keep composing each new function element of the argument array to the composing accumulator, keeping the same order than `compose` expects.
From now on, any reference to `compose` will refer to the more complete `multipleCompose` variadic version.
## Currying
Currying is a technique that will help you compose functions when they require multiple arguments. Currying is the modification of a function expecting n arguments into a series of n functions each expecting one argument.
```typescript   
const localizeDate = (date, locale) => date.toLocaleString(locale);
``` 
is curryfied either to
```typescript   
const localizeDate = date => locale => date.toLocaleString(locale);
```
or
```typescript   
const localizeDate = locale => date => date.toLocaleString(locale);
```
The structure of the curried function is very straightforward thanks to how Javascript *closures* backpack the arguments for usage to all the nested functions within its scope. This allows creating (or importing from a fp library) a higher-order function that can auto-curry functions.
```typescript
const curry = (fn) => (...args1) =>
  args1.length === fn.length
    ? fn(...args1)
    : (...args2) => {
        const args = [...args1, ...args2];
        return args.length >= fn.length ? fn(...args) : curry(fn)(...args);
      };

const localizeDate = curry((locale, date) => date.toLocaleString(locale));
```
`curry` mandates that you satisfy all the arguments of a function before it evaluates. Until that happens, curry keeps returning the "meaningless" *partially applied* functions with the remaining arguments waiting to be passed in, from first to last. This *discretization* delivers *lazy evaluation*, as until all arguments are provided the curried function won't be evaluated. With curried functions, the order of arguments is important as well. Normally, we don’t pay much attention to the order in object-oriented or imperative doctrines. But in FP, argument order is crucial because it relies so much on partial application. We have to think beforehand how arguments should be arranged to benefit from partial evaluations.

`curry` and `compose` are *combinator* functions (generic higher-order functions without any special bussiness logic on their own) and make for a common and powerful duo:
```typescript   
function() {
  const localizeDate = curry((locale, date) => date.toLocaleString(locale));
  const wholeHour = (date) => date.round({ smallestUnit: 'hour', roundingMode: 'floor' });
  const uKHour = compose(localizeDate('en-UK'), wholeHour);
  const date = Temporal.now.plainDateTimeISO();
  return uKHour(date);
}
```
