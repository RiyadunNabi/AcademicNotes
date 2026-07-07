# Lexical Analysis: Tokens, Patterns, Lexemes, Attributes & Symbol Table

*Compiler Design (CSE309/310) — Study Notes*

---

## 0. The Big Picture: Why Does Lexical Analysis Exist?

A compiler's job starts by reading your source code as a raw stream of characters — just letters, digits, and symbols with no meaning attached yet. The **first phase** of a compiler, called **Lexical Analysis** (or **scanning**), takes this raw character stream and groups it into meaningful chunks, then labels each chunk with a category.

This is exactly what your brain does when reading English. Take the sentence:

> **"Boy eats 2 apples."**

Your brain automatically chops this up and categorizes each piece:

- "Boy" and "apples" → **Nouns**
- "eats" → **Verb**
- "2" → **Number**

A compiler does the exact same thing with code. Given a line like `int x = 5;`, the lexical analyzer reads it character by character and groups it into chunks: `int`, `x`, `=`, `5`, `;` — then labels each one so the next phase (the **parser**) can understand the structure.

To describe this process precisely, compiler theory uses three core terms: **Lexeme**, **Pattern**, and **Token**. Then, once tokens exist, we need two more ideas to make them actually *useful*: **Attributes** and the **Symbol Table**.

---

## 1. Lexeme — The Actual Word

**Definition:** A lexeme is the raw, actual sequence of characters in your source code. It's the exact text you typed — nothing abstracted, nothing categorized yet.

**Analogy:** In our sentence, the lexeme is the literal word "apples" or the symbol "2" — the actual text sitting on the page.

**Code Example:** In `int x = 5;`, the lexemes are the raw pieces of text:
```
int    x    =    5    ;
```

---

## 2. Pattern — The Rule

**Definition:** A pattern is the rule or description that tells the lexical analyzer *how* to recognize a specific category of lexeme. It's usually expressed as a regular expression.

**Analogy:** The rule in English that says "a number is made of digits 0–9" is a pattern. It doesn't refer to any specific number — it's the general shape that any valid number must follow.

**Code Example:**
- The pattern for an **identifier** (variable name) is something like: *"must start with a letter, followed by any number of letters or digits."*
- The pattern for the keyword `int` is simply the exact sequence of characters i-n-t.

A pattern is what the lexical analyzer checks *against*. A lexeme either matches a pattern or it doesn't.

---

## 3. Token — The Category / Label

**Definition:** A token is the category name (label) assigned to a lexeme once it has matched a pattern. This label is what actually gets passed forward to the parser — the parser doesn't care about your exact variable names, only about the grammatical structure (which token types appear in which order).

**Analogy:** The label "**Noun**" or "**Number**" — the category itself, not the specific word.

**Code Example:** When the compiler sees the lexeme `x`, it checks it against known patterns, finds that it matches the identifier pattern, and tags it with the token **id**.

### Relationship Between the Three

```
Lexeme  →  (checked against)  →  Pattern  →  (if matched, assign)  →  Token
"x"                "letter, then letters/digits"              id
```

- **Lexeme** = what you typed
- **Pattern** = the rule used to recognize it
- **Token** = the label given after a successful match

---

## 4. Worked Example: Breaking Down a Real Statement

Let's apply this to a C statement:

```c
printf("Total = %d\n", score);
```

Putting on our "compiler glasses," here's exactly how it gets read and labeled:

| Lexeme (what you typed) | Pattern (the rule it matched) | Token (the label assigned) |
|---|---|---|
| `printf` | Starts with a letter, contains letters | **id** (Identifier) |
| `(` | The open parenthesis symbol | **Left_Paren** (Punctuation) |
| `"Total = %d\n"` | Anything wrapped inside double quotes | **literal** (String Literal) |
| `,` | The comma symbol | **Comma** (Punctuation) |
| `score` | Starts with a letter, contains letters | **id** (Identifier) |
| `)` | The close parenthesis symbol | **Right_Paren** (Punctuation) |
| `;` | The semicolon symbol | **Semicolon** (Punctuation) |

---

## 5. The 5 Main Token Families

Almost everything you can type in a programming language falls into one of five token buckets:

1. **Keywords** — Reserved words like `if`, `while`, `int`.
2. **Operators** — Math or logic symbols like `+`, `-`, `==`.
3. **Identifiers** — Names you invent for variables or functions (like `score` or `printf`).
4. **Constants / Literals** — Hardcoded values like `42` or `"Hello"`.
5. **Punctuation** — Symbols that separate things like `(`, `)`, `,`, `;`.

---

## 6. The Problem: Tokens Alone Lose Information

Here's a subtle but important issue. If the compiler reduces `x = 5;` down to just:

```
<id> <assign-op> <number>
```

...it now knows the *structure* of the statement, but it has completely lost *which* variable it was and *what* the number was. If the compiler later sees `y = 9;`, that also reduces to `<id> <assign-op> <number>` — identical structure, different meaning.

So how does the compiler tell `x` apart from `y`, or `5` apart from `9`, when it later needs to actually generate code or do math? This is where **attributes** come in.

---

## 7. Attribute — The Extra Detail

**Definition:** An attribute is the extra piece of information attached to a token so the compiler doesn't lose the specific value or identity behind that token.

**Format:** Written as a pair — **`<Token_Name, Attribute_Value>`**

**Examples:**
- The number `5` becomes the token `<number>`, but with its attribute it is fully represented as **`<number, 5>`**.
- The `=` operator has only one possible lexeme that matches the "assignment operator" pattern, so there's nothing extra to distinguish — `<assign-op>` alone is enough. No attribute needed.

**Rule of thumb:** If a token type can only ever correspond to one exact lexeme (like `=`, `*`, `;`), it typically doesn't need an attribute. If a token type can correspond to *many different* lexemes (like `id` or `number`), it needs an attribute to record which specific one this occurrence was.

---

## 8. The Symbol Table — The Compiler's Database

**Definition:** The Symbol Table is the compiler's internal database (think: an Excel spreadsheet) that tracks every variable, function, or class name you invent in your code.

**Important distinction:** The Symbol Table does **not** store tokens. It stores the **lexemes** (actual names) of your identifiers, along with rich metadata about each one.

If you write `int score = 100;`, the Symbol Table creates a new row:

| Field | Value |
|---|---|
| Lexeme (Name) | `score` |
| Type | `int` |
| Scope | Where this variable is valid/visible |
| Memory Location | Where in RAM/stack it will live |

**Why not put all this into the token itself?** Because it would make every token bulky and slow to pass around. Instead, the lexical analyzer hands the token a lightweight **pointer** — like a ticket number — pointing to the corresponding row in the Symbol Table.

---

## 9. Worked Example: Connecting Attributes and the Symbol Table

Take this Fortran-style statement:

```
E = M * C ** 2
```

Here is exactly what the lexical analyzer produces, using the `<Token, Attribute>` format:

| Lexeme | Token Name | Attribute Value | Why |
|---|---|---|---|
| `E` | **id** | Pointer to `E`'s row in Symbol Table | Needed later to look up `E`'s type, scope, etc. |
| `=` | **assign-op** | *(none)* | Only one possible lexeme — no extra detail needed. |
| `M` | **id** | Pointer to `M`'s row in Symbol Table | Needed to look up `M`'s details. |
| `*` | **mult-op** | *(none)* | Just a multiply sign. |
| `C` | **id** | Pointer to `C`'s row in Symbol Table | Needed to look up `C`'s details. |
| `**` | **exp-op** | *(none)* | Just an exponent (power) sign. |
| `2` | **number** | The integer value `2` | Compiler needs the exact number to use later. |

**In short:**
- **Lexeme:** the raw text (e.g. `score`)
- **Token:** the broad category (e.g. `id`)
- **Attribute:** the specific detail — either a pointer into the Symbol Table (for identifiers) or the literal value itself (for constants)
- **Symbol Table:** the database holding the heavy metadata about your variables

---

## 10. Where Is the Attribute Actually *Stored*?

We've established:
- Tokens go to the parser as a **token stream**.
- The Symbol Table holds the heavy metadata, sitting off to the side.

So where does the attribute physically live in between these two? **The attribute is stored inside the token itself.**

### The Token Is a "Box" (an Object / Struct)

When the lexical analyzer sends a "token" to the parser, it isn't sending a bare string like `"id"`. It's sending a small structured object — a `struct` in C, or an `Object` in Java/Python — with two compartments: one for the category, one for the attribute.

```c
struct Token {
    int   token_name;   // Compartment 1: the category (e.g. ID, NUMBER, PLUS)
    void* attribute;    // Compartment 2: the attribute (pointer or actual value)
};
```

### Tracing the Flow in Memory

Let's trace what happens when the compiler reads `score = 100;`:

**Step 1 — Handling `score`:**
- Lexer reads: `s-c-o-r-e`
- **Symbol Table action:** the lexer checks — "Have I seen `score` before?" If not, it creates a new row for `score` in the Symbol Table, say at memory address `0x10A`.
- **Token creation:** a new `Token` struct is built:
  - `token_name = id`
  - `attribute = 0x10A` (the pointer)
- The attribute is physically stored right inside this `Token` struct, in RAM, which then gets pushed onto the token stream.

**Step 2 — Handling `=`:**
- Lexer reads: `=`
- **Token creation:**
  - `token_name = assign_op`
  - `attribute = NULL` (empty — not needed)

**Step 3 — Handling `100`:**
- Lexer reads: `1-0-0`
- **Token creation:**
  - `token_name = number`
  - `attribute = 100` (the actual integer value, stored directly)

### Summary of Where Everything Lives

- The **token stream** is literally a conveyor belt of these structs/objects, one after another.
- The **attribute** travels *inside* that struct, riding along the conveyor belt — it's the lightweight piece of data carried directly by the token.
- The **heavy metadata** (type, scope, memory allocation) lives separately in the Symbol Table. The attribute, when it's a pointer, is just the map that leads back to that heavy data.

---

## 11. Full Mental Model (Quick Recap)

```
Source code (raw characters)
        │
        ▼
  Lexical Analyzer
   - matches lexemes against patterns
   - assigns a token name to each lexeme
   - creates/looks up entries in the Symbol Table for identifiers
   - packs each token as <token_name, attribute> inside a Token struct
        │
        ▼
   Token Stream ──────────────► sent to Parser
        │
        │ (attribute, if it's a pointer)
        ▼
   Symbol Table (heavy metadata: type, scope, memory location, etc.)
```

**One-line definitions to memorize:**
- **Lexeme** — the actual text typed.
- **Pattern** — the rule used to recognize a category of lexeme.
- **Token** — the category label assigned after a pattern match.
- **Attribute** — the extra value/pointer stored inside the token so identity/value isn't lost.
- **Symbol Table** — the external database holding full metadata about identifiers, referenced via attribute pointers.

----------
Exercise: Identifying Tokens
For the following code fragment, write the sequence of tokens
produced by the lexical analyzer.

```c
long long int x = 1;
printf("value = %d ", x);
```

Here is the step-by-step breakdown of how a lexical analyzer would read that code fragment and convert it into a sequence of tokens.

Remember, the lexical analyzer reads left-to-right, character by character, grouping them into lexemes and attaching the correct Token and Attribute.

### The Breakdown

| Lexeme | Token Name | Attribute Value |
| --- | --- | --- |
| `long` | **keyword** | *(None)* |
| `long` | **keyword** | *(None)* |
| `int` | **keyword** | *(None)* |
| `x` | **id** | Pointer to `x` in Symbol Table |
| `=` | **assign-op** | *(None)* |
| `1` | **number** | Integer value 1 |
| `;` | **punctuation** | *(None)* |
| `printf` | **id** | Pointer to `printf` in Symbol Table |
| `(` | **punctuation** | *(None)* |
| `"value = %d "` | **literal** | String value `"value = %d "` |
| `,` | **punctuation** | *(None)* |
| `x` | **id** | Pointer to `x` in Symbol Table |
| `)` | **punctuation** | *(None)* |
| `;` | **punctuation** | *(None)* |

---

### The Final Token Stream

When the lexical analyzer finishes, it sends this continuous stream of `<Token, Attribute>` pairs to the Parser.

Here is exactly what that stream looks like:

`<keyword>`
`<keyword>`
`<keyword>`
`<id, pointer to x>`
`<assign-op>`
`<number, 1>`
`<punctuation>`
`<id, pointer to printf>`
`<punctuation>`
`<literal, "value = %d ">`
`<punctuation>`
`<id, pointer to x>`
`<punctuation>`
`<punctuation>`

*(Note: Depending on the specific compiler design, punctuation like `;`, `(`, and `)` might be given specific token names like `<semicolon>`, `<left-paren>`, and `<right-paren>`, but they all fall under the punctuation/separator category!)*