# Introduction to READMEs and Markdown

This guide will introduce you to the [README file](https://en.wikipedia.org/wiki/README) and fundamentals of the [Markdown](https://daringfireball.net/projects/markdown/) programming language.
If you have some experience with HTML, XML or any other [markup language](https://en.wikipedia.org/wiki/Markup_language), then you'll be able to pick Markdown up with little to no difficulty.

If not, don't fear! Markdown is designed to be simple and intuitive, and is a useful language for beginners and professionals alike.

There are [additional resources](#additional-resources) at the end to provide some extra information for a deeper dive into READMEs and Markdown.

## What is a README and what are they used for?

### General

A README is a simple text file that is used to communicate important information about a project or application, especially when opening it for the first time.
READMEs can be written in any plain-text editor and common formats include `.txt` and `.md` (Markdown).

README files typically contain relevant information about the software, including:

-   download, configuration and installation instructions
-   basic operation guide and features
-   list of known bugs
-   troubleshooting information and FAQs
-   credits and acknowledgements

Author and contributor information, copyright, license specifications and changelogs are also important bits of information that may be present in a README or can be found in separate text files in the root of the project, where the README is typically located.

### GitHub READMEs

A common use case for READMEs is in [GitHub](https://github.com/) repositories, where they appear to visitors immediately on arriving at a repository's main page, to provide an outline and setup guide for the project.

GitHub READMEs generally contain information on why the project is useful, what users can do with the project, how users can leverage its features, who is involved in the project and where help can be found.

Lengthy technical information and documentation is best saved for project [wikis](https://en.wikipedia.org/wiki/Wiki), and README files should only contain the necessary information for users and contributors to get started.

## What is Markdown?

### Description

Markdown is a [lightweight markup language](https://en.wikipedia.org/wiki/Lightweight_markup_language) that was developed by John Gruber and Aaron Swartz, and that is used for formatting plain text.

As a [human-readable programming language](https://en.wikipedia.org/wiki/Human-readable_medium) with simple and unobtrusive syntax, it is commonly used for blogging, instant messaging (e.g. Discord and Microsoft Teams) and in forums (e.g. Reddit and StackExchange).

It is also commonly found in collaborative software environments (such as GitHub and BitBucket), in documentation and can be used to write README files.

### Useful syntax elements

> Style specifications are known to be slightly different between variants of Markdown, so it is advised to confirm best practices in your relevant flavor guide if you're not using the standard vanilla version.
> For example, [GitHub provide their own flavor](<(https://github.github.com/gfm/)>), which has its own specifications.
> For general best practice guidelines, see the [Markdown Guide](https://www.markdownguide.org/).

#### Headings

These are formatted with the `#` symbol placed immediately before the text that you would like to make up the heading.
The number of `#` symbols you use corresponds with the heading level, where the text size of each heading gets smaller as you go deeper:

```
# Level 1

## Level 2

### Level 3

#### Level 4

##### Level 5
```

# Level 1

## Level 2

### Level 3

#### Level 4

##### Level 5

#### Line breaks

To create a line break, finish the line with two or more spaces, then press `RETURN`:

```
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
Volutpat sed cras ornare arcu dui vivamus arcu felis bibendum.
Vulputate enim nulla aliquet porttitor lacus luctus accumsan tortor posuere.
```

Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.  
Volutpat sed cras ornare arcu dui vivamus arcu felis bibendum.  
Vulputate enim nulla aliquet porttitor lacus luctus accumsan tortor posuere.

#### Paragraphs

> Note that there are no spaces after each line in the following example, so each sentence is concatenated onto the previous one without a line break, despite being on a separate line.

These are separated by empty lines:

```
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
Volutpat sed cras ornare arcu dui vivamus arcu felis bibendum.
Vulputate enim nulla aliquet porttitor lacus luctus accumsan tortor posuere.
Sed lectus vestibulum mattis ullamcorper velit sed ullamcorper morbi tincidunt.

Duis at tellus at urna condimentum mattis pellentesque id.
Risus pretium quam vulputate dignissim suspendisse.
Duis ut diam quam nulla porttitor massa id neque aliquam.
Suspendisse interdum consectetur libero id faucibus nisl tincidunt eget nullam.

Eget nullam non nisi est sit amet facilisis magna.
Etiam sit amet nisl purus in.
Sagittis vitae et leo duis ut.
Consectetur adipiscing elit duis tristique sollicitudin.
```

Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
Volutpat sed cras ornare arcu dui vivamus arcu felis bibendum.
Vulputate enim nulla aliquet porttitor lacus luctus accumsan tortor posuere.
Sed lectus vestibulum mattis ullamcorper velit sed ullamcorper morbi tincidunt.

Duis at tellus at urna condimentum mattis pellentesque id.
Risus pretium quam vulputate dignissim suspendisse.
Duis ut diam quam nulla porttitor massa id neque aliquam.
Suspendisse interdum consectetur libero id faucibus nisl tincidunt eget nullam.

Eget nullam non nisi est sit amet facilisis magna.
Etiam sit amet nisl purus in.
Sagittis vitae et leo duis ut.
Consectetur adipiscing elit duis tristique sollicitudin.

#### Emphasis

##### Italics

Italic text is written with the desired text being surrounded by a single underscore (`_`) or asterisk (`*`) on each side:

```
Lorem ipsum _dolor_ sit amet, *consectetur adipiscing elit*, sed do _eiusmod tempor incididunt_ ut labore et dolore magna *aliqua*.
```

Lorem ipsum _dolor_ sit amet, _consectetur adipiscing elit_, sed do _eiusmod tempor incididunt_ ut labore et dolore magna _aliqua_.

---

##### Bold

Text can be made bold with two asterisks (`**`) or two underscores (`__`) on either side of the desired text:

```
Lorem ipsum **dolor** sit amet, __consectetur adipiscing elit__, sed do **eiusmod tempor incididunt** ut labore et dolore magna __aliqua__.
```

Lorem ipsum **dolor** sit amet, **consectetur adipiscing elit**, sed do **eiusmod tempor incididunt** ut labore et dolore magna **aliqua**.

> For simplicity's sake, it may be a good idea to save asterisks for bold text and underscores for italic text (or vice versa).

##### Bold and italic

Text that is bold and italic can be achieved by using three asterisks (`***`) or three underscores (`___`) on either side of the desired text, two asterisks and an underscore (`**_` to start and `_**` to end) or two underscores and an asterisk (`__*` to start and `*__` to end):

```
Lorem ipsum ***dolor*** sit amet, ___consectetur adipiscing elit___, sed do **_eiusmod tempor incididunt_** ut labore et dolore magna __*aliqua*__.
```

Lorem ipsum **_dolor_** sit amet, **_consectetur adipiscing elit_**, sed do **_eiusmod tempor incididunt_** ut labore et dolore magna **_aliqua_**.

#### Lists

##### Ordered

These are achieved by using a number following by a period (`.`).
The order of the numbers does not matter as Markdown automatically formats the list to start from 1, without affecting the order of the items:

```
2. Item 1
3. Item 2
1. Item 3
```

1. Item 1
2. Item 2
3. Item 3

##### Unordered

These are written with items preceded by a minus sign (`-`), an asterisk (`*`) or plus sign (`+`).
All of these render the same round bullet-points:

```
- foo
- bar
- baz
```

-   foo
-   bar
-   baz

```
* foo
* bar
* baz
```

-   foo
-   bar
-   baz

```
+ foo
+ bar
+ baz
```

-   foo
-   bar
-   baz

#### Code

Individual words or phrases (e.g. keywords, variables names, expressions, function names, file names etc.) are surrounded by a single backtick (`` ` ``):

```
`true`
`someVar`
`x = 4`
`helloWorld()`
`file.txt`
```

becomes:

`true`  
`someVar`  
`x = 4`  
`helloWorld()`  
`file.txt`

Longer fenced code blocks are written with three backticks (` ``` `) at the beginning and end of the code snippet on their own lines.
Optional syntax highlighting is added by providing the name of the programming language immediately after the opening backticks (e.g. ` ```typescript ` below), on the first line:

````
```typescript
function printHelloWorld() {
    let hello: string = "Hello";
    let world: string = "World!";
    console.log(hello, world);
}
```
````

becomes:

```typescript
function printHelloWorld() {
    let hello: string = "Hello";
    let world: string = "World!";
    console.log(hello, world);
}
```

#### Links

##### With label text

Links are written with the link label enclosed in square braces (`[ ]`) followed immediately by the URL iteslf in parentheses (`( )`):

```
More information on [getting started with Markdown](https://www.markdownguide.org/getting-started/) can be found in the Markdown Guide.
```

More information on [getting started with Markdown](https://www.markdownguide.org/getting-started/) can be found in the Markdown Guide.

##### Without a label

Email addresses and standalone URLs are written with the link enclosed in angle brackets (`< >`):

```
Contact support at <foobar@bazmail.com> or visit our website at <https://foo.bar/baz/>.
```

Contact support at <foobar@bazmail.com> or visit our website at <https://foo.bar/baz/>.

---

Links can also be bold or italic:

```
Contact support at **<foobar@bazmail.com>** or visit our website at _<https://foo.bar/baz/>_.
```

Contact support at **<foobar@bazmail.com>** or visit our website at _<https://foo.bar/baz/>_.

## Final notes

Markdown is an incredibly useful tool to compose formatted text files, and READMEs are only one of its many use cases.
A well-written README can be found in any professional project or application and it is crucial to allow end users and contributors to confidently access and interact with your software.
While this guide only covers some of the basics, it is worth noting that taking the time to write a README that is purpose-built and properly formatted in a language like Markdown is an important part of developing software, and is highly recommended.

## Additional resources

### READMEs

-   [Make a README](https://www.makeareadme.com/)
-   [Awesome README - GitHub repository](https://github.com/matiassingers/awesome-readme)
-   [About READMEs - GitHub docs](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-readmes)
-   [About Wikis - GitHub docs](https://docs.github.com/en/communities/documenting-your-project-with-wikis/about-wikis)

### Markdown

-   [Markdown - Wikipedia](https://en.wikipedia.org/wiki/Markdown)
-   [Markdown Cheat Sheet - Markdown Guide](https://www.markdownguide.org/cheat-sheet/)
-   [Basic Syntax - Markdown Guide](https://www.markdownguide.org/basic-syntax/)
-   [Extended Syntax - Markdown Guide](https://www.markdownguide.org/extended-syntax/)
