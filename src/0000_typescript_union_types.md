# Typescript Union Types

Typescript has some pretty awesome support for union types:

```typescript
type Id = string | number;
```

This can useful if you want to make the input to functions flexible without making them `object` or `any`:

```typescript
// Instead of using `any`...
// function join(source: string[], separator: any): string

// ...give the user the option to use a string OR a number
function join(source: string[], separator: string | number): string {
  return "";
}

```


but if you want to give your types super powers then you have to use a "tag" to [discriminate the union](https://www.typescriptlang.org/docs/handbook/unions-and-intersections.html#discriminating-unions)

```typescript
type Message = 
  | { _type: "MessageA", a: number }
  | { _type: "MessageB", b: string }
  ;

function blahblah(message: IoTMessage) {
  // console.log(message.a); // <- doesn't compile, we don't KNOW that the type has a property `.a`
  switch (message._type) {
    case "MessageA":
      console.log(message.a);
      break;
    case "MessageB":
      console.log(message.b);
      break;
    // no `default` needed, since the compiler knows that `"MessageA"` and `"MessageB"` are the only possibilities
  }
}
```

## Errors

```typescript
type FlowSpecificErrors = 
  | { _type: "RedisConnectionError", host: string, port: number, operation: string }
  | { _type: "SpecificExternalApiStatusCodeError", operation: string, status_code: number, response: string }
  ;
```
