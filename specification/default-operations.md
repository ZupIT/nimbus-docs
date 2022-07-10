# Operations shipped with Nimbus
The Nimbus specification provides a set of operations that must exist in any implementation. They are:

## Number
- `sum(...numbers)`: sums all parameters passed and returns the result. Example: `@{sum(1, 2.5, 3, 1)}` results in `7.5`.
- `subtract(...numbers)`: starts with the first parameter, then subtract each next parameter from it. Returns the result. Example: `@{sum(10, 5, 3)}` 
results in `2`.
- `multiply(...numbers)`: multiply every parameter with the next, the result is returned. Example: `@{sum(10, 5, 3)}` results in `2`.
- `divide(...numbers)`: divide every parameter by the next, the result is returned. Example: `@{divide(10, 2.5)}` results in `4`.
- `gt(left, right)`: returns true if the left number is greater than the right number. Example: `@{gt(10, 5)}` results in true while `@{gt(10, 10)}`
and `@{gt(10, 9)}` result in false.
- `gte(left, right)`: returns true if the left number is greater than or equal to the right number. Example: `@{gte(10, 5)}` and `@{gte(10, 10)}`
result in true while `@{gte(10, 9)}` results in false.
- `lt(left, right)`: returns true if the left number is less than the right number. Example: `@{lt(1, 5)}` results in true while `@{gt(1, 1)}`
and `@{lt(1, 0)}` result in false.
- `lte(left, right)`: returns true if the left number is less than or equal to the right number. Example: `@{lte(1, 5)}` and `@{lte(1, 1)}`
result in true while `@{lte(1, 0)}` results in false.

## String
- `capitalize(string)`: transform the first letter of the single parameter to an uppercase letter. Example: `@{capitalize('hello')}` results in
`'Hello'`.
- `uppercase(string)`: transform all letters of the single parameter to an uppercase letter. Example: `@{capitalize('hello')}` results in `'HELLO'`.
- `lowercase(string)`: transform all letters of the single parameter to a lowercase letter. Example: `@{lowercase('HELLO')}` results in `'hello'`.
- `substr(string, from [, length])`: returns a part of the string passed. This operation may have 2 or 3 arguments, where the first is the string, the second is the index where we want our substring to start and the third (optional) is the length of the substring. When the third parameter is not
supplied, the substring ends where the original string (first parameter) ends. Examples: `@{substr('hello world', 0, 5)}` results in `'hello'`;
`@{substr('hello world', 6)}` results in `world`.
- `replace(string, regex, replace)`: TODO
- `match(string, regex)`: TODO

## Array
Attention: in the descriptions below, an element is said to be equal to another if it has the same primitive value (boolean, string, number); or if
it's a map and has the same key-value pairs; or if it's an array and has the same elements, in the same order.

Attention: arrays in Nimbus are zero-based, i.e., their index (or position) starts at 0 (zero).

In the examples below, consider `ids` to be a state where the value is the array `[4, 20, 5]`.
- `insert(array, element [, index])`: inserts an element into the given array at the position given by the 3rd parameter. If the position (index) is
not provided, the element is inserted at the last position of the array. Example: `@{insert(ids, 2, 1)}` results in `[4, 2, 20, 5]`.
- `remove(array, element)`: if the array contains an element equal to the 2nd parameter, this element is removed. Example: `@{remove(ids, 20)}`
results in `[4, 5]`.
- `removeIndex(array [, index])`: removes the element at the given index. If no index is provided, removes the last element. Example:
`@{removeIndex(ids, 1)}` results in `[4, 5]`.
- `contains(array, element)`: 

## Logic
For the examples below, suppose the state `user` is a map where the key `age` is a number and the key `isEmancipated` is a boolean.
- `condition(premise, ifTrue, otherwise)`: returns the second parameter if the first is true; returns the third parameter if the first is false.
Example: `@{gte(user.age, 18), 'adult', 'minor'}` results in `'adult'` if `user.age` is 18 or more, otherwise results in `'minor'`.
- `not(value)`: returns true if the parameter is false and false if the parameter is true. Example: `@{not(gte(user.age, 18))}` results in true if
the `user.age` is less than 18 and false otherwise.
- `and(...values)`: returns true if every parameter is true. Example: `@{and(gte(user.age, 18), lt(user.age, 60))}` returns true if `user.age` is in
the interval [18, 60[ and false if it's not;
- `or(...values)`: returns true if at least one of the parameters is true. Example: `@{or(gte(user.age, 18), lt(user.age, 60))}` returns true despite
the value of `user.age` because every number satisfy at least one of the conditions. Another example: `@{or(gte(user.age, 18), user.isEmancipated)}`
results in true whenever `user.age` is greater or equal to 18 or `user.isEmancipated` is true. If both conditions are false, it results in false.

## Other
Suppose the state `user` with the properties `address = {}`, `age = 21`, `documents = []`, `names = { first: 'John', last: 'Smith', nick: '' }`,
`permissions: [0, 3, 6, 7]`.
- `isNull(value)`: `value` can be of any type, if it's null, this operation results in true, otherwise, it results in false. Examples:
`@{isNull(user.id)}` returns true, because when the state is not found, it's resolved to null. `@{isNull(address)}` and `@{isNull(documents)}` return
false.
- `isEmpty(value)`: `value` can be of any type, if it's an en empty string (`''`), an empty array (`[]`), an empty object (`{}`) or `null`, this
operation returns true, otherwise, it returns false. Examples: `@{isEmpty(user.id)}`, `@{isEmpty(user.address)}`, `@{isEmpty(user.documents)}` and
`@{isEmpty(user.names.nick)}` return true. `@{isEmpty(user)}` and `@{isEmpty(user.names)}` return false.
- `length(value)`: `value` can be of any type, if it's an array, this operation returns the number of items in the array; if it's a string, this
operation returns the number of characters in the string; if it's an object (map), it returns the number of entries in the map. In any other case, 
this operation returns 0 (zero). Examples: `@{length(user.id)}`, `@{length(user.age)}`, `@{length(user.address)}` and `@{length(user.documents)}`
return 0 (zero). `@{length(user.names)}` returns 3. `@{length(user.permissions)}` returns 4. `@{length(user.names.last)}` returns 5.
- `concat(...strings)`: concatenates each string parameter with the next. Example: `@{concat('hello', ' world')}` results in `'hello world'`. TODO: explain it now also works for arrays and maps.
- `contains(collection, element)`: returns `true` if the array contains an element equal to the second parameter. Returns `false` otherwise. Example:
`@{contains(ids, 10)}` results in false and `@{contains(ids, 4)}` results in true. TODO: explain it now works for Strings and Maps.
