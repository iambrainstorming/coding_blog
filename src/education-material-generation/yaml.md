# Single Source Format for All Publishing

Date: 03-06-2026 

A single source format can drive:

- Printed books (PDF, EPUB)
- HTML books with quizzes
- TTS audiobook generation
- Video generation with images, animations, and narration
- Mobile apps and web apps

Recommendation: YAML metadata


A practical architecture would look like this:

```text
book.yaml
    ↓
serde_yaml
    ↓
Rust structs
    ↓
Grade validation
    ↓
XML inline parser
    ↓
Document AST
    ↓
HTML Generator
Typst Generator
TTS Generator
```

---

# 1. YAML

```yaml
title: Biology
grade: 11
language: en

chapters:
  - id: cell
    title: Cell

    sections:
      - title: Introduction

        content:
          - type: paragraph
            text: |
              Cells are the <b>basic unit of life</b>.

              The <i>cell membrane</i> protects the cell.

              <color value="red">DNA</color> stores genetic information.

          - type: image
            src: images/cell.png
            alt: Animal Cell

          - type: quiz
            question: What is the basic unit of life?

            options:
              - Cell
              - Tissue
              - Organ

            answer: 0
```

---

# 2. Rust Structures

```rust
use serde::Deserialize;
use garde::Validate;

#[derive(Debug, Deserialize, Validate)]
pub struct Book {
    #[garde(length(min = 1))]
    pub title: String,

    #[garde(range(min = 1, max = 12))]
    pub grade: u8,

    pub language: String,

    pub chapters: Vec<Chapter>,
}

#[derive(Debug, Deserialize)]
pub struct Chapter {
    pub id: String,
    pub title: String,
    pub sections: Vec<Section>,
}

#[derive(Debug, Deserialize)]
pub struct Section {
    pub title: String,
    pub content: Vec<Content>,
}

#[derive(Debug, Deserialize)]
#[serde(tag = "type")]
pub enum Content {
    #[serde(rename = "paragraph")]
    Paragraph {
        text: String,
    },

    #[serde(rename = "image")]
    Image {
        src: String,
        alt: String,
    },

    #[serde(rename = "quiz")]
    Quiz {
        question: String,
        options: Vec<String>,
        answer: usize,
    },
}
```

---

# 3. Grade Validation

```rust
use garde::Validate;

let yaml = std::fs::read_to_string("book.yaml")?;

let book: Book = serde_yaml::from_str(&yaml)?;

book.validate()?;
```

Fails:

```yaml
grade: 15
```

Error:

```text
grade must be between 1 and 12
```

---

# 4. Parse Inline XML

Wrap text inside a root tag:

```rust
let wrapped = format!("<root>{}</root>", text);
```

Parse with:

```toml
roxmltree = "0.20"
```

Example:

```rust
use roxmltree::Document;

let doc = Document::parse(&wrapped)?;
```

---

# 5. Internal AST

Convert XML to:

```rust
pub enum Inline {
    Text(String),

    Bold(Vec<Inline>),

    Italic(Vec<Inline>),

    Color {
        color: String,
        children: Vec<Inline>,
    },
}
```

Example:

```xml
Cells are the <b>basic unit of life</b>.
```

becomes:

```rust
[
    Text("Cells are the "),
    Bold([
        Text("basic unit of life")
    ]),
    Text(".")
]
```

---

# 6. HTML Generator

```rust
fn html_inline(node: &Inline) -> String {
    match node {
        Inline::Text(t) => t.clone(),

        Inline::Bold(children) => {
            format!(
                "<strong>{}</strong>",
                children.iter()
                    .map(html_inline)
                    .collect::<String>()
            )
        }

        Inline::Italic(children) => {
            format!(
                "<em>{}</em>",
                children.iter()
                    .map(html_inline)
                    .collect::<String>()
            )
        }

        Inline::Color {
            color,
            children
        } => {
            format!(
                "<span style=\"color:{}\">{}</span>",
                color,
                children.iter()
                    .map(html_inline)
                    .collect::<String>()
            )
        }
    }
}
```

Output:

```html
<p>
Cells are the
<strong>basic unit of life</strong>.
</p>

<p>
<em>cell membrane</em>
</p>

<p>
<span style="color:red">
DNA
</span>
</p>
```

---

# 7. Typst Generator

```rust
fn typst_inline(node: &Inline) -> String {
    match node {
        Inline::Text(t) => t.clone(),

        Inline::Bold(children) => {
            format!(
                "#strong[{}]",
                children.iter()
                    .map(typst_inline)
                    .collect::<String>()
            )
        }

        Inline::Italic(children) => {
            format!(
                "#emph[{}]",
                children.iter()
                    .map(typst_inline)
                    .collect::<String>()
            )
        }

        Inline::Color {
            color,
            children
        } => {
            format!(
                "#text(fill: {})[{}]",
                color,
                children.iter()
                    .map(typst_inline)
                    .collect::<String>()
            )
        }
    }
}
```

Output:

```typst
Cells are the
#strong[basic unit of life].

The
#emph[cell membrane]
protects the cell.

#text(fill: red)[DNA]
stores genetic information.
```

---

# 8. Quiz HTML

```html
<div class="quiz">
  <h4>
    What is the basic unit of life?
  </h4>

  <input type="radio"> Cell
  <input type="radio"> Tissue
  <input type="radio"> Organ
</div>
```

---

# 9. TTS

Ignore formatting tags entirely:

```rust
fn extract_text(nodes: &[Inline]) -> String
```

Output:

```text
Cells are the basic unit of life.

The cell membrane protects the cell.

DNA stores genetic information.
```

Feed directly into a TTS engine.

---

# 10. Video Generation

Each content block becomes a scene:

```yaml
- paragraph
- image
- quiz
```

↓

```text
Scene 1:
Narration:
Cells are the basic unit of life.

Scene 2:
Show:
images/cell.png

Scene 3:
Quiz Screen
```

This gives you a single YAML source that can generate:

* HTML books
* Typst/PDF textbooks
* EPUB
* TTS audiobooks
* Educational videos
* Interactive quizzes

while keeping authoring simple with XML-like inline formatting only where needed.
