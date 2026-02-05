# Blog Writing Guide

This folder contains blog posts for dylanoconnell.com.

## Quick Start

1. Create a folder with a URL-friendly slug: `blog/my-post-name/`
2. Copy the template below into `blog/my-post-name/index.html`
3. Write your content in the `<div class="post-content">` section
4. Add images to `blog/my-post-name/images/`
5. Add a link to the post in the main `index.html` (Miscellany section)

## Folder Structure

```
blog/
└── my-post-name/
    ├── index.html    # Your post
    └── images/       # Images, GIFs, etc.
        ├── photo.jpg
        └── diagram.png
```

## Post Template

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Your Title - Dylan O'Connell</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=IBM+Plex+Sans:wght@400;500;600&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="../../styles.css">
    <!-- KaTeX for math -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/katex.min.css">
    <script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/katex.min.js"></script>
    <script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/contrib/auto-render.min.js"
        onload="renderMathInElement(document.body);"></script>
</head>
<body>
    <main class="blog-post">
        <article>
            <header class="post-header">
                <h1>Your Title Here</h1>
                <p class="post-meta">Month Day, Year</p>
            </header>
            <div class="post-content">

<h1>First Section</h1>
<p>Your content here...</p>

            </div>
        </article>
        <nav class="post-nav">
            <a href="../../index.html">&larr; Home</a>
        </nav>
    </main>
</body>
</html>
```

## Writing Content

### Basic HTML

```html
<h1>Section Heading</h1>
<p>A paragraph of text with <em>italics</em> and <strong>bold</strong>.</p>

<ul>
    <li>Bullet point</li>
    <li>Another point</li>
</ul>

<ol>
    <li>Numbered item</li>
    <li>Second item</li>
</ol>

<blockquote>
    <p>A quote from someone wise.</p>
</blockquote>

<a href="https://example.com">Link text</a>
```

### Images

Put images in the `images/` subfolder, then reference them:

```html
<p><img src="images/my-photo.jpg" alt="Description of image"></p>
```

Images automatically get proper styling (max-width 100%, centered, margins).

### Math with LaTeX

KaTeX renders LaTeX automatically. No escaping needed for most symbols.

**Inline math** (within a sentence):
```html
The equation \(E = mc^2\) changed physics.
```

**Display math** (centered on its own line):
```html
\[
\int_0^\infty e^{-x^2} dx = \frac{\sqrt{\pi}}{2}
\]
```

**Common LaTeX examples:**
- Fractions: `\frac{a}{b}`
- Subscripts/superscripts: `x_i^2`
- Greek letters: `\alpha`, `\beta`, `\gamma`
- Sums: `\sum_{i=1}^n x_i`
- Square root: `\sqrt{x}`
- Matrices: `\begin{pmatrix} a & b \\ c & d \end{pmatrix}`

### Footnotes

```html
<p>Some text with a footnote.<a href="#fn1" class="footnote-ref" id="fnref1"><sup>1</sup></a></p>

<!-- At the bottom of post-content, before closing div -->
<div class="footnotes">
<hr>
<ol>
<li id="fn1"><p>The footnote text.<a href="#fnref1" class="footnote-back">↩︎</a></p></li>
</ol>
</div>
```

### Code

Inline code:
```html
Use the <code>printf()</code> function.
```

Code blocks (no syntax highlighting, just monospace):
```html
<pre><code>def hello():
    print("Hello, world!")
</code></pre>
```

## Adding to Homepage

After creating your post, add it to `index.html` in the Miscellany section:

```html
<section>
    <h2>Miscellany</h2>
    <ul class="project-list">
        <li>
            <a href="blog/my-post-name/index.html">My Post Title</a>
        </li>
        <!-- other items -->
    </ul>
</section>
```

## Testing Locally

Open the HTML file directly in your browser, or run a local server:

```bash
cd personal_site
python -m http.server 8000
```

Then visit `http://localhost:8000/blog/my-post-name/`

## Deploying

```bash
git add blog/my-post-name/
git add index.html  # if you added the link
git commit -m "Add blog post: My Post Title"
git push
```

The post will be live at `https://dylanpotteroconnell.github.io/blog/my-post-name/`
