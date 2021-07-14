# ECMAProposal: Template Literals Juxtaposition and String.join

## The Syntax format

[Juxtaposition](https://en.wikipedia.org/wiki/Juxtaposition) is used in many programming languages like AWK, C, Python, and many FP languages.

1.  String juxtaposition in

    1.  C: `printf("Hello" "World");` will print `HelloWorld`
    2.  Python & Ruby & awk: `print('Hello' "World")` result in `'HelloWorld'`

2.  Juxtaposition in FP like

    1.  Scheme/Lisp: `(+ 1 2 3)`
    2.  OCaml: `sum 1 2 3`

All of the above **allow NEWLINE** between juxtaposition in addition to spaces.

This format has the simplicity and flexibility to make template literals more reasonable in ECMA Script in the following cases:

1.  RegExp
2.  Multiple line template literals

## High-level API

**Template literals juxtaposition**(**TLJ**) can be written as below:

```
var template = `This is `
    `in a `
    `line.`;

template === 'This is in a line.';
```

1.  The syntax uses juxtaposition for simplicity.
2.  TLJ will be joint together in parser, and converted into just one TemplateLiteral in runtime, this allows some transpilers to convert the syntax beforehand, allow more flexibility in authoring time.
3.  Each Template Literal item in **TLJ** is called `Template Literal Fragment` (**TLF**), so the whole is called TLJ, each part is called TLF.

The format also improved some experience in industrial production, like below:

### RegExp (Syntax 1: Plain TLJ)

Many languages have regular expression engines that support `Free-Spacing Mode`, which allow the author to break long regular expression into multiple lines. This mode [had been discussed](https://esdiscuss.org/topic/regexp-free-spacing-comments) but cannot be applied to javascript.

Nowadays the `String.raw` can be a powerful way to construct RegExp, with TLJ, we can write below:

```js
var re = new RegExp(String.raw
    `(\d{3}-)?` // area code (optional)
    `\d{3}-`    // prefix
    `\d{4}`     // line number
)
```

It is an alternate way to simulate the `Free-Spacing Mode` for RegExp.

The above code have no syntax errors, but have semantic errors, since `String.raw` doesn't return another tag function.

**This problem will be discussed later.**


### New line in TLJ (Syntax 2: Full TLJ)

Think about the below python code case using Syntax 1:

```
var pythonCode = `def add(a, b):\n` //def add
                 `  return a + b\n`
                 `print(add(1, 2))\n`;  // call and print add
```

**The problem here is the new-line** `\n` **char in each TLF, still ugly, this may lead to an advanced syntax (String.join):**

* Plain TLJ is the syntax sugar of the newly introduced `String.join()` function (Full TLJ), so below is same:

```js
`abc``def`
// ===
String.join()`abc```def`
```

* `String.join` can be used to generate a `TLJ joiner`, like above case:

```js
var pythonCode = String.join(index => '\n')
                 `def add(a, b):` //def add
                 `  return a + b`
                 `print(add(1, 2))`;  // call and print add

```

The `index` is index of TLJ element, the returned `'\n'` will be a string padder after current TLJ element.


### With placeholders

TLJ has the same behavior as the joint counterpart version, using **Syntax 2: Full TLJ** as below:

```
var s = `Hello: ${name}, ` // name
        `your email is: ${email}.`; // email

// same as below
var s = `Hello: ${name}, your email is: ${email}.`;
```

### Using variables in between TLJ

Since TLJ will be processed in parse time, it's not possible to place a dynamic variable in between TLJ directly, but it can be wrapped as a TLF:

```
var s = `Hello,`
        `${name}` // <---way of place the "name" variable
        `Welcome!`;
```

### Problems with tag function

Since Tagged Template Literals cannot be chained, so this syntax maybe break the template literal function that return another template literal function:

```js
const tag = () => console.log
tag`abc``def`
```

So currently be used with tag functions, the solution related to how to `chain` tag functions, a possible way is to introduce new operator for this purpose:

```js
tag <- `abc``def`
```

Introducing a new operator here only a possible way, but definitely not a best way maybe?

This problem should be discussed more, but the `RegExp` example should be write as below way to avoid semantic bugs:

```js
var re = new RegExp(String.raw <-
    `(\d{3}-)?` // area code (optional)
    `\d{3}-`    // prefix
    `\d{4}`     // line number
)
```


### Why not heredoc syntax?

There's already a proposal of heredoc like triple-backticks syntax:

https://github.com/tc39/proposal-string-dedent

But it's trying to improve only for the string indention case, which is also one of the cases this proposal tried to resolve, with some improvements:

1.  **Code compression**: using TLJ can reduce the JS size when using code compress tools like [UglifyJS](https://github.com/mishoo/UglifyJS) or [Terser](https://github.com/terser/terser), and minimized the unnecessary whitespaces (including the NewLine char in Syntax 2).
2.  **Runtime overhead**: TLJ has no or little runtime overhead, it's very suitable to transpile the TLJ into one combined Template Literal, or has little overhead to just combine TLJ in runtime with simple.
3.  **Comment**: TLJ allow comment each line out or add comment after.
4.  **Tools integration**: it's very simple to use an IDE plugin or similar tools to convert multiple line text into TLJ. it's common to use helper tools to make coding more easy, or using editor features to add backticks to start and end of each line. (e.g. The *VISUAL BLOCK mode* In vim).

Moreover, **Indentation detection** may be **failed** when *Mix* tabs and spaces.

## Reference

https://github.com/tc39/proposal-string-dedent

[Juxtaposition in Programming Languages · Daniel’s blog](http://www.storytotell.org/post/juxtaposition/)

https://en.wikipedia.org/wiki/Comparison_of_programming_languages_(strings)

