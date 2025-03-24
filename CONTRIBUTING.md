# Contributing to DC4EU WP7 Documentation

## Table of Content

- [Introduction](#introduction)
- [Style Guide](#style-guide)
- [Tooling](#tooling)
- [Linting and Style Checks](#linting-and-style-checks)
  - [Using markdownlint](#using-markdownlint)
    - [Option 1: Editor Plugin (VS Code - markdownlint)](#option-1-editor-plugin-vs-code---markdownlint)
    - [Option 2: Command Line with npm](#option-2-command-line-with-npm)
    - [Option 3: Run with Docker (markdownlint)](#option-3-run-with-docker-markdownlint)
  - [Using Vale](#using-vale)
    - [Option 1: Editor Plugin (VS Code – Vale)](#option-1-editor-plugin-vs-code--vale)
    - [Option 2: Command Line (Vale CLI)](#option-2-command-line-vale-cli)
    - [Option 3: Run with Docker (Vale)](#option-3-run-with-docker-vale)
  - [Config](#config)
- [Writing at 80 Characters in VS Code](#writing-at-80-characters-in-vs-code)
  - [Visual ruler at 80 characters](#visual-ruler-at-80-characters)
  - [Enforce hard line breaks with Rewrap](#enforce-hard-line-breaks-with-rewrap)
- [Contribution Process](#contribution-process)

## Introduction

All documentation should be written in Markdown and maintained in Git
repositories. To ensure consistency and high-quality documentation, please
follow the guidelines below.

## Style Guide

- **Follow the GitHub Style Guide**  
  Use the
  [GitHub Style Guide](https://docs.github.com/en/contributing/style-guide-and-content-model/style-guide)
  as the baseline for tone, formatting, and structure. Prioritize clarity and
  simplicity.

- **Keep line length under 80 characters**  
  Use hard line breaks at 80 characters to improve readability in diffs,
  editors, and terminals. This also makes reviews and collaboration easier
  across different tooling environments.

- **Use semantic Markdown**  
  Structure documents using appropriate headings (`#`, `##`, etc.), lists, code
  blocks, and tables. Use descriptive link text instead of raw URLs.

  **Examples of Semantic Markdown:**

  | **Intent**     | **Use this Markdown**  | **Not this**                    |
  |----------------|------------------------|---------------------------------|
  | Section title  | `# Section Title`      | `**Section Title**`             |
  | Subsection     | `## Subsection`        | `*Subsection*` or `> Subsection`|
  | Ordered steps  | `1. Step one`          | `- Step one`                    |
  | Unordered list | `- Item` or `* Item`   | `1. Item`                       |
  | Code snippet (inline) | `` `code()` ``  | `'code()'`                      |
  | Link with description | `[VC Spec](https://...)`    | `https://...`   |
  | Quote or citation     | `> This is a quote.` | Italics or just text       |

- **Code blocks (multiline)**
  Use fenced code blocks for multiple lines of code:

  <pre>
  ```bash
  some_command_here
  ```
  </pre>

  Avoid manually indenting or quoting code blocks.

- **Be concise and technical**  
  Write for a skilled technical audience. Avoid fluff. Include implementation
  details, protocol logic, and references to relevant specifications.

- **Version control friendly**  
  Use consistent formatting and break lines logically to minimize diff noise in
  pull requests.

---

## Tooling

To enforce style and consistency automatically, we recommend the following
tools:

- **markdownlint**  
  A Markdown linter that checks formatting rules like line length, heading
  style, and list indentation. Configured via `.markdownlint.json`.

- **Vale**  
  A prose linter that can enforce tone, voice, clarity, and other stylistic
  guidelines. Configured via `.vale.ini` and style definitions.

Configuration files for these tools are included in the repository.

## Linting and Style Checks

Please lint your Markdown files before submitting a pull request.

### Using markdownlint

You can run `markdownlint` in several ways depending on your environment and
preferences. Choose whichever works best for your setup:

#### Option 1: Editor Plugin (VS Code - markdownlint)

The easiest way to lint Markdown while writing.

1. Open VS Code.
2. Go to Extensions (`Ctrl+Shift+X` or `Cmd+Shift+X`).
3. Search for **markdownlint** by David Anson.
4. Install it.

This provides real-time linting as you write, with inline warnings and quick
fixes.

#### Option 2: Command Line with npm

Recommended if you want to lint manually or use markdownlint in automation/CI.

1. Install Node.js if you don’t already have it:  
   [Node.js](https://nodejs.org/)

2. Install markdownlint CLI globally:

   ```bash
   npm install -g markdownlint-cli
   ```

3. Run it on your Markdown files:

   ```bash
   markdownlint '**/*.md'
   ```

If the project includes a `.markdownlint.json`, it will be used automatically.

#### Option 3: Run with Docker (markdownlint)

Useful if you don't want to install anything locally.

```bash
docker run --rm -v "$(pwd)":/work ghcr.io/igorshubovych/markdownlint-cli2 \
markdownlint-cli2 "**/*.md"
```

- This mounts your current folder into the container and runs the linter.
- It will respect your local `.markdownlint.json` config if present.

---

### Using Vale

Vale is a style linter for writing. It checks your Markdown for grammar, tone,
clarity, and consistency. You can run it in multiple ways:

#### Option 1: Editor Plugin (VS Code – Vale)

Provides instant feedback while writing.

1. Open VS Code.
2. Go to Extensions (`Ctrl+Shift+X` or `Cmd+Shift+X`).
3. Search for **Vale**.
4. Install the extension by **Chris Chinchilla**.
5. Make sure you have a `.vale.ini` file in the workspace root.

#### Option 2: Command Line (Vale CLI)

Works across systems and is CI-friendly.

1. Download Vale from:  
   [Vale CLI Installation](https://vale.sh/docs/vale-cli/installation/)

   Or use Homebrew (macOS):

   ```bash
   brew install vale
   ```

2. Make sure a `.vale.ini` config file exists in the repo root.
3. Run it:

   ```bash
   vale CONTRIBUTING.md
   vale docs/
   ```

#### Option 3: Run with Docker (Vale)

No local install needed.

```bash
docker run --rm -v "$(pwd)":/docs jdkato/vale vale CONTRIBUTING.md
```

- You can also mount a config and styles directory with `-v` if you use custom
  styles.
- Make sure `.vale.ini` is in the root or explicitly passed via `--config`.

### Config

- The default config in this project uses Vale’s built-in `write-good` style.

---

## Writing at 80 Characters in VS Code

To help follow the 80-character line length guideline, we recommend the
following setup in Visual Studio Code:

### Visual ruler at 80 characters

To help you follow the 80-character line limit, add a vertical guide in VS Code:

1. Open `File → Preferences → Settings`
2. Search for `rulers`
3. Click **“Edit in settings.json”** (required because `editor.rulers` is an
   array)
4. Add the following setting:

   ```json
   "editor.rulers": [80]
   ```

This will show a vertical line at column 80 in all files, making it easier to
break lines consistently.

### Enforce hard line breaks with Rewrap

Install the **Rewrap** extension to automatically insert hard line breaks at 80
characters:

1. Go to the Extensions view (`Ctrl+Shift+X` or `Cmd+Shift+X`)
2. Search for **Rewrap**
3. Install it

Then:

- Select a paragraph in a Markdown file
- Press `Alt+Q` (`Cmd+Q` on Mac) to rewrap it to 80 characters per line

You can set the wrapping column explicitly in your user settings:

```json
"rewrap.wrappingColumn": 80
```

This ensures the file has real line breaks, which makes Git diffs smaller and
more readable in pull requests.

> Soft wrapping (i.e. just enabling `"editor.wordWrap": "on"`) is not enough
> — it doesn’t insert real line breaks and won’t help with diff noise.

---

## Contribution Process

- Submit documentation updates or new content via pull request.
- Use clear commit messages and group related changes together.
- PRs will be reviewed for content, structure, and adherence to the style guide.

We may formalize these rules further over time, but this is the current working
standard.

Thank you for contributing!
