# Knowledgebase

A Markdown-based wiki storing knowledge items in a graph/hierarchical document structure.

---

## Structural Rules for Contributors

### 1. Directory Structure & Indexes

Every folder — **including the root** — **must** contain an `index.md` file.

The `index.md` is used **exclusively** to map relationships and hierarchy. It must contain a generated list / table of contents that links to:

- All **sub-folders** within that specific folder.
- All **regular documents** within that specific folder.

Do **not** place primary knowledge content inside `index.md`; keep it as a navigation file only.

### 2. Regular Documents

Regular documents are the primary knowledge items. Guidelines:

- Use standard Markdown for prose content.
- Use Markdown tables for dataset/tabular content.
- **Cross-reference** other documents in this repository using **relative paths**, e.g.:

  ```markdown
  See also: [API Reference](../engineering/api-reference.md)
  ```

- **External links** use standard Markdown hyperlinks:

  ```markdown
  [MDN Web Docs](https://developer.mozilla.org)
  ```

### 3. Media Handling

- All images and diagrams **must** be stored locally inside the `/assets` directory (sub-folders are allowed for organisation).
- Reference media **inline** within the relevant document using a relative path:

  ```markdown
  ![Architecture Diagram](../assets/architecture-diagram.svg)
  ```

- Do **not** hotlink external images — download and commit them to `/assets` first.

---

## Repository Layout

```
/
├── README.md               ← project overview & contributor rules (this file)
├── index.md                ← root navigation index
├── assets/
│   ├── index.md            ← assets folder index
│   └── *.svg / *.png …     ← locally stored media files
├── engineering/
│   ├── index.md            ← engineering section index
│   └── *.md                ← engineering knowledge documents
└── datasets/
    ├── index.md            ← datasets section index
    └── *.md                ← dataset documents
```

---

## Quick-Start

1. Clone the repository.
2. Browse `index.md` at the root for a full table of contents.
3. Navigate into any folder and open its `index.md` to see the local hierarchy.
4. Add new documents alongside an updated `index.md` in the same folder.
