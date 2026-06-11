# Oxide

Elixir NIF bindings for [Tailwind CSS Oxide](https://tailwindcss.com) — the Rust-powered content scanner that powers Tailwind CSS v4.

Scans source files in parallel to extract Tailwind CSS candidate class names at native speed, with built-in support for HEEx, EEx, Elixir, Vue, Svelte, and 20+ other template languages.

## Features

- **Parallel filesystem scanning** — walks directories using Rayon, respects `.gitignore`
- **Candidate extraction** — state-machine parser pulls valid Tailwind candidates from any content
- **Incremental scanning** — only re-extracts changed files, returns only new candidates
- **Language-aware preprocessing** — built-in support for `.heex`, `.eex`, `.ex`, `.vue`, `.svelte`, `.erb`, and more
- **Stateless extraction** — extract candidates from a string without a scanner

## Installation

```elixir
def deps do
  [
    {:oxide, "~> 0.1.0"}
  ]
end
```

Requires a Rust toolchain (`rustup` recommended). The NIF compiles automatically on `mix compile`.

## Usage

### Scanner (stateful, incremental)

```elixir
# Create a scanner with source patterns
scanner = Oxide.new(sources: [
  %{base: "lib/", pattern: "**/*.{ex,heex}"},
  %{base: "assets/", pattern: "**/*.{vue,ts,tsx}"}
])

# Full scan — walks filesystem, returns all candidates
candidates = Oxide.scan(scanner)
# ["flex", "items-center", "bg-blue-500", "hover:text-white", ...]

# On file change — only returns NEW candidates not seen before
new = Oxide.scan_files(scanner, [
  %{file: "lib/app_web/live/page.ex", extension: "ex"}
])
# ["mt-8", "space-y-4"]

# Get discovered files (useful for watcher setup)
files = Oxide.files(scanner)

# Get generated glob patterns
globs = Oxide.globs(scanner)
```

### Extract (stateless)

```elixir
candidates = Oxide.extract(~s(class="flex bg-red-500 hover:text-white"), "html")
# [
#   %Oxide.Candidate{value: "class", position: 0},
#   %Oxide.Candidate{value: "flex", position: 7},
#   %Oxide.Candidate{value: "bg-red-500", position: 12},
#   %Oxide.Candidate{value: "hover:text-white", position: 23}
# ]
```

## How It Works

This library wraps the `tailwindcss-oxide` Rust crate — the same scanner used by Tailwind CSS v4 itself. The scanner:

1. Walks the filesystem in parallel using Rayon, respecting `.gitignore` rules
2. Preprocesses each file based on its extension (strips non-class content from HEEx, Vue, etc.)
3. Runs a state-machine-based extractor that identifies valid Tailwind candidates
4. Tracks seen candidates for fast incremental scanning on subsequent calls

All NIF calls run on the dirty CPU scheduler so they don't block the BEAM.

## Part of Elixir Volt

oxide scans content for Tailwind CSS candidates at native speed, with HEEx/EEx awareness built in.

It is part of a frontend stack that runs inside the BEAM — builds, JS
runtimes, icons, and Vue-to-LiveView compilation as supervised parts of the
application instead of external toolchain processes. See the
[Elixir Volt](https://github.com/elixir-volt) organization for the rest, and
[Building Blocks for the Future Web](https://github.com/elixir-vibe/building-blocks)
for the thesis, architecture, and roadmap that tie them together.

## License

MIT — Copyright 2026 Danila Poyarkov
