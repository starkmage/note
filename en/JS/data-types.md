## The 7th Basic Data Type — BigInt

### What is BigInt?

> BigInt is a new data type that is used when integer values are larger than the range supported by the Number data type. This data type allows us to safely perform arithmetic operations on `large integers`, represent high-resolution timestamps, use large integer IDs, etc., without needing libraries.

### Why do we need BigInt?

In JS, all numbers are represented in double-precision 64-bit floating-point format, so what problem does this cause?

This means that JS's Number cannot precisely represent very large integers; it will round very large integers. Specifically, **the Number type in JS can only safely represent integers between -9007199254740991 (-(2^53-1)) and 9007199254740991 ((2^53-1))**, any integer values outside this range may lose precision.

```js
console.log(999999999999999);  //=>10000000000000000
```

There are also some security issues:

```js
9007199254740992 === 9007199254740993;    // → true Actually true!
```

### How to create and use BigInt?

To create a BigInt, simply append n to the end of the number.

```js
console.log( 9007199254740995n );    // → 9007199254740995n	
console.log( 9007199254740995 );     // → 9007199254740996
```

Another way to create a BigInt is to use the BigInt() constructor.

```js
BigInt("9007199254740995");    // → 9007199254740995n
```

Simple usage is as follows:

```js
10n + 20n;    // → 30n	
10n - 20n;    // → -10n	
+10n;         // → TypeError: Cannot convert a BigInt value to a number	
-10n;         // → -10n	
10n * 20n;    // → 200n	
20n / 10n;    // → 2n	
23n % 10n;    // → 3n	
10n ** 3n;    // → 1000n	

let x = 10n;	
++x;          // → 11n	
--x;          // → 9n
console.log(typeof x);   //"bigint"
```

### Points to be cautious about

1. BigInt does not support the unary plus operator, which may be because some programs might rely on + always generating a Number, or throwing an exception. Additionally, changing the behavior of + would break asm.js code.
2. Mixed operations between BigInt and Number are not allowed because implicit type conversion might lose information. When mixing BigInts and floating-point numbers, the resulting value might not be precisely representable by either BigInt or Number.

```js
10 + 10n;    // → TypeError
```

3. BigInt cannot be passed to Web APIs and built-in JS functions that require a Number. Attempting to do so will result in a TypeError.

```js
Math.max(2n, 4n, 6n);    // → TypeError
```

4. When Boolean type meets BigInt type, BigInt is treated similarly to Number. In other words, as long as it's not 0n, BigInt is considered a true value.

```js
if(0n){//condition evaluates to false

}
if(3n){//condition evaluates to true

}
```

5. Arrays with all elements as BigInt can be sorted normally.

6. BigInt can perform bitwise operations normally, such as |, &, <<, >>, and ^.

### Reference Article:

[Understanding BigInt](http://47.98.159.95/my_blog/js-base/007.html)

## ['1', '2', '3'].map(parseInt) Return Value

> First, the return value is: [1, NaN, NaN]

The function passed to map has 3 parameters: value, index, arr, while parseInt has two parameters:

> parseInt(string, radix);

```
string
The value to be parsed. If the parameter is not a string, it will be converted to a string (using the toString abstract operation). Leading whitespace in the string will be ignored.
```