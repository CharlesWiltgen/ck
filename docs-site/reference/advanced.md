# Advanced Configuration

Advanced configuration options, environment variables, and power-user techniques for ck.

## Environment Variables

### HuggingFace Cache Reuse

Reuse your existing HuggingFace cache instead of downloading models again:

```bash
# Set HuggingFace home directory
export HF_HOME=~/.cache/huggingface

# Or set hub cache directly
export HF_HUB_CACHE=~/.cache/huggingface/hub

# Now ck will use existing cached models
ck --index .
```

**Use case**: You already have embedding models downloaded for other projects.

**Benefits**:
- Avoid duplicate downloads
- Save disk space
- Faster initial setup

### Model Cache Locations

ck/fastembed uses these cache directories:

| Platform | Default Location |
|----------|------------------|
| Linux | `~/.cache/ck/models/` |
| macOS | `~/.cache/ck/models/` |
| Windows | `%LOCALAPPDATA%\ck\cache\models\` |
| Fallback | `.ck_models/models/` (current directory) |

**Override with environment variables**:
```bash
export HF_HOME=/custom/cache/dir
```

### Rust Logging

Enable detailed logging for debugging:

```bash
# All logs
RUST_LOG=debug ck --sem "pattern" src/

# Specific module
RUST_LOG=ck_engine=trace ck --sem "pattern" src/

# Multiple modules
RUST_LOG=ck_engine=debug,ck_index=info ck --index .
```

**Log levels**: `error`, `warn`, `info`, `debug`, `trace`

## Performance Tuning

### Large Codebase Strategies

For codebases >1M LOC:

#### 1. Choose Faster Model

```bash
# Use BGE-Small (fastest)
ck --index --model bge-small .
```

BGE-Small trades some accuracy for speed:
- 400-token chunks (vs 1024 for others)
- Smaller embedding dimensions (384 vs 768)
- Faster inference

#### 2. Aggressive Exclusions

Create comprehensive `.ckignore`:

```gitignore
# Build artifacts
target/
dist/
build/
out/

# Dependencies
node_modules/
vendor/
venv/
.venv/

# Generated code
*.generated.*
*_pb2.py
*.pb.go

# Documentation
docs/
*.md
README*

# Tests (if you don't search them)
**/test/
**/tests/
**/*_test.*
**/*.test.*

# Data files
*.json
*.yaml
*.csv
*.xml

# Media
*.png
*.jpg
*.gif
*.svg
*.mp4
```

#### 3. Index in Batches

For extremely large monorepos:

```bash
# Index core first
echo "/*" > .ckignore
echo "!/src/" >> .ckignore
ck --index .

# Add more directories incrementally
echo "!/lib/" >> .ckignore
ck --index .

# And so on...
```

#### 4. Separate Indexes for Components

For microservices/monorepos, index each service separately:

```bash
cd service-a && ck --index .
cd ../service-b && ck --index .
# etc.
```

### Memory Optimization

#### Reduce Chunk Size

For memory-constrained environments:

```bash
# Use smaller model with smaller chunks
ck --index --model bge-small .
```

#### Limit Search Results

```bash
# Limit to top 10 results
ck --sem --topk 10 "pattern" src/

# Use aggressive threshold
ck --sem --threshold 0.8 "pattern" src/
```

#### JSONL Without Snippets

For AI agents with tight memory budgets:

```bash
# Metadata only, no content
ck --jsonl --no-snippet --sem "pattern" src/
```

### Disk Space Optimization

#### Clean Unused Indexes

```bash
# Find all .ck directories
find . -type d -name ".ck" -exec du -sh {} \;

# Remove indexes from old branches
find . -type d -name ".ck" -path "*/old-feature/*" -exec rm -rf {} \;
```

#### Use Symlinks for Shared Models

If working with multiple repositories:

```bash
# Central model cache
mkdir -p ~/shared-ck-models

# Symlink from each repo
cd ~/project1
ln -s ~/shared-ck-models ~/.cache/ck/models

cd ~/project2
ln -s ~/shared-ck-models ~/.cache/ck/models
```

## Advanced Search Patterns

### Custom Threshold Strategies

Different thresholds for different use cases:

```bash
# Exploratory (cast wide net)
ck --sem --threshold 0.3 --topk 50 "vague concept" src/

# Precise (only high confidence)
ck --sem --threshold 0.8 --topk 5 "specific pattern" src/

# Balanced (default)
ck --sem --threshold 0.6 --topk 20 "pattern" src/
```

### Combining Search Modes

```bash
# Semantic search then keyword filter
ck --sem "authentication" src/ | grep -i "jwt"

# Hybrid with custom output
ck --hybrid --scores "pattern" src/ | awk '$1 > 0.7'

# Multiple searches combined
{
  ck --sem "auth" src/
  ck --sem "login" src/
} | sort -u
```

### Pipeline Integration

#### Git Hooks

**Pre-commit hook** (`.git/hooks/pre-commit`):
```bash
#!/bin/bash

# Find TODOs in staged files
TODOS=$(git diff --cached --name-only | xargs ck --sem "TODO" 2>/dev/null)

if [ -n "$TODOS" ]; then
    echo "Warning: Found TODOs in staged files:"
    echo "$TODOS"
    read -p "Commit anyway? (y/n) " -n 1 -r
    echo
    if [[ ! $REPLY =~ ^[Yy]$ ]]; then
        exit 1
    fi
fi
```

**Pre-push hook** (`.git/hooks/pre-push`):
```bash
#!/bin/bash

# Check for potential security issues
SECURITY=$(ck --hybrid "password|api_key|secret" $(git diff --name-only origin/main...HEAD))

if [ -n "$SECURITY" ]; then
    echo "Error: Potential security issues found:"
    echo "$SECURITY"
    exit 1
fi
```

#### CI/CD Integration

**GitHub Actions**:
```yaml
- name: Install ck
  run: cargo install ck-search

- name: Check for security issues
  run: |
    if ck --hybrid "password|secret|api_key" src/; then
      echo "::error::Found potential security issues"
      exit 1
    fi

- name: Verify test coverage
  run: |
    # Find source files without corresponding tests
    if ck -L --sem "test" src/**/*.rs; then
      echo "::warning::Files without tests found"
    fi
```

**GitLab CI**:
```yaml
semantic_audit:
  script:
    - cargo install ck-search
    - ck --sem "TODO|FIXME" src/ || true
    - ck --hybrid "deprecated" src/ || true
  artifacts:
    reports:
      - audit_report.txt
```

### Shell Integration

#### Aliases

Add to `.bashrc` or `.zshrc`:

```bash
# Semantic search aliases
alias cks='ck --sem'
alias ckh='ck --hybrid'
alias ckf='ck --sem --full-section'

# Common patterns
alias ck-todo='ck --hybrid "TODO|FIXME|XXX"'
alias ck-bug='ck --sem "potential bug"'
alias ck-auth='ck --sem "authentication"'

# Search with scores
alias ckscore='ck --sem --scores --threshold 0.7'

# High-confidence search
alias ckstrict='ck --sem --threshold 0.8 --topk 5'
```

#### Functions

```bash
# Search and open in editor
ckopen() {
    local query="$1"
    local file=$(ck --sem "$query" . | fzf | cut -d: -f1)
    [ -n "$file" ] && $EDITOR "$file"
}

# Compare search modes
ckcompare() {
    local query="$1"
    echo "=== Semantic ==="
    ck --sem "$query" src/
    echo "\n=== Hybrid ==="
    ck --hybrid "$query" src/
    echo "\n=== Keyword ==="
    ck "$query" src/
}

# Index all git repos in directory
ck-index-all() {
    find . -name ".git" -type d | while read gitdir; do
        repo=$(dirname "$gitdir")
        echo "Indexing $repo"
        (cd "$repo" && ck --index .)
    done
}
```

## Advanced .ckignore Patterns

### Monorepo Patterns

```gitignore
# Exclude everything by default
/*

# Include only specific services
!/services/api/
!/services/web/
!/shared/

# But exclude their tests and mocks
services/**/test/
services/**/mocks/
```

### Framework-Specific Patterns

**Next.js**:
```gitignore
.next/
out/
*.config.js
public/
**/_app.tsx
**/_document.tsx
```

**Rails**:
```gitignore
db/schema.rb
db/migrate/
spec/fixtures/
tmp/
log/
```

**Django**:
```gitignore
**/migrations/
manage.py
**/settings.py
```

### Development vs Production

**Development** (`.ckignore.dev`):
```gitignore
# Aggressive exclusions for fast iteration
*.test.*
*.spec.*
**/__tests__/
docs/
*.md
```

**Production** (`.ckignore.prod`):
```gitignore
# Minimal exclusions for comprehensive search
node_modules/
dist/
*.log
```

Use with:
```bash
# Development
cp .ckignore.dev .ckignore && ck --index .

# Production
cp .ckignore.prod .ckignore && ck --index .
```

## Edge Cases & Workarounds

### Very Large Files

For files >10K LOC that slow indexing:

**Option 1**: Exclude from index
```gitignore
# .ckignore
schema.sql
generated/
```

**Option 2**: Split if possible
```bash
# Split large file into chunks
split -l 5000 large_file.py chunk_

# Index separately
ck --add chunk_*
```

**Option 3**: Use keyword search
```bash
# Skip semantic indexing for specific patterns
ck "specific_function" large_file.sql
```

### Mixed Language Repositories

For repos with many languages:

```bash
# Index supported languages first
echo "*.java" >> .ckignore
echo "*.php" >> .ckignore
ck --index .

# Use keyword search for unsupported
ck "pattern" **/*.java
```

### Symlinked Directories

ck follows symlinks by default. To exclude:

```bash
# Find symlinks
find . -type l

# Add to .ckignore
echo "symlinked-dir/" >> .ckignore
```

### Binary Files Incorrectly Detected as Text

Rare edge case where binary is indexed:

```gitignore
# .ckignore - explicit binary exclusions
*.bin
*.dat
*.exe
*.dll
*.so
*.dylib
specific-binary-file
```

## Debugging & Troubleshooting

### Verbose Indexing

```bash
# See what's being indexed
RUST_LOG=ck_index=debug ck --index . 2>&1 | grep "Processing"

# See chunking details
ck --inspect src/file.rs
```

### Check Index Health

```bash
# Status
ck --status .

# Verify file count
find . -type f -name "*.rs" | wc -l
# Compare with index status

# Check index size
du -sh .ck/
```

### Force Rebuild

When index seems corrupted:

```bash
# Nuclear option
rm -rf .ck/
ck --index .

# Or use clean command
ck --clean .
ck --index .
```

### Compare Models

```bash
# Script to compare models
for model in bge-small nomic-v1.5 jina-code; do
    echo "=== $model ==="
    ck --switch-model $model --force .
    time ck --sem "authentication" src/ | wc -l
done
```

## Security Considerations

### Sensitive Data

Never commit indexes of sensitive codebases:

```gitignore
# .gitignore
.ck/
```

### Access Control

For shared development machines:

```bash
# Restrict index directory permissions
chmod 700 .ck/

# Or use per-user cache
export HF_HOME=~/private/.ck-cache
```

### API Key Scanning

Regular semantic scans for secrets:

```bash
# Scheduled check
crontab -e
0 2 * * * cd /path/to/repo && ck --hybrid "api_key|password|secret" . | mail -s "Security Scan" dev@company.com
```

## Next Steps

- [Configuration](/reference/configuration) — Standard configuration options
- [CLI Reference](/reference/cli) — Complete command reference
- [Models](/reference/models) — Embedding model details
- [FAQ](/guide/faq) — Common questions and answers
