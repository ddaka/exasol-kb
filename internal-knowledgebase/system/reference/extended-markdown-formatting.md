---
tool_name: internal-knowledgebase
doc_type: reference
category: system
title: "Extended markdown formatting"
summary: "Reference for markdown syntax compatibility across VS Code, GitHub, and Salesforce (MadCap Flare pipeline)."
---
# Extended markdown formatting

## Scope

This reference is for internal knowledge-base authors.

Articles are written in Markdown and converted to HTML through MadCap Flare before publication to Salesforce. Not all markdown extensions render consistently across toolchains.

## Compatibility matrix

| Feature | VS Code | GitHub | Salesforce |
| --- | --- | --- | --- |
| Table alignment (`:---`, `:---:`, `---:`) | Supported | Supported | Supported |
| Footnotes (`[^id]`) | Supported | Supported | Supported (link behavior limited) |
| Reusable link definitions (`[text]: URL`) | Supported | Supported | Supported (title/tooltip may be dropped) |
| Subscript/superscript (`~sub~`, `^sup^`) | Supported | Not supported | Not supported |
| Highlight (`==text==`) | Supported | Not supported | Not supported |
| Custom heading IDs (`## H {#id}`) | Supported | Not supported | Not supported |
| HTML comments (`<!-- -->`) | Supported | Supported | Supported |
| Unused refs/definitions as comments | Supported | Supported | Supported |

## Recommended usage

1. Prefer core markdown features for customer-facing content.
1. Use compatibility-sensitive extensions only when required.
1. Validate rendering in Salesforce before publishing.

## Tables: column alignment

Use alignment markers in the separator row:

```markdown
| left | center | right |
| :--- | :---: | ---: |
| a | b | c |
```

## Footnotes and reusable links

Footnotes and reusable link definitions are useful for readability in source files.

```markdown
This is text with a footnote[^note].

[^note]: Footnote content.
```

```markdown
See [Exasol Docs] for details.

[Exasol Docs]: https://docs.exasol.com "Docs"
```

Salesforce may not preserve all local-link or tooltip behavior.

## Comments in markdown source

Preferred form:

```markdown
<!-- internal author note -->
```

Do not include `--` inside an HTML comment body.

## Using unused references as non-rendered notes

Unused link/reference definitions can act as source-only notes, but markdownlint rule `MD053` may trigger.

Example controls:

```markdown
<!-- markdownlint-disable-next-line MD053 -->
[^unused]: author-only note
```

```markdown
<!-- markdownlint-disable MD053 -->
[//]: # (author-only note)
<!-- markdownlint-enable MD053 -->
```

## Authoring guidelines

- Keep syntax portable unless there is a clear reason not to.
- Avoid relying on custom anchors for Salesforce navigation.
- Keep hidden notes non-sensitive even if not rendered.

## References

- Markdown basic syntax: <https://www.markdownguide.org/basic-syntax/>
- Markdown extended syntax: <https://markdownguide.offshoot.io/extended-syntax/>
- MadCap Flare: <https://www.madcapsoftware.com/products/flare/>


