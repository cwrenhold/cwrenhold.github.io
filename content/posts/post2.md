---
title: "Code block testing"
date: 2023-02-19T15:55:41Z
summary: This page is just for testing how different code blocks look in hugo out of the box, nothing special.
---

## Some subtitle

This page is just for testing how different code blocks look in hugo out of the box, nothing special.

```javascript
const a = 5;
let b = "Hello world!";

function c() {
    console.log(b);
}
```

```csharp
namespace something;

public class SomeClass
{
    public string SomeText { get; init; }

    public int SomeMutable { get; set; }

    public void Method()
    {
        Console.WriteLine($"{this.SomeText}: {this.SomeMutable}");
    }
}
```

```goat
          .                     .                .               .--- 1          .-- 1     / 1
         / \                    |                |           .---+            .-+         +
        /   \               +---+--.          .--+--.        |   '--- 2      |   '-- 2   / \ 2
       +     +              |       |        |       |    ---+            ---+          +
      / \   / \           .-+-.   .-+-.     .+.     .+.      |   .--- 3      |   .-- 3   \ / 3
     /   \ /   \          |   |   |   |    |   |   |   |     '---+            '-+         +
     1   2 3   test123    1   2   3   4    1   2   3   4         '--- 4          '-- 4     \ 4
```
