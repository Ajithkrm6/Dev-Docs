1. **Headers**
   Headers are used to create sections and subsections in your document. The number of # symbols before the text determines the header level.

markdown

```bash
# Header 1
## Header 2
### Header 3
#### Header 4
##### Header 5
###### Header 6
```

2. **Emphasis**

```bash
Italic: Use single asterisks * or underscores _ around the text.
*italic* or _italic_
Bold: Use double asterisks ** or double underscores __.
**bold** or __bold__
Bold and Italic: Use triple asterisks *** or triple underscores ___.
markdown
Copy code
***bold and italic*** or ___bold and italic___
```

3. **Lists**
   Unordered List: Use -, +, or \* followed by a space.

markdown
Copy code

- Item 1
- Item 2
  - Subitem 2.1
    Ordered List: Use numbers followed by a period.

markdown
Copy code

1. First item
2. Second item

   1. Subitem 2.1

3. **Code**
   Inline Code: Use single backticks ` for inline code.

markdown
Copy code
Use the `printf` function in C.
Code Blocks: Use triple backticks ``` or indent with four spaces for multi-line code blocks. Optionally specify the language for syntax highlighting.

markdown
Copy code

```bash
echo "Hello, World!"
Copy code
```

5. **Links**
   Inline Links: Use [link text](URL).

markdown
Copy code
[OpenAI](https://www.openai.com)
Reference Links: Define the link reference elsewhere in the document.

markdown
Copy code
[OpenAI][1]

[1]: https://www.openai.com

6. **Images**
   Inline Images: Use ![alt text](URL).
   markdown
   Copy code
   ![OpenAI Logo](https://www.openai.com/logo.png)

7. **Blockquotes**
   Use > for blockquotes.

markdown
Copy code

> This is a blockquote.
>
> It can span multiple lines.

8. **Horizontal Rules**
   Use three or more dashes ---, asterisks \*\*\*, or underscores \_\_\_.

markdown
Copy code

---

9. **Tables**
   Tables are created using pipes | and dashes -. You can align text using colons :.

markdown
Copy code
| Header 1 | Header 2 |
|----------|----------|
| Cell 1 | Cell 2 |
| Cell 3 | Cell 4 |

| Left Aligned | Center Aligned | Right Aligned |
| :----------- | :------------: | ------------: |
| Cell 1       |     Cell 2     |        Cell 3 |

10. **Task Lists**
    Use - [ ] for incomplete tasks and - [x] for completed tasks.

markdown
Copy code

- [ ] Task 1
- [x] Task 2

11. **HTML Tags**
    You can also use HTML tags for elements not directly supported by Markdown.

markdown
Copy code

<p>This is a paragraph in HTML.</p>

12. **Strikethrough**
    Use double tildes ~~ around the text.

markdown
Copy code
~~strikethrough~~
Putting It All Together
Here’s a complete Markdown example incorporating various elements:

markdown
Copy code

# My Markdown Guide

## Introduction

Markdown is a lightweight markup language for creating formatted text using a plain-text editor.

### Formatting

- **Bold**: `**bold**`
- _Italic_: `*italic*`
- **_Bold and Italic_**: `***bold and italic***`

### Code

Inline code: `print("Hello, World!")`

Code block:

````python
def hello_world():
    print("Hello, World!")
    ```
````
