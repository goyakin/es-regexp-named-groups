# RegExp Named Capture Groups

## Introduction

Numbered capture groups allow one to refer to certain portions of a string that a regular expression matches. Each capture group is assigned a unique number and can be referenced using that number, but this can make a regular expression hard to grasp and refactor.

For example, given `/(\d{4})-(\d{2})-(\d{2})/` that matches a date, one cannot be sure which group corresponds to the month and which one is the day without examining the surrounding code. Also, if one wants to swap the order of the month and the day, the group references should also be updated.

Named capture groups provide a nice solution for these issues.

## High Level API

A capture group can be given a name using the `(?<name>...)` syntax. The regular expression for a date then can be written as `/(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/`. Each name should be unique and follow the grammar for ECMAScript identifiers.

Named groups can be accessed from the regular expression result using the `groups` property. This property would also allow numbered references to the groups. For example:

```js
let re = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
let result = re.exec('2015-01-02');
if (result) {
    // result.groups.year === '2015';
    // result.groups.month === '01';
    // result.groups.day === '02';

    // result.groups[0] === '2015-01-02';
    // result.groups[1] === '2015';
    // result.groups[2] === '01';
    // result.groups[3] === '02';
}
```

A named group can be accessed within a regular expression via the `\k<name>` construct. For example, `/(?<asterisk>\*).\k<asterisk>/` would match `'*a*'`. Named references can also be used simultaneously with numbered references. For example, `/(?<asterisk>\*).\k<asterisk>.\1/` would match `'*a*a*'`.

Named groups can be referenced from the replacement value passed to `RegExp.prototype[@@replace]` too. If the value is a string, named groups can be accessed using the `${name}` syntax. For example:

 ```js
let re = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
let result = re[Symbol.replace]('2015-01-02', '${day}/${month}/${year}');
// result === '02/01/2015'
```

If the value is a function, then the named groups can be accessed via a new parameter called `groups`. The new signature would be `function (matched, capture1, ..., captureN, position, S, groups)`.

## Open Questions

### How to Return Named Groups

Currently, the proposed way to accessed named groups via the regular expression result is via a new property called `groups`. Another option is to allow accessing groups directly from the result, for example:

 ```js
let re = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
let result = re.exec('2015-01-02');
if (result) {
    // result.year === '2015'
    // result.month === '01'
    // result.day === '02'
}
 ```

However, given that the result already has non-numerical properties, e.g. `length` and `index`, named groups corresponding these properties would need to be disallowed.

It could be also possible to provide named groups both directly via the result and the new `groups` property. If a property with the same name as a group already exists on the result, that property would keep its original value. If access to the named groups is needed, it could be done via the `groups` property.
