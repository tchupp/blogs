# Deploying vs Releasing

### Original Code

```typescript
function doStuff(param1: string, param2: number) {
  return `${param1}-1`;
}
```

### New Code

```typescript
const myAwesomeFeatureFlag = false;

// rename the original implementation to this
function doStuff1(param1: string, param2: number) {
  return `${param1}-1`;
}

// add new functionality
function doStuff2(param1: string, param2: number) {
  return `${param1}-${param2}`;
}

function doStuff(param1: string, param2: number) {
  // if the feature flag is true, use the new code
  if (myAwesomeFeatureFlag) {
    return doStuff2(param1, param2);
  }

  // otherwise, default to the original implementation
  return doStuff1(param1, param2);
}
```
