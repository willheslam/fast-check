[Home](/) &gt; [fast-check](../fast-check.md) &gt; [set](set_2.md)

## set() function

For arrays of unique values coming from `arb` having lower and upper bound size

<b>Signature:</b>

```typescript
declare function set<T>(arb: Arbitrary<T>, minLength: number, maxLength: number): Arbitrary<T[]>;
```

#### Parameters

|  Parameter | Type | Description |
|  --- | --- | --- |
|  arb | <code>Arbitrary&lt;T&gt;</code> |  |
|  minLength | <code>number</code> |  |
|  maxLength | <code>number</code> |  |

<b>Returns:</b>

`Arbitrary<T[]>`

