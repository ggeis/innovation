# Objects
Aside from a few primitives, everything else in Javascript is an object. So before going deeper on the functional side, it's crucial that we analyze all the details of this essential type.
```typescript   
function () {
  return typeof Function.prototype.__proto__; // object
};
```
*Functions are objects too*
## Literal

There are many idiomatic ways of declaring objects. The simplest one is the object literal declaration, but we should restrict its use to very few scenarios really, a very common usage is the *transfer object* pattern when you need to group some data to pass to another function and immediately destructure it on the receiving function. This allows for easier grouped, free-style ordering, argument interface. Another possible use case is for storing a shallow, normalized, and immutable global object variable. Using the literal for more than this and other very limited scopes and lifecycles can generate lots of unnecessary complexity as your codebase scales.
```typescript   
function () {
  const now = { 
    locale: Intl.DateTimeFormat().resolvedOptions().locale, 
    date: new Date(Date.now()) 
  };
  const localizeDate = ({locale, date}) => { 
    return date.toLocaleString(locale);
  } 
  return localizeDate(now);
};
```
Note how the `now` object itself and the expressions wired with its properties (`locale` and `date`) are executed once and neither deferred nor arranged in advance. This is a limitation of the literal declaration.
## Literal with interface
Also, we can expose some utility methods that will have the literal as the owner of the execution context
```javascript
function () {
  const now = {
    locale: Intl.DateTimeFormat().resolvedOptions().locale,
    date: new Date(Date.now()),
    localize(locale_) {
      return this.date.toLocaleString(locale_ || this.locale);
    }      
  };
  const localizedNow = now.localize();
  return localizedNow;
};
```
*But, do really `localize` and `locale` belong to something as specific as `now`?*

Here we are somehow pulling apart from functional programming, we got rid of the generalized function `localizeDate` in favor of the tightly coupled `localize` invokable method. The `now` literal might now resemble remotely a data structure. Functional programming must necessarily blend with data structures, particularly with the very low level and others that can take advantage of side effects and mutations to improve performance and reduce the computational complexity (*O-notation*) of some operations. Thankfully, properly designed data structures implement a *single designated type* and provide a contract of operations transparent for the functional workflow consumption.

This is not the case for the `now` object in many levels: it doesn't try to overcome any pure efficiency problem, it exposes two types and the implicit contract logic can be circumvented by laissez-faire calls to them:
```javascript
function () {
  const now = {
    locale: Intl.DateTimeFormat().resolvedOptions().locale,
    date: new Date(Date.now()),
    localize(locale_) {
      return this.date.toLocaleString(locale_ || this.locale);
    }      
  };
  now.date.setFullYear(2000);
  const localizedNow = now.localize();
  return localizedNow;
};
```
As hiring interviews demonstrate, there is a need for data structures and intensive computing in applications, and every programmer should be up-to-date if not mastering them, but the leading cause of problems in big applications is accidental and most of the times related to the state management dynamics... because Javascript is terrible when it comes to securing an object state.

| FP appeal         | DS appeal        |
|-------------------|------------------|
|                   | Computationally intense data operations |
| State management  |                  |
| Control flow and declarativeness      |                  |
| Thread safety     |                  |
| Encapsulation     |                  |

We should not confuse DS algorithmics with the object-oriented paradigm and some of its patterns. OOP spreads tight coupling and contracts to *every* entity in the model, much beyond the logarithmic performance pursuing of DS. OOP promotes specialized types as the unit of composition, which may lead to efferent coupling (a type depending on many others), abstractness, `super` chains, "explosion" of classes, cyclomatic complexity on back and forth conversions, etc. We can argue that Javascript is essentially suited for FP, but both angles are valid. Even some admixture is possible as long as we are aware of the possible incompatibilities of, in the end, two general ways of organizing and composing code. FP does function composition, generalized coarse-grained polymorphic functions that crosscut across data types. OPP does object composition, where each specialized type encapsulates fine-grained coupling between data and behavior via methods.
