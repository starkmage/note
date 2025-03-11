## Concept

`===` Strict equality, compares both the type and value of two values

`==` Abstract equality, when comparing, it first performs type conversion, then compares the values

## Situation Analysis

For `===`, the ECMA specification defines it as follows:

- If Type(x) and Type(y) are different, return false
- If Type(x) and Type(y) are the same
  - If Type(x) is Undefined, return true
  - If Type(x) is Null, return true
  - If Type(x) is String, return true if and only if x and y have exactly the same character sequence (same length and same characters at each position), otherwise return false
  - If Type(x) is Boolean, return true if x and y are both true or both false, otherwise return false
  - If Type(x) is Symbol, return true if x and y are the same Symbol value, otherwise return false
  - If Type(x) is Number type
    - If x is NaN, return false
    - If y is NaN, return false
    - If x's numeric value equals y, return true
    - If x is +0 and y is -0, return true
    - If x is -0 and y is +0, return true
    - Otherwise return false

For `==`, the ECMA specification defines it as follows:

- If Type(x) and Type(y) are the same, return the result of x===y
- If Type(x) and Type(y) are different
  - If x is null and y is undefined, return true
  - If x is undefined and y is null, return true
  - If Type(x) is Number and Type(y) is String, return the result of x==ToNumber(y)
  - If Type(x) is String and Type(y) is Number, return the result of ToNumber(x)==y
  - If Type(x) is Boolean, return the result of ToNumber(x)==y
  - If Type(y) is Boolean, return the result of x==ToNumber(y)
  - If Type(x) is one of String, Number, or Symbol and Type(y) is Object, return the result of x==ToPrimitive(y)
  - If Type(x) is Object and Type(y) is one of String, Number, or Symbol, return the result of ToPrimitive(x)==y
  - Otherwise return false

## Counter-intuitive Examples

```js
console.log('0' == '');     // false
console.log(0 == '');        // true
console.log(0 == '0');       // true
console.log('0' == []);      // false
console.log(0 == []);        // true
console.log(2 == [2]);       // true
console.log('2' == [2]);     // true
console.log(2 == {'0': 2});  // false
console.log('0' == false);   // true
console.log('false' == false);   // false
console.log(null == undefined);  // true
console.log(null == false);      // false
console.log(null == true);       // false
console.log(undefined == false); // false
console.log(true == 1);          // true
console.log(true == 2);          // false
console.log({a: 1} == "[object Object]");//true
console.log([] == false);        // true
console.log({} == false);        // false
console.log([] == ![]);          // true
console.log(![] == false);       // true
if ([]) console.log('yes')       // yes
if ([] == false) console.log(1)  // 1
if ([] == true) console.log(2)   // not printed
```

## How Does the Process of Converting Objects to Primitive Types Work?

When converting objects to primitive types, the built-in [ToPrimitive] function is called. For this function, the logic is as follows:

1. If Symbol.toPrimitive() method exists, call it first and return the result
2. Call valueOf(), if it converts to a primitive type, return that
3. Call toString(), if it converts to a primitive type, return that
4. If none of the above returns a primitive type, an error will be thrown

```js
var obj = {
  value: 3,
  valueOf() {
    return 4;
  },
  toString() {
    return '5'
  },
  [Symbol.toPrimitive]() {
    return 6
  }
}
console.log(obj + 1); // outputs 7
```

```js
let obj = {
  p: 'zzz'
}
console.log(obj.toString());
console.log(obj.valueOf());
console.log(typeof obj.valueOf());
```