---
name: notes-markdown-authoring
description: Write or revise Markdown notes in this repository, including Tatum rendering, KaTeX maths, derivations, and the custom proof layout. Use for files under content/ or examples/ and when adding a note to the rendered site.
---

# Write notes for this repository

Write source material in `content/**/*.md` or examples in `examples/**/*.md`. Preserve the surrounding file's terminology, heading depth, notation, and level of detail. Do not edit generated `notes/**/*.html` or `structure.json`; the build recreates them.

The site is rendered by Tatum with `.tatum/bluetot`. Markdown may contain raw HTML because the proof and indentation components depend on it.

## Markdown and maths

- Use ordinary Markdown for prose, headings, lists, tables, code, links, and emphasis.
- Leave blank lines around raw HTML blocks, display-math blocks, and the Markdown contained inside HTML blocks. This lets the Markdown parser recognise their contents reliably.
- Use `$...$` for inline maths and a `$$` block for display maths. Put each display delimiter on its own line when the expression spans multiple lines.
- Keep explanatory prose in Markdown. Use `\text{...}` only for short labels or annotations inside a formula.
- Do not put a whole proof in one maths block to obtain a mathematical font. The `.proof` component already uses the KaTeX text font and keeps inline maths at the same apparent size as its prose.

The configured convenience macros are:

| Macro | Meaning |
| --- | --- |
| `\N`, `\Z`, `\Q`, `\R`, `\C` | Standard blackboard-bold number sets |
| `\vv{x}` | `\vec{x}` |
| `\bb{x}` | Bold maths text |
| `\derivation{...}` | An aligned, automatically left-aligned derivation |

This renderer uses KaTeX, not a complete LaTeX installation. Do not add `\usepackage` declarations or assume commands supplied by arbitrary LaTeX packages are available. Commands such as `\centernot` and `\underleftrightarrow` must be checked against the configured KaTeX version in an actual rendered page. If a command fails, prefer a supported equivalent such as `\not\le` or `\xleftrightarrow{R}` where it preserves the intended notation; do not silently change the mathematical meaning.

## Derivations

Use `\derivation` for a multi-line calculation or implication chain:

```markdown
$$
\derivation{
f(n+1)
&= f(n) + 2n + 1 \\
&= n^2 + 2n + 1
&& \text{by the induction hypothesis} \\
&= (n+1)^2.
}
$$
```

Place `&` before the relation on each aligned line and use `\\` between lines. A second `&&` column may hold a short justification. The page script recognises a display block whose content starts with `\derivation{`, adds the `.math-left` and `.math-derivation` classes, and makes it horizontally scrollable when necessary. Do not wrap a derivation in `<div class="math-left">`.

Ordinary `$$...$$` display maths remains centred. For a non-derivation display that genuinely needs left alignment, the legacy wrapper remains available:

```markdown
<div class="math-left">

$$
P(A \mid B) = \frac{P(A \cap B)}{P(B)}
$$

</div>
```

Prefer `\derivation` whenever the content is a sequence of aligned steps.

## Proofs

Use `.proof` for the visual proof container. Put the theorem or claim and the `Proof.` line at the outer level, then put the proof's body in `.proof-body`:

```markdown
<div class="proof">

**Theorem.** For every $n \in \mathbb{N}$, the stated property holds.

**Proof.**

<div class="proof-body">

Start with prose in normal Markdown and use inline maths such as $n \in \mathbb{N}$.

$$
\derivation{
a
&= b \\
&= c.
}
$$

Therefore the result follows. $\square$

</div>

</div>
```

The outer component supplies the border, background, mathematical text face, and compact proof line spacing. `.proof-body` indents everything after the `Proof.` line. Keep the theorem outside `.proof-body`.

For cases, inclusions, subarguments, or other multiple indentation levels, nest another `.proof-body`. There is no fixed nesting limit, but avoid nesting that does not represent a real logical level:

```markdown
**Case 1.** Suppose $x \in A$.

<div class="proof-body">

Explain this case, including prose or a `\derivation` block.

</div>
```

Every opening `<div>` must have a matching closing `</div>`. Retain blank lines immediately inside each wrapper so Markdown paragraphs and maths are parsed rather than emitted as literal text.

Use `examples/proof.md` as the canonical source of complete proof patterns before inventing another wrapper or class.

## Adding and checking notes

When adding a new source note that belongs on the site, add its source-to-output mapping to `.tatum/render-list.json`. Keep the output beneath `notes/` and give it an `.html` extension.

After edits that affect rendering, run from the repository root:

```sh
make clean && make build
```

Confirm that the expected HTML file was produced. For maths or layout changes, inspect the rendered page in a browser as well: maths is rendered client-side, and failures receive the `.math-error` class plus a browser-console error and tooltip. Check proof indentation, line spacing, horizontal overflow, and KaTeX errors rather than relying only on a successful Tatum command.

When introducing or changing a custom maths macro, update both `.tatum/bluetot/macros.tex` and `.tatum/bluetot/katex-macros.js` with equivalent definitions. Build-time Tatum parsing and browser-side KaTeX use those separate sources; changing only one can make preview and final rendering disagree.
