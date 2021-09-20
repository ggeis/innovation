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
