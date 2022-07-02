# setState
This action changes the value of an existing state or of part of a state. Its properties must respect the following structure:

```typescript
interface SetStateProperties {
  path: string,
  value: any,
}
```

Where:
#### `path`
This is a string identifying which state or part of a state to alter. A path must use `.` to specify a property to change. Example:
`user.name.first`. The string before the first `.` identifies the state, while the rest of the string tells us which property of the state
we want to change. To change the entire state, you can just use the state name as the path.

When the path identifies a state that doesn't exist, nothing is changed. When the state exists but the property doesn't, the property is created.

In cases where part of the path exists, but it is not an object, the original value is lost and replaced by an object. For instance, if the
path is `user.name.first`, but `user.name` already has a value and it is a string, then, the string value is lost and replaced by an object.

Attention: we can't use this action to set a specific position in an array, so `[` and `]` are not valid.

#### `value`
This is the new value for the state specified by `path`. This can be of any type. 

## Examples
Suppose a state called `user` with the value `{ names: { first: 'John', last: 'Smith' }, permissions: [0, 1, 2] }`. Then we can:

#### Change the first name to Joseph:
```json
{
  "_:action": "setState",
  "path": "user.name.first",
  "value": "Joseph"
}
```
**Result**: `{ names: { first: 'Joseph', last: 'Smith' }, permissions: [0, 1, 2] }`

#### Change the entire name to Mary Stone and add a nickname ("MS"):
```json
{
  "_:action": "setState",
  "properties": {
    "path": "user.name",
    "value": {
      "first": "Mary",
      "last": "Stone",
      "nick": "MS"
    }
  }
}
```
**Result**: `{ names: { first: 'Mary', last: 'Stone', nick: 'MS' }, permissions: [0, 1, 2] }`

#### Add the property "age" and set it to `26`:
```json
{
  "_:action": "setState",
  "properties": {
    "path": "user.age",
    "value": 26
  }
}
```
**Result**: `{ names: { first: 'John', last: 'Smith' }, permissions: [0, 1, 2], age: 26 }`

#### Change the array of permissions to `[0, 10, 2]`:
```json
{
  "_:action": "setState",
  "properties": {
    "path": "user.permissions",
    "value": [0, 10, 2]
  }
}
```
**Result**: `{ names: { first: 'John', last: 'Smith' }, permissions: [0, 10, 2] }`

#### Change the structure of the property "permissions" by setting `permissions.read` to `true`:
```json
{
  "_:action": "setState",
  "properties": {
    "path": "user.permissions.read",
    "value": true
  }
}
```
**Result**: `{ names: { first: 'John', last: 'Smith' }, permissions: { read: true } }`

#### Change the entire state to a new value:
```json
{
  "_:action": "setState",
  "properties": {
    "path": "user",
    "value": {
      "name": {
        "first": "Mary",
        "last": "Stone",
        "nickname": "MS"
      },
      "permissions": {
        "read": true,
        "write": true,
        "admin": false
      },
      "age": 26
    }
  }
}
```
**Result**: `{ names: { first: 'Mary', last: 'Stone', nick: 'MS' }, permissions: { read: true, write: true, admin: false }, age: 26 }`