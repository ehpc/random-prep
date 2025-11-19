# What is the difference between typescript void and undefined?

1. undefined is a real value

2. void is not a value — it’s a TypeScript-only return type

```js
function log(msg: string): void {
  console.log(msg);
}
```

At runtime, the function actually returns undefined, but TypeScript forbids 
you from using that return value.

You cannot assign a void result to a non-void variable.

You cannot expect void to carry information.

void is a type-level concept, not a runtime value.
