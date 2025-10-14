# CLI Reference

Complete command-line reference for ck.

## Usage

```bash
ck [OPTIONS] [PATTERN] [PATH...]
```

## Search Modes

### Keyword Search (Default)

```bash
ck "pattern" [paths]
ck -i "case-insensitive" src/
ck -w "whole-word" src/
```

### Semantic Search

```bash
ck --sem "conceptual query" [paths]
ck --semantic "error handling" src/
```

### Hybrid Search

```bash
ck --hybrid "combined query" [paths]
```

### MCP Server

```bash
ck --serve
```

## Search Options

| Flag | Description |
|------|-------------|
| `--sem`, `--semantic` | Semantic search mode |
| `--hybrid` | Hybrid search (semantic + keyword) |
| `-i`, `--ignore-case` | Case-insensitive search |
| `-w`, `--word-regexp` | Match whole words only |
| `-v`, `--invert-match` | Invert match (show non-matching lines) |

## Output Options

| Flag | Description |
|------|-------------|
| `-n`, `--line-number` | Show line numbers |
| `-l`, `--files-with-matches` | List files with matches only |
| `-L`, `--files-without-match` | List files without matches |
| `-c`, `--count` | Count matches per file |
| `--no-filename` | Hide filenames in output |
| `--scores` | Show relevance scores (semantic/hybrid) |

## Context Control

| Flag | Description |
|------|-------------|
| `-A NUM`, `--after-context NUM` | Show NUM lines after match |
| `-B NUM`, `--before-context NUM` | Show NUM lines before match |
| `-C NUM`, `--context NUM` | Show NUM lines before and after match |

## Search Scope

| Flag | Description |
|------|-------------|
| `-R`, `-r`, `--recursive` | Recursive search |
| `--exclude PATTERN` | Exclude files matching pattern |
| `--no-ignore` | Don’t respect .gitignore |
| `--no-ckignore` | Don’t respect .ckignore |

## Semantic Options

| Flag | Description |
|------|-------------|
| `--topk NUM`, `--limit NUM` | Limit to top NUM results (default: 100) |
| `--threshold FLOAT` | Minimum relevance score (0.0-1.0, default: 0.6) |
| `--full-section` | Return complete code sections |

## Index Management

| Flag | Description |
|------|-------------|
| `--index [PATH]` | Build/update index |
| `--status [PATH]` | Show index status |
| `--clean [PATH]` | Remove index and rebuild |
| `--add FILE` | Add single file to index |
| `--inspect FILE` | Show file chunking details |

## Model Selection

| Flag | Description |
|------|-------------|
| `--model NAME` | Embedding model (bge-small, nomic-v1.5, jina-code) |
| `--switch-model NAME` | Switch to different model |
| `--force` | Force rebuild (with --switch-model) |

## Output Formats

| Flag | Description |
|------|-------------|
| `--json` | JSON array output |
| `--jsonl` | JSONL (one JSON object per line) |
| `--no-snippet` | Metadata only (no content snippets) |
| `--snippet-length NUM` | Snippet size in characters |

## MCP Options

| Flag | Description |
|------|-------------|
| `--serve` | Start MCP server |
| `--page-size NUM` | Results per page (MCP pagination) |
| `--cursor TOKEN` | Pagination cursor |

## Examples

### Basic Search

```bash
# Keyword search
ck "TODO" src/

# Case-insensitive
ck -i "error" src/

# With line numbers
ck -n "pattern" file.txt
```

### Semantic Search

```bash
# Find concepts
ck --sem "error handling" src/

# With scores
ck --sem --scores "auth" src/

# High confidence only
ck --sem --threshold 0.7 "pattern" src/

# Top 5 results
ck --sem --topk 5 "pattern" src/
```

### Output Formats

```bash
# List matching files
ck -l "pattern" src/

# JSON output
ck --json --sem "pattern" src/

# JSONL for streaming
ck --jsonl --sem "pattern" src/
```

### Index Operations

```bash
# Check status
ck --status .

# Build index
ck --index .

# Rebuild from scratch
ck --clean .
ck --index .

# Switch models
ck --switch-model nomic-v1.5 .
```

## Exit Codes

- `0`: Matches found
- `1`: No matches found
- `2`: Error occurred

## Environment Variables

Currently none. All configuration is via command-line flags or `.ckignore` file.

## Next Steps

- Learn [basic usage](/guide/basic-usage)
- Explore [semantic search](/features/semantic-search)
- Try [MCP integration](/features/mcp-integration)
- Check [configuration](/reference/configuration)
