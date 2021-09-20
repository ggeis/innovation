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

The optimal functional approach to the problem (i.e., obtain a localized now) would be abstaining completely from using an object, and rely on independent immutable values (`Temporal` is ECMA Stage-2 proposal) and generalized arrow function:

```javascript
import { Temporal } from 'proposal-temporal';
const date: Temporal.PlainDateTime = Temporal.now.plainDateTimeISO();
const { locale } = Intl.DateTimeFormat().resolvedOptions();
const localizeDate = (date_: Temporal.PlainDateTime, locale_: string): string =>
  date_.toLocaleString(locale_);
const localizedNow = localizeDate(date, locale);
```
*`Temporal` to "replace" the flawed built-in `Date` (and the third-party libraries to handle it) with a immutable type*

## Precise creation
But as the point of this chapter is reviewing objects, let's refine the previous implementation with the convenient an fully functional `Object` API:
```javascript
function() {
  const now = Object.create(
    Object.prototype,
    (() => {
      const date = new Date(Date.now());
      return {
        date: {
          value: new Date(date),
          enumerable: true,
          writable: false,
          configurable: false,
        },
        locale: {
          value: Intl.DateTimeFormat().resolvedOptions().locale,
          enumerable: true,
          writable: true,
          configurable: false
        },
        localize: {
          value: function (locale_) {
            return date.toLocaleString(locale_ || this.locale);
          },
          enumerable: true,
          writable: false,
          configurable: false
        }
      };
    })()
  );
  const localizedNow = now.localize();
  return localizedNow;
}
```
Using `Object.create(proto,[propertiesObject])` second parameter we obtain much more control on the properties behavior. In this case, with the `writable` data descriptor we protect `date` from hindered rewrites. `configurable` prevents deletion which makes for a more bulletproof object state in its journey through the codebase.

Be aware that data descriptor `value` is not a *getter* accessor function, it's not executed every time the property is accessed but only once. Data accessors (getters/setters) are also available in the `propertiesObject` parameter, but they are mutually exclusive with `value` and `writable`. 

Data descriptors are not enough as Date is not immutable, we also need to use other functional tools like *IIFE* (Immediately Invoked Function Expression) and *closures* to encapsulate the type and expose only a copy of it for potential external parallelizations. 

Recreating objects and making copies for the sake of immutability is a common practice in FP, this can bring some conceptual misunderstandings or inefficiencies when testing equalities. Not even shallow value comparisons are not supported by Javascript runtime (Javascript is *duck-typed*, so it's inherently open-to developer discretion if two objects that share the same properties and values but have a different prototype, for example, should be considered equal or not. The application developer should define the objects equality rules). FP benefits not only from immutability, but also when simple and shallow objects are in charge of storing data.

Until this point, we have been reviewing the ways of creating a solitary object, but as we start using multiple objects that share a shape, and we want to reuse the code while modeling relationships, we need more than this.

## Inheritance
Inheritance is the bread-and-butter pattern for prototyping type hierarchies. Javascript implements it with one of the many applications of prototypes. Let's set up a basic prototype chain.

This time, we're going to use the first argument of `Object.create` with something different than the native `Object.prototype`:
```typescript
function() {
  const moment = Object.create(Object.prototype, {
    date: { 
      value: Temporal.now.plainDateTimeISO(), 
      writable: true,
      enumerable: true 
    }
  });
  const localizedMoment = Object.create(moment, {
    locale: {
      value: Intl.DateTimeFormat().resolvedOptions().locale,
      writable: true,
      enumerable: true
    },
    localizedDate: {
      get() {
        return this.date.toLocaleString(this.locale);
      },
      enumerable: true
    }
  });
  moment.date = moment.date.add({days: 5});
  assert(localizedMoment.date === moment.date);
  return localizedMoment.localizedDate;
}
```
`localizedMoment` is inheriting from `moment`, we have just created a prototypal link between the two objects. Any properties of the parent (hereafter called the *prototype*) are accessible from the child object. An object can only have a single prototype, but they can be chained down. We will have access to every ancestors properties via *dot notation* only (`Object.keys` and similar will traverse the object own properties but not the ones coming from the prototype). Note how `moment.date` and `localizedMoment.date` point to the same exact reference. Even if `moment` is used as a go-in-between facilitator, `moment` is a object and can be manipulated as such, its not like a kind of *abstract class*, or something created from thin air at the point of forming the inheritance association.

`localizedMoment.localizedDate` seems a bit redundant so let's do some naming that will serve us to explain the prototype lookup mechanism:
```typescript
function() {
  const moment = Object.create(Object.prototype, {
    date: {
      value: Temporal.now.plainDateTimeISO(),
      enumerable: true
    }
  });
  const localizedMoment = Object.create(moment, {
    locale: {
      value: Intl.DateTimeFormat().resolvedOptions().locale,
      enumerable: true
    },
    date: {
      get() {
        return Object.getPrototypeOf(this).date.toLocaleString(this.locale);
      },
      enumerable: true
    }
  });
  return localizedMoment.date;
}
```
As we can see now we have two `date` properties, the one from the child and the one from the prototype, that might be colliding because they have the same exact concise name but a different reference an even type (`Temporal.PlainDateTime` vs `string`). This is not the case in Javascript as the prototype resolution mechanism flows up from the calling object to its linked object (and so on), allowing that any newly derived object can differentiate itself from its parent with new behavior. New behavior includes adding new properties or, like in this example, overriding an existing property from the linked object. This pattern is know as *shadowing* a property. 
So any dot-notation access to `localizedMoment.date` will be satisfied with the first `date` it encounters in a bottom-up traversal of the prototype chain.

Using developer tools we can clearly see how the inheritance is implemented as a prototype chain, with native Object at the bottom, which is the top from a hierarchical perspective.

Also we still have access to the `moment` and `date` through the prototype as the getter exemplifies, so we are shadowing and neither overriding nor overwriting.

Object association through the prototypes is completely different from the inheritance implementation in OOP languages, in which instances gain copies of the inherited data. In Javascript it's links not copies.

## Constructor functions

Up to this point we have been creating very inelastic objects regarding its data. We can use parametrizable functions to initialize objects populated with different data values, these *constructor functions* can also encapsulate the enforcement of some preconditions.

```typescript
function() {
  function Moment(date) {
    if (!new.target) new Moment(date);
    this.date = date || Temporal.now.plainDateTimeISO();
  }

  function LocalizedMoment(date, locale) {
    if (!new.target) new LocalizedMoment(date, locale);
    Object.assign(this, new Moment(date));
    this.locale = locale || Intl.DateTimeFormat().resolvedOptions().locale;
  }

  Object.setPrototypeOf(LocalizedMoment.prototype, Moment.prototype);

  LocalizedMoment.prototype.localize = function (locale) {
    return this.date.toLocaleString(locale || this.locale);
  };

  assert(Moment.prototype.constructor === Moment);
  assert(Moment.prototype.__proto__.constructor === Object);
  assert(LocalizedMoment.prototype.constructor === LocalizedMoment);
  assert(LocalizedMoment.prototype.__proto__.constructor === Moment);

  const now = new LocalizedMoment(Temporal.now.plainDateTimeISO(), 'en-UK');
  return now.localize('es-ES');
}
```
By convention, it's recommended that a constructor function name is capitalized. It's important that we safeguard the usage of `new` (even if the caller forgets to) for the runtime to implicity point the `this` context of the constructor function to the newly created object. Notice how we also had to set the prototype link to state the parent-child relationship with `Object.setPrototypeOf`.

 Also `localize()` is declared outside of the constructor function because, as the code is shared among, it is more efficient to bound it to the `prototype` and not allocating a new function copy for every instance.
 ## Classes
 There is no dichotomy between classes and prototypes in Javascript. But as prototypes sometimes can be dull boilerplates, Javascript provides the more syntactically clean classes to rough the edges of constructor functions.
 ```typescript
 function() {
  class Moment {
    date = Temporal.now.plainDateTimeISO();
    constructor(date = this.date) {
      this.date = date;
    }
  }

  class LocalizedMoment extends Moment {
    locale = Intl.DateTimeFormat().resolvedOptions().locale;
    constructor(date, locale) {
      super(date);
      this.locale = locale;
    }
    localize(locale) {
      return this.date.toLocaleString(locale ?? this.locale);
    }
    get date() {
      return super.date.toLocaleString(this.locale);
    }
  }

  const now = new LocalizedMoment(Temporal.now.plainDateTimeISO(), 'en-UK');
  return now.date;
}
 ```
Classes also provide another useful perks like private members, default values, static functions, etc., at the expanse of an foreign mental model. In short, classes are syntactic sugar that abstracts away the mundane prototype scaffolding and are very helpful as long as you keep in mind what is going on beyond the surface and don't overcommit to the tightly-coupled and rigid architecture that inheritance might lean to.
## OLOO (Objects Linked to Other Objects)
This pattern still uses implicit prototype linking, in which object and proto life cycles are interwined, but doesn't employ neither classes or prototype semantics but `Object.create`.

```typescript
function() {
  const Moment = Object.create(Object.prototype, {
    date: { enumerable: true, writable: true },
    init: {
      value: function (date) {
        this.date = date || Temporal.now.plainDateTimeISO();
      },
      enumerable: true
    }
  });

  const LocalizedMoment = Object.create(Moment, {
    locale: { enumerable: true, writable: true },
    init: {
      value: function (date, locale) {
        Moment.init.call(this, date);
        this.locale = locale || Intl.DateTimeFormat().resolvedOptions().locale;
        return new Proxy(this, {
          get(target, key) {
            return key === 'date'
              ? target.date.toLocaleString(target.locale)
              : target[key];
          }
        });
      },
      enumerable: true
    }
  });
  const localizedNow = Object.create(LocalizedMoment);
  return localizedNow.init({ locale: 'en-UK' }).date;   
}
```
With OLOO we just do abandon the inheritance terminology. First, notice how the properties are created empty and not initialized but on-demand, or lazily, which means object properties are being carried on but as an empty shell, it is in the init() initialization implementations, which are a requirement of the pattern, where data is populated when needed. In the code we are also adding a Proxy to shadow `date`, this is not part of the OLOO pattern so we can ignore it until we will review Proxies on a subsequent chapter.
## Mixins
The idea of behind mixins, composing or assembling objects, is different from the approaches we've seen so far. With prototypes, we share state and behavior using chains of links of single objects, but with mixins, we are going to copy fine-grained pieces of multiple independent slices of objects that together represent the entirety of the new object’s API. Mixins by themselves don't provide much value on their own, but they are used to compose the business objects. Object composition promotes creating *has-a* or *uses-a* rather than *is-a* relationships among objects. So instead of implicitly delegating to an object’s parent, we completely circumvent inheritance and copy all the properties we need from different sources and directly to the target object.
```typescript
function() {
  const hasLocale = (locale) => ({
    locale: locale || Intl.DateTimeFormat().resolvedOptions().locale
  });

  const hasDate = (date) => ({
    date: date || Temporal.now.plainDateTimeISO()
  });

  const LocalizedMoment = function (date, locale) {
    if (!new.target) return new LocalizedMoment(date, locale);
    return new Proxy(Object.assign({}, hasLocale(locale), hasDate(date)), {
      get: (target, key) => {
        if (key === 'date') return target.date.toLocaleString(target.locale);
        return target[key];
      },
      getOwnPropertyDescriptor: function (target, key) {
        switch (key) {
          case 'locale':
            return { configurable: true, writable: true, enumerable: true };
          case 'date':
            return { writable: false, enumerable: true };
          default:
            return Object.getOwnPropertyDescriptor(target, key);
        }
      },
      deleteProperty: function (target, key) {
        if (key === 'locale' || key === 'date') {
          return false;
        }
        return delete target[key];
      }
    });
  };

  const localizedMoment = new LocalizedMoment();
  return localizedMoment.date;
};
```
`hasLocale` and `hasDate` are the mixins, in this case functions that return simple objects without further context, `Object.assign` in `LocalizedMoment` constructor function does all the gluing and linearization. This is a possible functional application of mixins, but you can do others like for instance extend a prototype or using them with a more OOP approach with classes. Adoption of mixins will bring more flexibility in modeling your objects, as objects won't inherit from each other but associate freely.
