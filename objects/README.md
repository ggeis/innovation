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
