---
layout: post
title: Snippets for Markdown to LaTeX with Pandoc
commentIssueId: 16
tags: markdown, pandoc, latex
---

This is another living post that serves more for my personal benefit than anything else. By living, I mean that I will continue to update it over time as I find things I want to keep easily accessible. This post is a place to paste snippets and boiler plate of Markdown/Pandoc/LaTeX material that I find useful but often forget. There's probably nothing very genius or awe-inspiring here, but hopefully I won't have to `grep` through by bash history as often, trying to remember what I did that one time to do that one thing.

I love using Markdown[^1] for a variety of tasks, and if I need to present my work elsewhere (like homework assignments, distribution of meeting notes, etc.) I use Pandoc[^2] to convert the Markdown into $$\LaTeX$$, which makes some lazy writing look much more professional, in my opinion. 

---

## Switch back and forth between multiple columns in Pandoc markdown

Include the following in the YAML header:

```yaml
header-includes:
  - \usepackage{multicol}
  - \newcommand{\hideFromPandoc}[1]{#1}\hideFromPandoc{\let\Begin\begin\let\End\end}
```

Then begin and end sections of multiple columns like so:

```latex
\Begin{multicols}{2} 

#### Sample Equations

\begin{equation}
\frac{dx_1}{dt} = x_2^2 + x_1^3
\end{equation}

\begin{equation}
\frac{dx_2}{dt} = \frac{1}{x_1} \cdot x_2 
\end{equation}

\End{multicols}
```

---

## Options commonly specified in header

Include in the YAML header as needed:

```yaml
title: My awesome title
subtitle: and cool subtitle
date: \today # Note: Only works on Linux/OSX going from Markdown -> PDF
geometry: margin=1in
classoption=twocolumn
```

---

## Output complete LaTeX source for a document

Running the following only outputs the LaTeX source for the content in the document:

```terminal
$ pandoc source.md -o output.tex
```

Running this inserts the LaTeX content into your default (or other specified template) and then outputs the complete LaTeX source for generating the document:

```terminal
$ pandoc source.md -o output.tex  --template default.latex
```

{% include footnotes.md %}

[^1]: [https://daringfireball.net/projects/markdown/](https://daringfireball.net/projects/markdown/) <br/> [https://en.wikipedia.org/wiki/Markdown](https://en.wikipedia.org/wiki/Markdown)
[^2]: [https://pandoc.org/](https://pandoc.org/)