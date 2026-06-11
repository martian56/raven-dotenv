# raven-dotenv

A `.env` parser for Raven. It reads the usual `KEY=value` lines into a config
object with typed getters, so you ask for an `Int` or a `Bool` and get one back,
with a fallback for when the key isn't set. No dependencies.

## Install

```toml
[dependencies]
"github.com/martian56/raven-dotenv" = "0.1"
```

## Usage

```raven
import "github.com/martian56/raven-dotenv" { load }

fun main() {
    match load(".env") {
        Ok(cfg) -> {
            let port = cfg.get_int_or("PORT", 8080)
            let url = cfg.get_or("DATABASE_URL", "sqlite://local.db")
            let debug = cfg.get_bool_or("DEBUG", false)
        },
        Err(e) -> print(e.to_string()),
    }
}
```

If you already have the text in hand (a string rather than a file), call `parse`
instead of `load`. Either way you get back a `Config`, or a `ParseError` that
names the line it choked on, like `.env:3: unterminated quoted value`.

A `Config` also has `require`, for the keys you can't run without:

```raven
match cfg.require("API_KEY") {
    Ok(key) -> connect(key),
    Err(msg) -> print(msg),     // "missing required key: API_KEY"
}
```

## What it understands

`KEY=value`, an optional `export` in front, blank lines, and `#` comments,
either on their own line or after a value (as long as there's a space before the
`#`, so a `#` inside a URL stays put). Double-quoted values run through the usual
`\n` / `\t` / `\r` escapes; single quotes are taken exactly as written.

```
# a comment
PORT=8080
export TOKEN=abc          # trailing comment
GREETING="hi\nthere"      # becomes two lines
LITERAL='no \t escapes'   # stays literal
URL=http://x/y#frag       # the # is part of the value
EMPTY=
```

## API

### Functions

| Function | Description |
|----------|-------------|
| `parse(text)` | Parse a `.env` string, returns `Result<Config, ParseError>` |
| `load(path)` | Read a file and parse it, same return type |

### `Config`

| Method | Description |
|--------|-------------|
| `Config.new()` | An empty config, handy as a base for defaults |
| `cfg.get(key)` | The raw value, `Option<String>` |
| `cfg.get_or(key, fallback)` | The value, or `fallback` if it's missing |
| `cfg.require(key)` | The value, or `Err` naming the missing key |
| `cfg.get_int(key)` / `get_int_or(key, fallback)` | The value as an `Int` |
| `cfg.get_bool(key)` / `get_bool_or(key, fallback)` | As a `Bool` (`true`/`1`/`yes`/`on`) |
| `cfg.has(key)` | Is `key` set? |
| `cfg.keys()` / `cfg.len()` | The keys, and how many there are |
| `cfg.set(key, value)` | Add or overwrite a value |

### `ParseError`

A 1-based `line` and a `message`, with a `to_string` that reads
`.env:<line>: <message>`.

## License

MIT
