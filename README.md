# Workshop book

This repository contains the participant-facing mdBook source in `src/`.

## Serve and open locally

Install [mdBook](https://rust-lang.github.io/mdBook/guide/installation.html), then run this from the repository root:

```sh
mdbook serve --open
```

The command builds the book, serves it at `http://localhost:3000`, and opens it in your default browser. Keep the command running while you read or edit; press Ctrl-C to stop it. If the browser cannot open automatically, visit the local URL yourself.

If port 3000 is in use, choose a free port:

```sh
mdbook serve --open --port 13174
```

For agents or headless environments, use `mdbook serve` without `--open` and inspect `src/SUMMARY.md` for chapter order. The generated `book/` directory is not source; edit the Markdown under `src/`.
