# Chapter 1: Markup String

Imagine you're building a website where users can leave comments. People love to express themselves, and sometimes, they might type things like:

```
Hello, this is great! <script>alert('You've been hacked!')</script>
```

If you simply take this text and put it directly into your webpage, something terrible could happen! The `<script>` tag might run, showing a pop-up message or even stealing user information. This is a common security problem called **Cross-Site Scripting (XSS)**.

How do we stop this? We need a way to tell the web browser that characters like `<`, `>`, `&`, `'`, and `"` are just text to be displayed, not special HTML instructions. This is where `markupsafe` comes in, and its most important tool is the `Markup` string.

## What is a `Markup` String?

Think of a `Markup` string as a **"certified safe"** label for text that's going to be displayed on a webpage. When you have a `Markup` string, it's like you're telling the system: "Hey, I've already checked this text, and all the tricky characters are handled. It's perfectly fine to show this directly in HTML or XML."

It's a special type of Python string that knows its content is ready for display. Because it's "certified safe," the system won't try to escape it again, which prevents common problems like double-escaping (where `&` becomes `&amp;amp;`).

## How to Get a `Markup` String

There are two main ways to get a `Markup` string, depending on whether your original text is already safe or potentially dangerous:

### 1. Declaring Text as Already Safe (Use `Markup()` directly)

If you have a piece of text that you *know* is already valid HTML and doesn't need any escaping (for example, it comes from a trusted source or you constructed it carefully), you can wrap it directly in `Markup()`.

```python
from markupsafe import Markup

# You know this is safe HTML
safe_html = "<b>Hello</b>, world!"
my_markup = Markup(safe_html)

print(my_markup)
print(type(my_markup))
```

**What happens here?**
The `Markup()` constructor just wraps your `safe_html` string. It doesn't perform any escaping on it. It simply puts the "certified safe" label on it. This is useful when you're sure your content is already clean HTML.

### 2. Making Potentially Unsafe Text Safe (Use `Markup.escape()`)

Most of the time, you'll be dealing with text that might come from users or other untrusted sources. This text needs to be *escaped* to become safe. `Markup` provides a convenient way to do this using its `escape` class method.

```python
from markupsafe import Markup

# This text is potentially unsafe
user_input = "Hello, <script>alert('XSS')</script>!"
safe_output = Markup.escape(user_input)

print(safe_output)
print(type(safe_output))
```

**What happens here?**
`Markup.escape()` takes your `user_input` and converts special HTML characters (`<`, `>`, `&`, `'`, `"`) into their "entity" equivalents (`&lt;`, `&gt;`, `&amp;`, `&apos;`, `&quot;`). This way, the browser displays them as plain characters instead of interpreting them as HTML code. The result is a `Markup` string, signaling it's now safe.

## `Markup` Strings Behave Like Regular Strings (Mostly!)

One of the great things about `Markup` is that it acts almost exactly like a regular Python string. You can concatenate them, format them, and use many other string methods. The clever part is that `Markup` ensures that these operations *maintain safety*.

```python
from markupsafe import Markup

name = "<Alice>"
greeting = Markup("Hello, <b>%s</b>!") % Markup.escape(name)
# Notice how we escape 'name' before combining it with the Markup string.

print(greeting)

tag_start = Markup("<em>")
tag_end = Markup("</em>")
content = "My & friends"

combined = tag_start + Markup.escape(content) + tag_end

print(combined)
```

**What happens here?**
In the first example, `Markup("Hello, <b>%s</b>!") % Markup.escape(name)`:
*   The `Markup` string is the template.
*   The `name` variable is first passed through `Markup.escape()` to become safe (`&lt;Alice&gt;`).
*   The `%` operator (for string formatting) then combines these, and the result is a `Markup` string that is completely safe for HTML display.

In the second example, `tag_start + Markup.escape(content) + tag_end`:
*   `tag_start` and `tag_end` are already `Markup` objects, so they are trusted.
*   `content` is a regular string, so it's explicitly escaped with `Markup.escape()` before being added.
*   When you add `Markup` strings to other strings (or other `Markup` strings), `markupsafe` intelligently ensures that any non-`Markup` parts are properly escaped to prevent introducing new vulnerabilities. The final result is always a `Markup` object.

## Under the Hood: How `Markup` is Created

Let's briefly look at the core idea behind how `Markup` objects are created. This will help you understand the "certified safe" concept better.

```mermaid
sequenceDiagram
    participant YourCode as Your Python Code
    participant MarkupType as Markup Object
    participant EscapeFunc as escape() Function

    YourCode->>EscapeFunc: Call escape("Potentially unsafe <text>")
    EscapeFunc-->>MarkupType: Internally, a Markup object is created from the escaped text.
    MarkupType-->>YourCode: Returns Markup('Escaped &lt;text&gt;')

    Note right of YourCode: If you already have safe content:
    YourCode->>MarkupType: Call Markup("<b>Trusted HTML</b>")
    MarkupType-->>YourCode: Returns Markup('<b>Trusted HTML</b>')
```

As the diagram shows, there are two paths:

1.  **Using `escape()` (or `Markup.escape()` which calls it):** When you use the `escape()` function (which we'll cover in more detail in the next chapter), `markupsafe` takes your string, processes it to replace special characters, and then wraps the *escaped* result in a `Markup` object. This means the `Markup` object holds text that has been *made* safe.

2.  **Using `Markup()` directly:** When you call `Markup("some string")`, you are directly creating a `Markup` object. In this case, `markupsafe` *does not* perform any escaping. It just trusts that *you* know this string is already safe HTML and puts the "certified safe" label on it.

Let's peek at the `Markup` class's constructor (the `__new__` method is like the constructor for special cases like `str` subclasses) from `src/markupsafe/__init__.py`:

```python
class Markup(str):
    # ...
    def __new__(
        cls, object: t.Any = "", encoding: str | None = None, errors: str = "strict"
    ) -> te.Self:
        # Check if the object knows how to provide its own HTML representation
        if hasattr(object, "__html__"):
            object = object.__html__() # Get the HTML string from the object

        if encoding is None:
            return super().__new__(cls, object) # Create a new Markup string directly
        # ... (encoding handling is skipped for simplicity)
```

**Explanation:**
When you create a `Markup` object with `Markup(some_object)`:
*   It first checks if `some_object` has a special method called `__html__`. If it does, `Markup` uses the output of `some_object.__html__()` as the content. This is another way an object can declare its own HTML content as safe. (We'll dive into `__html__` in [HTML Safety Protocol (`__html__`)](03_html_safety_protocol_____html_____.md).)
*   If there's no `__html__` method, it simply takes `some_object` (converted to a string if it's not already) and uses it directly as the content for the new `Markup` string. No escaping happens here! This is why you must be careful when using `Markup()` directly with untrusted input.

And how does `Markup.escape()` work? It's a class method that simply calls the main `escape()` function, then ensures the result is of the correct `Markup` type:

```python
class Markup(str):
    # ...
    @classmethod
    def escape(cls, s: t.Any, /) -> te.Self:
        """Escape a string. Calls :func:`escape` and ensures that for
        subclasses the correct type is returned.
        """
        rv = escape(s) # Calls the global escape function
        if rv.__class__ is not cls:
            return cls(rv) # Ensure it's a Markup object
        return rv
```

So, `Markup.escape()` is just a convenient wrapper around the more general `escape()` function provided by `markupsafe`.

## Conclusion

In this chapter, we learned that `markupsafe`'s `Markup` class is a special string type that signals its content is safe for HTML. This prevents security vulnerabilities like XSS and avoids issues like double-escaping. We saw two main ways to get a `Markup` string:
1.  By directly creating `Markup("my safe html")` when you're sure the content is already safe.
2.  By using `Markup.escape("my potentially unsafe text")` to automatically handle special characters and make the content safe.

We also briefly saw how `Markup` strings maintain their "safe" property during operations like concatenation and formatting.

In the next chapter, we'll dive deeper into the core `escape()` function itself, understanding exactly how it transforms unsafe characters into their safe HTML equivalents.

[Escape Function](02_escape_function_.md)

---

Generated by [AI Codebase Knowledge Builder](https://github.com/The-Pocket/Tutorial-Codebase-Knowledge)