# The Complete HTML & CSS Tutorial: Beginner to Advanced

A comprehensive, hands-on guide to building websites with HTML and CSS. Work through it top to bottom, typing out the examples yourself — you'll learn far more by typing than by reading.

---

## Table of Contents

1. [How the Web Works (Quick Primer)](#part-1-how-the-web-works)
2. [Setting Up Your Environment](#part-2-setting-up-your-environment)
3. [HTML Fundamentals](#part-3-html-fundamentals)
4. [HTML Forms](#part-4-html-forms)
5. [Semantic HTML & Accessibility](#part-5-semantic-html--accessibility)
6. [CSS Fundamentals](#part-6-css-fundamentals)
7. [The Box Model](#part-7-the-box-model)
8. [Typography & Text Styling](#part-8-typography--text-styling)
9. [Backgrounds, Borders & Shadows](#part-9-backgrounds-borders--shadows)
10. [CSS Layout: Display, Position & Float](#part-10-css-layout-display-position--float)
11. [Flexbox (Deep Dive)](#part-11-flexbox-deep-dive)
12. [CSS Grid (Deep Dive)](#part-12-css-grid-deep-dive)
13. [Responsive Design & Media Queries](#part-13-responsive-design--media-queries)
14. [Pseudo-Classes & Pseudo-Elements](#part-14-pseudo-classes--pseudo-elements)
15. [Transitions, Transforms & Animations](#part-15-transitions-transforms--animations)
16. [CSS Custom Properties (Variables)](#part-16-css-custom-properties-variables)
17. [The Cascade, Specificity & Inheritance](#part-17-the-cascade-specificity--inheritance)
18. [Modern CSS Layout Tools](#part-18-modern-css-layout-tools)
19. [Best Practices, Naming & Architecture](#part-19-best-practices-naming--architecture)
20. [Full Project Walkthrough](#part-20-full-project-walkthrough)
21. [Where to Go Next](#part-21-where-to-go-next)

---

## Part 1: How the Web Works

Before writing code, it helps to know what you're actually building.

- **HTML (HyperText Markup Language)** describes the *structure and content* of a page — headings, paragraphs, images, links, forms.
- **CSS (Cascading Style Sheets)** describes the *presentation* — colors, fonts, spacing, layout, animation.
- **JavaScript** (not covered here) adds *behavior* — interactivity, logic, dynamic updates.

A browser requests an HTML file from a server, parses it into a tree called the **DOM (Document Object Model)**, then applies CSS rules to that tree to decide how each element should look, and finally paints pixels to the screen. Understanding that HTML = structure and CSS = presentation will keep your code cleaner as you go — resist the urge to use HTML to "make things look a certain way."

---

## Part 2: Setting Up Your Environment

You need only three things:

1. **A text editor.** [VS Code](https://code.visualstudio.com/) is the most popular free option and has excellent HTML/CSS support out of the box.
2. **A web browser.** Chrome, Firefox, or Edge all have great built-in developer tools (right-click → "Inspect").
3. **A folder** on your computer to hold your files.

Create a project folder, and inside it create two files:

```
my-site/
├── index.html
└── style.css
```

`index.html` is the conventional name for a site's homepage — most web servers automatically serve this file when someone visits your domain.

### Your first HTML file

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My First Page</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <h1>Hello, World!</h1>
    <p>This is my first web page.</p>
</body>
</html>
```

Save this, then double-click `index.html` (or drag it into a browser window) to view it. Every change you make and save can be viewed by refreshing the browser. This is the entire feedback loop you'll use for the rest of this guide: **edit → save → refresh**.

Let's break down what you just wrote:

- `<!DOCTYPE html>` tells the browser this is an HTML5 document. Always include it, first line, exactly as shown.
- `<html lang="en">` is the root element wrapping everything. The `lang` attribute helps screen readers and search engines.
- `<head>` holds metadata — information *about* the page that isn't displayed directly (title, character encoding, linked stylesheets, etc.).
- `<meta charset="UTF-8">` ensures special characters (emoji, accented letters, etc.) render correctly.
- `<meta name="viewport" ...>` is essential for responsive design — it tells mobile browsers to use the device's actual width rather than pretending to be a desktop. Never skip this.
- `<title>` sets the browser tab text.
- `<link rel="stylesheet" href="style.css">` connects your CSS file.
- `<body>` holds everything that's actually visible on the page.

---

## Part 3: HTML Fundamentals

### 3.1 Elements, Tags, and Attributes

An HTML **element** usually consists of an opening tag, content, and a closing tag:

```html
<p>This is a paragraph.</p>
```

- `<p>` — opening tag
- `This is a paragraph.` — content
- `</p>` — closing tag (note the forward slash)

Some elements are **self-closing** (void elements) because they don't wrap content:

```html
<img src="photo.jpg" alt="A description of the photo">
<br>
<hr>
<input type="text">
```

**Attributes** provide extra information about an element and always go inside the opening tag as `name="value"` pairs:

```html
<a href="https://example.com" target="_blank">Visit Example</a>
```

Here `href` and `target` are attributes of the `<a>` (anchor/link) element.

### 3.2 Headings and Paragraphs

```html
<h1>Main Page Title (only one per page)</h1>
<h2>A Major Section</h2>
<h3>A Subsection</h3>
<h4>Smaller heading</h4>
<h5>Even smaller</h5>
<h6>Smallest heading</h6>

<p>A paragraph of text. Browsers automatically add space before and after paragraphs.</p>
```

Use headings to convey a **document outline**, not for font size — never skip from `<h1>` straight to `<h4>` just because you want smaller text. Use CSS for sizing instead.

### 3.3 Text Formatting

```html
<p>You can make text <strong>strongly important</strong> or <em>emphasized</em>.</p>
<p>Other options: <b>bold</b>, <i>italic</i>, <mark>highlighted</mark>, <small>small print</small>.</p>
<p>Superscript: E = mc<sup>2</sup>. Subscript: H<sub>2</sub>O.</p>
<p>Strikethrough for edits: <del>old price $40</del> <ins>new price $30</ins>.</p>
<hr>
<blockquote>
    A longer quotation goes here, often indented by default browser styles.
</blockquote>
<p>Inline code: <code>const x = 5;</code></p>
<pre><code>function greet() {
  console.log("Preformatted text preserves whitespace and line breaks");
}</code></pre>
```

`<strong>` and `<em>` carry semantic meaning (importance/emphasis, useful to screen readers), while `<b>` and `<i>` are purely visual. Prefer `<strong>`/`<em>` when meaning matters.

### 3.4 Lists

**Unordered (bullet) list:**

```html
<ul>
    <li>Milk</li>
    <li>Eggs</li>
    <li>Bread</li>
</ul>
```

**Ordered (numbered) list:**

```html
<ol>
    <li>Preheat the oven</li>
    <li>Mix the batter</li>
    <li>Bake for 25 minutes</li>
</ol>
```

**Description list** (term/definition pairs):

```html
<dl>
    <dt>HTML</dt>
    <dd>The markup language used to structure web pages.</dd>
    <dt>CSS</dt>
    <dd>The stylesheet language used to style web pages.</dd>
</dl>
```

Lists can be nested:

```html
<ul>
    <li>Fruits
        <ul>
            <li>Apple</li>
            <li>Banana</li>
        </ul>
    </li>
    <li>Vegetables</li>
</ul>
```

### 3.5 Links

```html
<a href="https://example.com">Absolute link to another site</a>
<a href="/about.html">Relative link (same domain)</a>
<a href="about.html">Relative link to a file in the same folder</a>
<a href="../images/photo.jpg">Relative link, one folder up</a>
<a href="#section2">Jump to an element with id="section2" on this page</a>
<a href="mailto:someone@example.com">Send an email</a>
<a href="tel:+1234567890">Call a phone number</a>
<a href="https://example.com" target="_blank" rel="noopener noreferrer">Open in new tab</a>
```

Always add `rel="noopener noreferrer"` when using `target="_blank"` — it's a security best practice that prevents the new page from gaining partial control over the original tab.

### 3.6 Images

```html
<img src="images/dog.jpg" alt="A golden retriever running on a beach" width="600" height="400">
```

- `src` — path to the image file.
- `alt` — **required**. Describes the image for screen readers, and displays if the image fails to load. Never leave this out; use `alt=""` (empty) only for purely decorative images.
- `width`/`height` — specifying these (even though CSS will usually override the actual display size) helps the browser reserve space and avoid layout shift while the image loads.

Responsive images with multiple sources:

```html
<picture>
    <source media="(min-width: 800px)" srcset="large.jpg">
    <source media="(min-width: 400px)" srcset="medium.jpg">
    <img src="small.jpg" alt="A responsive example image">
</picture>
```

### 3.7 Tables

Use tables for **tabular data only** — never for page layout.

```html
<table>
    <caption>Quarterly Sales</caption>
    <thead>
        <tr>
            <th>Quarter</th>
            <th>Revenue</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Q1</td>
            <td>$50,000</td>
        </tr>
        <tr>
            <td>Q2</td>
            <td>$62,000</td>
        </tr>
    </tbody>
    <tfoot>
        <tr>
            <td>Total</td>
            <td>$112,000</td>
        </tr>
    </tfoot>
</table>
```

- `<thead>`, `<tbody>`, `<tfoot>` group rows semantically.
- `<th>` is a header cell (bold and centered by default); `<td>` is a regular data cell.
- `<th scope="col">` or `<th scope="row">` improves accessibility by clarifying what the header describes.

### 3.8 Div and Span

`<div>` and `<span>` carry **no inherent meaning** — they exist purely as hooks for CSS or JavaScript.

```html
<div>A block-level generic container (starts on a new line, takes full width).</div>
<p>A paragraph with a <span>generic inline container</span> inside it.</p>
```

Rule of thumb: reach for a semantic element (Part 5) first; use `<div>`/`<span>` only when nothing more meaningful fits.

### 3.9 Comments

```html
<!-- This is a comment. It won't be shown in the browser. -->
```

---

## Part 4: HTML Forms

Forms collect user input — the backbone of logins, search bars, checkouts, and more.

```html
<form action="/submit" method="POST">
    <label for="name">Name:</label>
    <input type="text" id="name" name="name" placeholder="Jane Doe" required>

    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>

    <label for="password">Password:</label>
    <input type="password" id="password" name="password" minlength="8" required>

    <label for="age">Age:</label>
    <input type="number" id="age" name="age" min="0" max="120">

    <label for="birthday">Birthday:</label>
    <input type="date" id="birthday" name="birthday">

    <fieldset>
        <legend>Favorite color</legend>
        <input type="radio" id="red" name="color" value="red">
        <label for="red">Red</label>
        <input type="radio" id="blue" name="color" value="blue">
        <label for="blue">Blue</label>
    </fieldset>

    <label for="newsletter">
        <input type="checkbox" id="newsletter" name="newsletter">
        Subscribe to newsletter
    </label>

    <label for="country">Country:</label>
    <select id="country" name="country">
        <option value="">-- Choose --</option>
        <option value="us">United States</option>
        <option value="ca">Canada</option>
        <option value="dz">Algeria</option>
    </select>

    <label for="message">Message:</label>
    <textarea id="message" name="message" rows="4" cols="40"></textarea>

    <button type="submit">Submit</button>
    <button type="reset">Reset</button>
</form>
```

Key points:

- Always pair `<label for="id">` with an input's matching `id`. This lets users click the label to focus the field, and it's essential for screen-reader accessibility.
- The `name` attribute is what gets sent to the server; the `id` attribute is what CSS/JavaScript/labels reference. They're different jobs, even when you give them the same value.
- HTML5 gives you many built-in input types (`email`, `number`, `date`, `tel`, `url`, `range`, `color`, `file`, etc.) that provide free browser-level validation and better mobile keyboards — use the specific type rather than defaulting everything to `type="text"`.
- `required`, `minlength`, `maxlength`, `min`, `max`, and `pattern` give you client-side validation without any JavaScript.
- `<fieldset>` + `<legend>` groups related fields (like a set of radio buttons) and announces the group's purpose to screen readers.

---

## Part 5: Semantic HTML & Accessibility

"Semantic" elements describe their *meaning*, not just their appearance. Prefer them over generic `<div>`s wherever a matching one exists — they improve accessibility, SEO, and code readability all at once.

```html
<header>
    <h1>My Website</h1>
    <nav>
        <ul>
            <li><a href="#home">Home</a></li>
            <li><a href="#about">About</a></li>
            <li><a href="#contact">Contact</a></li>
        </ul>
    </nav>
</header>

<main>
    <article>
        <h2>Blog Post Title</h2>
        <p>Published on <time datetime="2026-09-04">September 4, 2026</time></p>
        <p>Article content...</p>
    </article>

    <aside>
        <h3>Related Links</h3>
        <ul>
            <li><a href="#">Another article</a></li>
        </ul>
    </aside>

    <section>
        <h2>Newsletter Signup</h2>
        <p>Section content...</p>
    </section>
</main>

<footer>
    <p>&copy; 2026 My Website. All rights reserved.</p>
</footer>
```

Quick reference:

| Element | Purpose |
| --- | --- |
| `<header>` | Introductory content, typically containing a logo/title and nav |
| `<nav>` | A block of navigation links |
| `<main>` | The primary content of the page (use once per page) |
| `<article>` | Self-contained content that could stand alone (blog post, news story) |
| `<section>` | A thematic grouping of content, usually with its own heading |
| `<aside>` | Content tangentially related to the main content (sidebar, pull quote) |
| `<footer>` | Footer content: copyright, links, contact info |
| `<figure>` / `<figcaption>` | An image/diagram plus its caption |

### A quick note on accessibility

- Every `<img>` needs `alt` text.
- Every form input needs an associated `<label>`.
- Use real buttons (`<button>`) for actions and real links (`<a href="...">`) for navigation — don't make a `<div>` clickable with JavaScript when a native element already does the job (and comes with keyboard support for free).
- Maintain a logical heading order (`h1` → `h2` → `h3`, no skipping).
- Ensure sufficient color contrast between text and background (Part 8 touches on this).
- Use `aria-*` attributes only when semantic HTML genuinely can't express what you need — they're a supplement, not a substitute.

---

## Part 6: CSS Fundamentals

### 6.1 Three Ways to Add CSS

**1. External stylesheet (recommended — keeps HTML and CSS separate):**

```html
<link rel="stylesheet" href="style.css">
```

**2. Internal `<style>` block (fine for quick demos):**

```html
<head>
    <style>
        p { color: navy; }
    </style>
</head>
```

**3. Inline style attribute (avoid except for one-off dynamic cases — hard to maintain, and highest specificity, which fights the cascade):**

```html
<p style="color: navy;">Text</p>
```

Use external stylesheets for real projects. It lets you reuse one CSS file across many pages and keeps content separate from presentation.

### 6.2 CSS Syntax

```css
selector {
    property: value;
    property: value;
}
```

Example:

```css
p {
    color: darkslategray;
    font-size: 18px;
    line-height: 1.6;
}
```

- **Selector** — which element(s) the rule applies to.
- **Declaration block** — the `{ }` wrapped set of rules.
- **Property: value;** — each individual style, ending in a semicolon.

### 6.3 Selectors

```css
/* Element selector — targets all <p> elements */
p { color: black; }

/* Class selector — targets elements with class="highlight" */
.highlight { background-color: yellow; }

/* ID selector — targets the one element with id="main-title" */
#main-title { font-size: 2rem; }

/* Universal selector — targets everything */
* { box-sizing: border-box; }

/* Grouping selector — apply the same rule to multiple selectors */
h1, h2, h3 { font-family: Georgia, serif; }

/* Descendant combinator — targets <a> anywhere inside <nav> */
nav a { text-decoration: none; }

/* Child combinator — targets direct <li> children only, not grandchildren */
ul > li { list-style: square; }

/* Adjacent sibling — targets a <p> immediately after an <h2> */
h2 + p { font-weight: bold; }

/* General sibling — targets any <p> after an <h2> at the same level */
h2 ~ p { color: gray; }

/* Attribute selector */
input[type="text"] { border: 1px solid #ccc; }
a[target="_blank"] { color: red; }

/* Multiple classes on the same element */
.card.featured { border: 2px solid gold; }
```

Corresponding HTML for the class/id examples:

```html
<p class="highlight">This paragraph has a yellow background.</p>
<h1 id="main-title">Big Title</h1>
```

- **Classes** (`.classname`) are reusable — many elements can share one class, and one element can have several classes (`class="card featured"`).
- **IDs** (`#idname`) must be unique per page — use them sparingly, mainly as anchors or JS hooks, since they carry very high specificity (Part 17) and can be hard to override.

### 6.4 Colors

```css
.box {
    color: red;                     /* named color */
    background-color: #3498db;      /* hex */
    border-color: rgb(52, 152, 219);/* rgb */
    outline-color: rgba(52, 152, 219, 0.5); /* rgba, with transparency */
    box-shadow: 0 0 5px hsl(204, 70%, 53%); /* hsl */
}
```

`hsl()` (hue, saturation, lightness) is often the most intuitive for tweaking a color — keep hue and saturation fixed and just slide lightness up or down to get a palette of tints and shades.

### 6.5 Units

| Unit | Meaning |
| --- | --- |
| `px` | Pixels — fixed, absolute size |
| `%` | Percentage of the parent element |
| `em` | Relative to the font size of the *current* element |
| `rem` | Relative to the font size of the *root* `<html>` element (usually 16px) — predictable, preferred for most sizing |
| `vw` / `vh` | 1% of viewport width / height — useful for full-screen layouts |
| `vmin` / `vmax` | 1% of the smaller/larger of viewport width and height |

```css
html { font-size: 16px; }              /* the "root" size rem is based on */
h1 { font-size: 2.5rem; }              /* 40px, and scales if the root changes */
.hero { height: 100vh; }               /* full viewport height */
.container { width: 90%; max-width: 1200px; margin: 0 auto; }
```

`rem` is generally the best default for font sizes and spacing because it respects the user's browser font-size settings (important for accessibility) while staying predictable — unlike `em`, which compounds as you nest elements.

---

## Part 7: The Box Model

Every HTML element is a rectangular box made of four layers, from the inside out:

```
┌─────────────────────────────┐
│           margin             │  (space outside the border, transparent)
│  ┌─────────────────────────┐ │
│  │         border           │ │
│  │  ┌─────────────────────┐│ │
│  │  │      padding         ││ │
│  │  │  ┌─────────────────┐ ││ │
│  │  │  │     content      │ ││ │
│  │  │  └─────────────────┘ ││ │
│  │  └─────────────────────┘│ │
│  └─────────────────────────┘ │
└─────────────────────────────┘
```

```css
.box {
    width: 300px;
    padding: 20px;
    border: 2px solid black;
    margin: 10px;
}
```

- **Content** — the actual text/image/etc.
- **Padding** — space between the content and the border (inherits the background color).
- **Border** — a visible (or invisible) line around the padding.
- **Margin** — space outside the border, separating this box from others (always transparent).

### 7.1 `box-sizing` — the setting that will save you headaches

By default (`box-sizing: content-box`), `width`/`height` apply only to the *content* — padding and border are added on top, making the box bigger than the width you set. Almost every developer instead uses `border-box`, where `width`/`height` include padding and border, making sizing far more predictable:

```css
*, *::before, *::after {
    box-sizing: border-box;
}
```

Put this at the top of every stylesheet you write. It's close to a universal convention in modern CSS.

### 7.2 Shorthand for padding and margin

```css
.box {
    padding: 10px;                    /* all four sides */
    padding: 10px 20px;               /* top/bottom, left/right */
    padding: 10px 20px 15px;          /* top, left/right, bottom */
    padding: 10px 20px 15px 5px;      /* top, right, bottom, left (clockwise) */

    margin: 0 auto;                   /* common trick: 0 top/bottom, auto left/right — centers a block element horizontally */
}
```

### 7.3 Display types (preview — more in Part 10)

```css
.block-example   { display: block; }        /* full width, new line (div, p, h1...) */
.inline-example   { display: inline; }       /* flows with text, ignores width/height (span, a...) */
.inline-block-example { display: inline-block; } /* flows inline but respects width/height/margin */
.hidden { display: none; }                   /* removed entirely from layout */
```

---

## Part 8: Typography & Text Styling

```css
body {
    font-family: "Helvetica Neue", Arial, sans-serif; /* fallback fonts in case the first isn't available */
    font-size: 16px;
    font-weight: 400;
    line-height: 1.6;         /* unitless is best — scales relative to font-size */
    color: #222;
}

h1 {
    font-family: Georgia, "Times New Roman", serif;
    font-weight: 700;
    font-style: italic;
    text-transform: uppercase;   /* uppercase | lowercase | capitalize | none */
    letter-spacing: 0.05em;
    text-align: center;          /* left | right | center | justify */
    text-decoration: underline;  /* underline | line-through | none */
}

a {
    text-decoration: none;
}
a:hover {
    text-decoration: underline;
}

p {
    text-indent: 2em;         /* indent first line, like a printed book */
    white-space: pre-wrap;    /* preserve whitespace/line breaks in the source */
    word-break: break-word;   /* prevent long words from overflowing their container */
}
```

### Web fonts with Google Fonts

```html
<head>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap" rel="stylesheet">
</head>
```

```css
body {
    font-family: "Inter", sans-serif;
}
```

### A note on contrast and readability

Aim for a **contrast ratio of at least 4.5:1** between text and its background for normal-sized text (WCAG AA standard). Dark gray text (`#333`) on white is comfortable to read; pure black on white (`#000` on `#fff`) can feel harsh; light gray on white often fails accessibility checks entirely. Browser dev tools can measure contrast ratios directly when you inspect an element.

---

## Part 9: Backgrounds, Borders & Shadows

```css
.card {
    background-color: #f5f5f5;
    background-image: url("pattern.svg");
    background-repeat: no-repeat;
    background-position: center;
    background-size: cover;      /* cover | contain | 100px 100px | etc. */
    background-attachment: fixed; /* parallax-style fixed background */

    /* shorthand version of all the above: */
    background: #f5f5f5 url("pattern.svg") no-repeat center / cover;
}

.gradient-box {
    background: linear-gradient(to right, #ff7e5f, #feb47b);
}
.radial-box {
    background: radial-gradient(circle, #ffffff, #cccccc);
}

.rounded {
    border: 1px solid #ddd;
    border-radius: 12px;          /* rounded corners */
    border-radius: 50%;           /* perfect circle, if width == height */
}

.shadowed {
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);  /* x-offset y-offset blur color */
    box-shadow: 0 4px 8px rgba(0,0,0,0.15), 0 1px 2px rgba(0,0,0,0.1); /* multiple layered shadows */
}

.text-shadow-example {
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
}
```

---

## Part 10: CSS Layout: Display, Position & Float

### 10.1 The `position` property

```css
.static-box {
    position: static;   /* default — normal document flow, top/left/etc. do nothing */
}

.relative-box {
    position: relative;  /* stays in normal flow, but top/left/etc. offset it FROM where it would normally be */
    top: 10px;
    left: 20px;
}

.absolute-box {
    position: absolute;  /* removed from normal flow entirely; positioned relative to nearest positioned ancestor */
    top: 0;
    right: 0;
}

.fixed-box {
    position: fixed;     /* positioned relative to the browser viewport; stays put when scrolling */
    bottom: 20px;
    right: 20px;
}

.sticky-box {
    position: sticky;    /* behaves like relative, then "sticks" like fixed once you scroll past a threshold */
    top: 0;
}
```

A very common pattern: give the *parent* `position: relative` so an *absolutely* positioned child is placed relative to that parent, not the whole page:

```html
<div class="card">
    <span class="badge">New</span>
    <p>Product description...</p>
</div>
```

```css
.card {
    position: relative;
}
.badge {
    position: absolute;
    top: 8px;
    right: 8px;
    background: crimson;
    color: white;
    padding: 4px 8px;
    border-radius: 4px;
}
```

### 10.2 Float (mostly legacy — Flexbox/Grid replace this today)

```css
.float-image {
    float: left;
    margin-right: 15px;
}
.clearfix::after {
    content: "";
    display: table;
    clear: both;
}
```

You'll still see `float` for wrapping text around an image, but for actual page layout, use Flexbox or Grid (next two parts) — they're far more powerful and predictable.

### 10.3 `z-index` — stacking order

```css
.back  { position: relative; z-index: 1; }
.front { position: relative; z-index: 2; } /* renders above .back when they overlap */
```

`z-index` only works on elements with a `position` value other than `static`.

---

## Part 11: Flexbox (Deep Dive)

Flexbox is designed for laying out items in a **single dimension** — a row or a column — and is the tool you'll reach for constantly: navbars, button groups, centering things, card rows.

### 11.1 Basic setup

```html
<div class="flex-container">
    <div class="item">1</div>
    <div class="item">2</div>
    <div class="item">3</div>
</div>
```

```css
.flex-container {
    display: flex;
}
```

That single line already lines the three `.item`s up in a row, all the same height. Everything else is fine-tuning.

### 11.2 Container properties

```css
.flex-container {
    display: flex;
    flex-direction: row;          /* row | row-reverse | column | column-reverse */
    flex-wrap: wrap;              /* nowrap (default) | wrap | wrap-reverse */
    justify-content: space-between; /* flex-start | flex-end | center | space-between | space-around | space-evenly */
    align-items: center;          /* flex-start | flex-end | center | stretch (default) | baseline */
    align-content: flex-start;    /* like align-items, but for multiple WRAPPED lines */
    gap: 16px;                    /* space between items — much cleaner than margin hacks */
}
```

**`justify-content`** controls alignment along the main axis (horizontal, if `flex-direction: row`).
**`align-items`** controls alignment along the cross axis (vertical, if `flex-direction: row`).

This combination is the single most common way to center something perfectly:

```css
.center-everything {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
}
```

### 11.3 Item properties

```css
.item {
    flex-grow: 1;      /* how much this item grows to fill extra space, relative to siblings */
    flex-shrink: 1;     /* how much this item shrinks when space is tight */
    flex-basis: 200px;  /* the item's starting size before growing/shrinking */
    flex: 1 1 200px;    /* shorthand: grow shrink basis */
    order: 2;           /* visually reorder items without changing HTML */
    align-self: flex-end; /* override align-items for just this one item */
}
```

A very common pattern — three equal-width columns, regardless of content length:

```css
.column {
    flex: 1;   /* shorthand for flex: 1 1 0 — grow equally, shrink equally, ignore content size */
}
```

### 11.4 Real example: a responsive navbar

```html
<nav class="navbar">
    <div class="logo">MySite</div>
    <ul class="nav-links">
        <li><a href="#">Home</a></li>
        <li><a href="#">About</a></li>
        <li><a href="#">Contact</a></li>
    </ul>
</nav>
```

```css
.navbar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem 2rem;
    background: #1a1a2e;
}
.nav-links {
    display: flex;
    gap: 1.5rem;
    list-style: none;
}
.nav-links a {
    color: white;
    text-decoration: none;
}
```

### 11.5 Real example: card row that wraps on small screens

```html
<div class="cards">
    <div class="card">Card 1</div>
    <div class="card">Card 2</div>
    <div class="card">Card 3</div>
</div>
```

```css
.cards {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
}
.card {
    flex: 1 1 250px;  /* grow, shrink, but never smaller than 250px — wraps naturally when space runs out */
    background: white;
    border-radius: 8px;
    padding: 1rem;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}
```

---

## Part 12: CSS Grid (Deep Dive)

Where Flexbox handles one dimension, **Grid handles two** — rows *and* columns simultaneously. Reach for Grid whenever you're laying out a whole page or a genuinely two-dimensional structure (photo galleries, dashboards, magazine-style layouts).

### 12.1 Basic setup

```html
<div class="grid-container">
    <div class="item">1</div>
    <div class="item">2</div>
    <div class="item">3</div>
    <div class="item">4</div>
</div>
```

```css
.grid-container {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;  /* three equal-width columns */
    grid-template-rows: 100px 100px;     /* two rows, each 100px tall */
    gap: 16px;
}
```

`fr` is a special Grid unit meaning "a fraction of the remaining space" — `1fr 1fr 1fr` splits the row into three equal parts, while `2fr 1fr` gives the first column twice the width of the second.

### 12.2 More flexible column definitions

```css
.grid-container {
    display: grid;
    grid-template-columns: 200px 1fr 1fr;      /* fixed sidebar, two flexible columns */
    grid-template-columns: repeat(4, 1fr);      /* shorthand for four equal columns */
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); /* auto-responsive! see below */
}
```

That last line is one of the most useful one-liners in modern CSS — it creates as many columns as will fit, each at least 200px wide, and automatically reflows as the screen resizes, **with no media queries required**:

```css
.gallery {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 1rem;
}
```

### 12.3 Placing items explicitly

```html
<div class="layout">
    <header class="header">Header</header>
    <nav class="sidebar">Sidebar</nav>
    <main class="content">Main content</main>
    <footer class="footer">Footer</footer>
</div>
```

```css
.layout {
    display: grid;
    grid-template-columns: 200px 1fr;
    grid-template-rows: auto 1fr auto;
    grid-template-areas:
        "header  header"
        "sidebar content"
        "footer  footer";
    min-height: 100vh;
    gap: 1rem;
}
.header  { grid-area: header; }
.sidebar { grid-area: sidebar; }
.content { grid-area: content; }
.footer  { grid-area: footer; }
```

`grid-template-areas` is a genuinely readable way to define a whole-page layout — you can visually see the shape of the page in the CSS itself. This is the classic "holy grail" layout (header, sidebar, content, footer) in about ten lines.

You can also place items by line number instead of naming areas:

```css
.item-a {
    grid-column: 1 / 3;   /* start at column line 1, end at column line 3 (spans 2 columns) */
    grid-row: 1 / 2;
}
.item-b {
    grid-column: span 2;  /* shorthand: just span 2 columns from wherever it naturally falls */
}
```

### 12.4 Flexbox vs. Grid — when to use which

| Use Flexbox when... | Use Grid when... |
| --- | --- |
| Laying out items in a single row or column | Laying out a full 2D structure (rows *and* columns) |
| Content size should drive layout | You want explicit control over both dimensions |
| Building a navbar, button group, or toolbar | Building a page layout, image gallery, or dashboard |

They're not competitors — most real sites use both together (Grid for the overall page skeleton, Flexbox for aligning content within individual components).

---

## Part 13: Responsive Design & Media Queries

Responsive design means your layout adapts gracefully to any screen size, from a phone to an ultrawide monitor.

### 13.1 Mobile-first vs. desktop-first

**Mobile-first** (recommended): write your base CSS for small screens, then use `min-width` media queries to add complexity as the screen grows.

```css
/* Base styles: apply to ALL screens, mobile included */
.container {
    display: flex;
    flex-direction: column;
}

/* Tablet and up */
@media (min-width: 768px) {
    .container {
        flex-direction: row;
    }
}

/* Desktop and up */
@media (min-width: 1024px) {
    .container {
        max-width: 1200px;
        margin: 0 auto;
    }
}
```

Common breakpoints as a starting point (adjust based on your actual content, not dogma):

```css
/* Small phones:      up to 480px  (no media query needed — this is the default/base) */
/* Large phones:       480px+ */
/* Tablets:            768px+ */
/* Small laptops:      1024px+ */
/* Desktops:           1280px+ */
```

### 13.2 Media query syntax

```css
@media (min-width: 768px) { /* screen is 768px or wider */ }
@media (max-width: 767px) { /* screen is 767px or narrower */ }
@media (min-width: 768px) and (max-width: 1023px) { /* only tablets */ }
@media (orientation: landscape) { /* wider than tall */ }
@media print { /* styles only applied when printing the page */ }
@media (prefers-color-scheme: dark) { /* user's OS is set to dark mode */
    body { background: #121212; color: #eee; }
}
```

### 13.3 Responsive typography

```css
html {
    font-size: 16px;
}
@media (min-width: 768px) {
    html { font-size: 17px; }
}
@media (min-width: 1200px) {
    html { font-size: 18px; }
}
```

Or use `clamp()` for fully fluid typography that scales continuously instead of jumping at breakpoints:

```css
h1 {
    font-size: clamp(1.75rem, 4vw + 1rem, 3.5rem);
    /* clamp(minimum, preferred, maximum) — never smaller than 1.75rem, never bigger than 3.5rem */
}
```

### 13.4 Responsive images (CSS side)

```css
img {
    max-width: 100%;
    height: auto;   /* prevents images from overflowing their container while keeping proportions */
}
```

This one rule belongs in almost every stylesheet.

### 13.5 Putting it together — a responsive card grid

```css
.cards {
    display: grid;
    grid-template-columns: 1fr;
    gap: 1rem;
}

@media (min-width: 600px) {
    .cards { grid-template-columns: repeat(2, 1fr); }
}

@media (min-width: 1000px) {
    .cards { grid-template-columns: repeat(3, 1fr); }
}
```

(Recall from Part 12 that `repeat(auto-fit, minmax(...))` can often achieve similar results with *zero* media queries — worth trying that approach first.)

---

## Part 14: Pseudo-Classes & Pseudo-Elements

**Pseudo-classes** target elements in a particular *state*. **Pseudo-elements** target a specific *part* of an element that isn't a real HTML tag.

### 14.1 Common pseudo-classes

```css
a:hover      { color: red; }               /* mouse is over the element */
a:active     { color: darkred; }           /* being clicked, mid-click */
a:visited    { color: purple; }            /* link has been visited before */
input:focus  { outline: 2px solid blue; }  /* element currently has keyboard focus */
input:disabled { opacity: 0.5; }
input:checked + label { font-weight: bold; } /* pairs well with custom checkboxes */
input:invalid { border-color: red; }       /* HTML5 validation state */

li:first-child { font-weight: bold; }
li:last-child  { border-bottom: none; }
li:nth-child(odd)  { background: #f9f9f9; }  /* zebra-striping */
li:nth-child(even) { background: #ffffff; }
li:nth-child(3)    { color: red; }           /* exactly the third item */

p:empty { display: none; }                 /* targets elements with no content at all */
:not(.excluded) { color: gray; }           /* everything EXCEPT elements with class "excluded" */
```

### 14.2 Common pseudo-elements

```css
p::first-line {
    font-weight: bold;
}
p::first-letter {
    font-size: 2em;
    float: left;
}
::selection {
    background: gold;   /* styles the text a user highlights with their mouse */
}
.tooltip::before {
    content: "→ ";
}
.required::after {
    content: " *";
    color: red;
}
```

`::before` and `::after` are extremely common for adding decorative content (icons, quotation marks, arrows) purely through CSS, without cluttering the HTML. They require a `content` property to render at all — even if it's just `content: "";`.

### 14.3 Practical example: custom-styled button

```css
.btn {
    background: #4361ee;
    color: white;
    padding: 12px 24px;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    transition: background 0.2s ease;
}
.btn:hover {
    background: #3651d4;
}
.btn:active {
    transform: scale(0.98);
}
.btn:disabled {
    background: #ccc;
    cursor: not-allowed;
}
```

---

## Part 15: Transitions, Transforms & Animations

### 15.1 Transitions — smooth changes between states

```css
.box {
    background: #4361ee;
    transition: background 0.3s ease, transform 0.3s ease;
    /* shorthand: property duration easing-function delay */
}
.box:hover {
    background: #f72585;
    transform: scale(1.05);
}
```

Common easing functions: `ease`, `linear`, `ease-in`, `ease-out`, `ease-in-out`, or a custom `cubic-bezier(...)`.

You can transition `all` properties, but it's better practice (and better for performance) to name the specific properties you're animating.

### 15.2 Transforms

```css
.rotate    { transform: rotate(45deg); }
.scale     { transform: scale(1.2); }
.translate { transform: translate(20px, 10px); }
.skew      { transform: skew(10deg, 0deg); }
.combo     { transform: translateX(20px) rotate(10deg) scale(1.1); } /* combine multiple */

.centered-absolute {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%); /* perfectly centers an absolutely positioned element, regardless of its size */
}
```

`transform` and `opacity` are the two properties browsers can animate most cheaply (they don't trigger layout recalculation), so prefer them over animating `top`/`left`/`width`/`height` when performance matters.

### 15.3 Keyframe animations

```css
@keyframes bounce {
    0%   { transform: translateY(0); }
    50%  { transform: translateY(-20px); }
    100% { transform: translateY(0); }
}

.bouncing-ball {
    animation: bounce 1s ease-in-out infinite;
    /* shorthand: name duration easing iteration-count direction */
}
```

```css
@keyframes fadeIn {
    from { opacity: 0; }
    to   { opacity: 1; }
}
.fade-in-element {
    animation: fadeIn 0.5s ease-out forwards;
    /* "forwards" keeps the final keyframe's styles after the animation ends */
}
```

Full animation shorthand properties:

```css
.element {
    animation-name: bounce;
    animation-duration: 1s;
    animation-timing-function: ease-in-out;
    animation-delay: 0.2s;
    animation-iteration-count: infinite;   /* or a number like 3 */
    animation-direction: alternate;         /* normal | reverse | alternate | alternate-reverse */
    animation-fill-mode: forwards;          /* none | forwards | backwards | both */
}
```

### 15.4 Respecting motion preferences

Some users get dizzy or uncomfortable from motion. Respect their OS-level setting:

```css
@media (prefers-reduced-motion: reduce) {
    * {
        animation-duration: 0.01ms !important;
        transition-duration: 0.01ms !important;
    }
}
```

---

## Part 16: CSS Custom Properties (Variables)

Custom properties let you define reusable values, dramatically simplifying theming and maintenance.

```css
:root {
    --primary-color: #4361ee;
    --secondary-color: #f72585;
    --font-body: "Inter", sans-serif;
    --spacing-unit: 8px;
    --border-radius: 6px;
}

.btn {
    background: var(--primary-color);
    font-family: var(--font-body);
    padding: calc(var(--spacing-unit) * 2) calc(var(--spacing-unit) * 3);
    border-radius: var(--border-radius);
}

.btn-secondary {
    background: var(--secondary-color);
}
```

`var(--name, fallback)` lets you supply a default if the variable isn't set:

```css
.box {
    color: var(--text-color, black);
}
```

### 16.1 Why this matters: instant dark mode

Because custom properties can be *redefined* in a different scope, you can build a full theme switcher with almost no extra code:

```css
:root {
    --bg: #ffffff;
    --text: #1a1a1a;
}

[data-theme="dark"] {
    --bg: #121212;
    --text: #eaeaea;
}

body {
    background: var(--bg);
    color: var(--text);
    transition: background 0.3s, color 0.3s;
}
```

```html
<body data-theme="dark">
```

Toggling that one `data-theme` attribute (with a tiny bit of JavaScript) restyles the entire page instantly — no duplicated CSS rules needed.

### 16.2 Scoped variables

Custom properties respect the cascade like anything else — define one inside a specific component to override it just there:

```css
.card {
    --card-padding: 1rem;
    padding: var(--card-padding);
}
.card.compact {
    --card-padding: 0.5rem;
}
```

---

## Part 17: The Cascade, Specificity & Inheritance

This is the part that explains *why* your CSS sometimes "doesn't work" — usually it's working exactly as designed, just not how you expected.

### 17.1 Specificity

When multiple rules target the same element, the browser picks a winner using **specificity** — essentially a points system:

| Selector type | Points |
| --- | --- |
| Inline style (`style="..."`) | 1000 |
| ID (`#id`) | 100 |
| Class, attribute, pseudo-class (`.class`, `[type]`, `:hover`) | 10 |
| Element, pseudo-element (`div`, `::before`) | 1 |
| Universal selector (`*`) | 0 |

```css
p { color: blue; }                 /* specificity: 0,0,0,1 → loses */
.text { color: green; }            /* specificity: 0,0,1,0 → wins over element selector */
#intro { color: red; }             /* specificity: 0,1,0,0 → wins over class */
p.text#intro { color: purple; }    /* specificity: 0,1,1,1 → wins over everything above */
```

```html
<p id="intro" class="text">What color am I? (Answer: purple — highest specificity wins)</p>
```

When specificity ties, **the rule that appears later in the stylesheet wins**. This is the "cascading" in Cascading Style Sheets.

### 17.2 `!important` — use sparingly

```css
p { color: blue !important; }
```

This overrides *any* other rule regardless of specificity (except another `!important` with higher specificity). It's a blunt instrument — reach for it only as a last resort, since it makes future overrides much harder. Fix specificity conflicts properly instead of reaching for `!important` as a default habit.

### 17.3 Inheritance

Some CSS properties automatically pass down from parent to child (typography-related ones, mostly); others don't (box-model related ones, mostly).

```css
body {
    color: #333;         /* INHERITED — all text inside body will be #333 unless overridden */
    font-family: sans-serif; /* INHERITED */
}
div {
    border: 1px solid black; /* NOT inherited — child divs need their own border rule */
    padding: 10px;            /* NOT inherited */
}
```

You can force inheritance explicitly:

```css
a {
    color: inherit; /* take on the parent's color instead of the browser's default blue */
}
```

Other useful global keywords: `initial` (reset to the property's default), `unset` (acts like `inherit` for inherited properties, `initial` otherwise), `revert` (roll back to the browser's built-in stylesheet).

---

## Part 18: Modern CSS Layout Tools

A few newer tools worth knowing as you go from beginner to advanced:

### 18.1 `clamp()`, `min()`, `max()`

```css
.container {
    width: clamp(300px, 80%, 1200px); /* fluid width with hard min/max bounds */
    padding: min(5vw, 40px);           /* whichever is smaller */
    font-size: max(1rem, 2vw);         /* whichever is larger */
}
```

### 18.2 Container queries (style based on a parent's size, not the viewport)

```css
.card-wrapper {
    container-type: inline-size;
    container-name: card;
}

@container card (min-width: 400px) {
    .card {
        display: flex;
        gap: 1rem;
    }
}
```

This is a genuine leap for component-based design — a card can respond to *its own* available space (e.g., inside a narrow sidebar vs. a wide main column) rather than only the overall browser window.

### 18.3 `:has()` — the "parent selector"

```css
/* Style a .card only if it CONTAINS an image */
.card:has(img) {
    padding-top: 0;
}

/* Style a form group when its input is invalid */
.form-group:has(input:invalid) {
    border-color: red;
}
```

For years CSS couldn't select a parent based on its children — `:has()` finally makes that possible.

### 18.4 Logical properties (better for internationalization)

```css
.box {
    margin-inline: 1rem;   /* left+right in LTR languages, right+left in RTL — adapts automatically */
    padding-block: 2rem;   /* top+bottom */
}
```

---

## Part 19: Best Practices, Naming & Architecture

### 19.1 BEM naming convention

BEM (Block, Element, Modifier) keeps large stylesheets predictable and avoids specificity wars:

```css
/* Block */
.card { }

/* Element (a part of the block) — double underscore */
.card__title { }
.card__image { }
.card__button { }

/* Modifier (a variant of the block or element) — double hyphen */
.card--featured { }
.card__button--disabled { }
```

```html
<div class="card card--featured">
    <img class="card__image" src="photo.jpg" alt="...">
    <h3 class="card__title">Product Name</h3>
    <button class="card__button card__button--disabled">Sold Out</button>
</div>
```

This isn't the only convention (utility-first approaches like Tailwind CSS are hugely popular too), but understanding BEM makes any team's CSS easier to read.

### 19.2 Organizing a stylesheet

A reasonable order to keep things predictable:

```css
/* 1. Reset / normalize */
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

/* 2. CSS custom properties / variables */
:root { --primary: #4361ee; }

/* 3. Base element styles */
body { font-family: sans-serif; line-height: 1.6; }
a { color: inherit; text-decoration: none; }

/* 4. Layout components (header, nav, footer, grid containers) */
.container { max-width: 1200px; margin: 0 auto; }

/* 5. Reusable components (buttons, cards, forms) */
.btn { ... }
.card { ... }

/* 6. Page-specific / utility overrides */
.text-center { text-align: center; }
```

### 19.3 General principles

- **Mobile-first.** Write base styles for small screens; add complexity with `min-width` media queries.
- **Avoid deep nesting / overly specific selectors.** `.sidebar ul li a span` is fragile and hard to override — a single well-named class is almost always better.
- **Don't use IDs for styling.** Reserve `#id` for JavaScript hooks and anchor links; style with classes.
- **Keep specificity flat.** If everything is roughly the same specificity, the cascade behaves predictably (last rule wins) and you rarely need `!important`.
- **Use a CSS reset or sensible defaults** (the `box-sizing: border-box` rule from Part 7 is the single most valuable line you can add).
- **Comment your CSS** for anything non-obvious — future you (or a teammate) will thank you.
- **Validate your HTML** occasionally with the [W3C validator](https://validator.w3.org/) — it catches unclosed tags and structural mistakes early.
- **Test in the browser's dev tools.** Right-click → Inspect lets you live-edit CSS, toggle device sizes, and check accessibility/contrast — this is how professionals actually debug layouts, far more than by re-reading code.

---

## Part 20: Full Project Walkthrough

Let's build a small, real, responsive landing page that uses almost everything covered above: semantic HTML, Flexbox, Grid, custom properties, media queries, and transitions.

**`index.html`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Aurora — Landing Page</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <header class="navbar">
        <div class="navbar__logo">Aurora</div>
        <nav>
            <ul class="navbar__links">
                <li><a href="#features">Features</a></li>
                <li><a href="#pricing">Pricing</a></li>
                <li><a href="#contact">Contact</a></li>
            </ul>
        </nav>
    </header>

    <main>
        <section class="hero">
            <h1 class="hero__title">Build faster. Ship sooner.</h1>
            <p class="hero__subtitle">Aurora helps small teams launch products without the busywork.</p>
            <a href="#pricing" class="btn btn--primary">Get Started</a>
        </section>

        <section id="features" class="features">
            <h2 class="section-title">Why Aurora</h2>
            <div class="features__grid">
                <article class="feature-card">
                    <h3>Fast Setup</h3>
                    <p>Go from zero to deployed in minutes, not days.</p>
                </article>
                <article class="feature-card">
                    <h3>Reliable</h3>
                    <p>Built on infrastructure trusted by thousands of teams.</p>
                </article>
                <article class="feature-card">
                    <h3>Affordable</h3>
                    <p>Transparent pricing with no surprise fees.</p>
                </article>
            </div>
        </section>
    </main>

    <footer class="site-footer">
        <p>&copy; 2026 Aurora Inc. All rights reserved.</p>
    </footer>
</body>
</html>
```

**`style.css`**

```css
/* ---- Reset & variables ---- */
*, *::before, *::after {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
}

:root {
    --primary: #4361ee;
    --primary-dark: #3651d4;
    --text: #1a1a2e;
    --text-muted: #666;
    --bg-light: #f7f8fc;
    --radius: 10px;
    --max-width: 1100px;
}

body {
    font-family: "Segoe UI", Arial, sans-serif;
    color: var(--text);
    line-height: 1.6;
}

a { color: inherit; text-decoration: none; }
img { max-width: 100%; height: auto; }

/* ---- Navbar (Flexbox) ---- */
.navbar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1.25rem 2rem;
    max-width: var(--max-width);
    margin: 0 auto;
}
.navbar__logo {
    font-weight: 700;
    font-size: 1.5rem;
    color: var(--primary);
}
.navbar__links {
    display: flex;
    gap: 2rem;
    list-style: none;
}
.navbar__links a {
    transition: color 0.2s ease;
}
.navbar__links a:hover {
    color: var(--primary);
}

/* ---- Hero ---- */
.hero {
    text-align: center;
    padding: 5rem 1.5rem;
    background: linear-gradient(180deg, var(--bg-light), #ffffff);
}
.hero__title {
    font-size: clamp(2rem, 5vw, 3.5rem);
    margin-bottom: 1rem;
}
.hero__subtitle {
    color: var(--text-muted);
    font-size: 1.1rem;
    max-width: 500px;
    margin: 0 auto 2rem;
}

.btn {
    display: inline-block;
    padding: 0.9rem 2rem;
    border-radius: var(--radius);
    font-weight: 600;
    transition: background 0.2s ease, transform 0.15s ease;
}
.btn--primary {
    background: var(--primary);
    color: white;
}
.btn--primary:hover {
    background: var(--primary-dark);
    transform: translateY(-2px);
}

/* ---- Features (Grid) ---- */
.features {
    max-width: var(--max-width);
    margin: 0 auto;
    padding: 4rem 1.5rem;
}
.section-title {
    text-align: center;
    font-size: 2rem;
    margin-bottom: 2.5rem;
}
.features__grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
    gap: 1.5rem;
}
.feature-card {
    background: var(--bg-light);
    border-radius: var(--radius);
    padding: 1.75rem;
    transition: transform 0.2s ease, box-shadow 0.2s ease;
}
.feature-card:hover {
    transform: translateY(-4px);
    box-shadow: 0 8px 20px rgba(0,0,0,0.08);
}
.feature-card h3 {
    color: var(--primary);
    margin-bottom: 0.5rem;
}

/* ---- Footer ---- */
.site-footer {
    text-align: center;
    padding: 2rem;
    color: var(--text-muted);
    font-size: 0.9rem;
}

/* ---- Responsive tweaks ---- */
@media (max-width: 600px) {
    .navbar {
        flex-direction: column;
        gap: 1rem;
    }
    .navbar__links {
        gap: 1rem;
    }
}
```

Try this yourself: save both files in the same folder, open `index.html` in a browser, then resize the window and watch the `features__grid` reflow automatically (thanks to `auto-fit`/`minmax`) and the navbar stack vertically below 600px. From here, experiment: change the color variables in `:root`, add a dark mode toggle using the technique from Part 16, or convert the hero into a two-column layout with an image using Flexbox.

---

## Part 21: Where to Go Next

You now have a solid, practical foundation covering everything from a single `<p>` tag to Grid-based responsive layouts and CSS variables. A few directions to keep building:

- **JavaScript** — to make pages interactive (form validation, dynamic content, animations triggered by user actions).
- **CSS preprocessors** (Sass/SCSS) — nesting, mixins, and variables with extra power, though native CSS has closed much of this gap in recent years.
- **A CSS framework** like Tailwind CSS (utility-first) or Bootstrap (component-based) — useful once you understand the fundamentals well enough to know what the framework is doing for you.
- **Accessibility (a11y) deep dive** — WCAG guidelines, screen reader testing, keyboard navigation.
- **Browser DevTools mastery** — the Elements/Inspector panel, the Network tab, and Lighthouse audits will make you dramatically faster at debugging.
- **Version control (Git)** — essential once you're managing real projects.
- **[MDN Web Docs](https://developer.mozilla.org/)** — the best reference documentation for every HTML/CSS feature; bookmark it and search it constantly. That's not cheating — it's what working developers do daily.

The best next step is simply to build something real: clone the look of a site you like, build a personal portfolio page, or recreate a small piece of an app's UI. Reading teaches vocabulary; building is what makes it stick.
