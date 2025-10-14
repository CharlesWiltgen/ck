---
layout: home

hero:
  name: ck
  text: Semantic Code Search
  tagline: Supercharge your coding LLM with grep-compatible, semantic search that understands what you’re looking for, instead of just string patterns
  image:
    src: /logo.png
    alt: ck logo
  actions:
    - theme: brand
      text: Get Started
      link: /guide/getting-started
    - theme: alt
      text: View on GitHub
      link: https://github.com/BeaconBay/ck

features:
  - icon: 🤖
    title: AI Agent Integration
    details: Built-in MCP (Model Context Protocol) server for seamless integration with Claude Desktop, Cursor, and any MCP-compatible AI client
    link: /features/mcp-integration

  - icon: 🔍
    title: Semantic Search
    details: Find code by concept, not keywords. Search for "retry logic" and find backoff, circuit breakers, and related patterns even without exact matches
    link: /features/semantic-search

  - icon: ⚡
    title: Drop-in grep Replacement
    details: All your muscle memory works. Same flags, same behavior, same output format — plus semantic understanding when you need it
    link: /features/grep-compatibility

  - icon: 🎯
    title: Hybrid Search
    details: Combine keyword precision with semantic understanding using Reciprocal Rank Fusion for best-of-both-worlds search results
    link: /features/hybrid-search

  - icon: 🚀
    title: Blazing Fast
    details: ~1M LOC indexed in under 2 minutes. Sub-500ms queries. Automatic delta indexing only processes changed files
    link: /guide/basic-usage

  - icon: 📦
    title: Completely Offline
    details: Everything runs locally. No code or queries sent to external services. Embedding model downloaded once and cached locally
    link: /reference/models
---

## Quick Start

```bash
# Install from crates.io
cargo install ck-search

# Just search - ck builds and updates indexes automatically
ck --sem "error handling" src/
ck --sem "authentication logic" src/
ck --sem "database connection pooling" src/

# Traditional grep-compatible search still works
ck -n "TODO" *.rs
ck -R "TODO|FIXME" .

# Combine both: semantic relevance + keyword filtering
ck --hybrid "connection timeout" src/
```

## Why ck?

**ck (seek)** finds code by meaning, not just keywords. It’s the grep you wish you had:

- 🎯 **Understand intent** – Search for "error handling" and find try/catch blocks, error returns, and exception handling even when those exact words aren't present
- 🤖 **AI-first** – Built-in MCP server for direct integration with AI coding assistants
- ⚡ **Fast & efficient** – Automatic incremental indexing, sub-second queries
- 🔧 **Drop-in replacement** – Works exactly like grep/ripgrep with all the flags you know
- 🌐 **Multi-language** – Python, JavaScript/TypeScript, Rust, Go, Ruby, Haskell, C#, and more
- 🔒 **Privacy-first** – 100% offline, no telemetry, no external API calls

## Installation

### From crates.io (recommended)
```bash
cargo install ck-search
```

### From source
```bash
git clone https://github.com/BeaconBay/ck
cd ck
cargo install --path ck-cli
```

## Next Steps

<div class=”vp-doc”>

- [**Getting Started Guide**](/guide/getting-started) — Installation and first search
- [**Basic Usage**](/guide/basic-usage) — Common patterns and workflows
- [**MCP Integration**](/features/mcp-integration) — Connect with AI agents
- [**CLI Reference**](/reference/cli) — Complete command-line reference

</div>
