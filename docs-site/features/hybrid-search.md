# Hybrid Search

Combine semantic understanding with keyword precision using Reciprocal Rank Fusion.

## What is Hybrid Search?

Hybrid search merges semantic and keyword search results to give you the best of both worlds:
- **Semantic** – Finds conceptually related code
- **Keyword** – Ensures query terms are present
- **Fusion** – Ranks results by combined relevance

## Basic Usage

```bash
# Simple hybrid search
ck --hybrid "connection timeout" src/

# With relevance scores
ck --hybrid --scores "cache invalidation" src/

# Filter by confidence
ck --hybrid --threshold 0.5 "auth" src/
```

## How It Works

1. **Dual search**: Runs semantic AND keyword search in parallel
2. **Ranking**: Each produces ranked results
3. **Fusion**: RRF (Reciprocal Rank Fusion) merges rankings
4. **Output**: Combined results sorted by fused relevance

### Reciprocal Rank Fusion Formula

```
RRF_score(doc) = Σ (1 / (k + rank_i))

where:
- k = 60 (constant)
- rank_i = position in each ranking
```

Results appearing high in both rankings get highest scores.

## When to Use Hybrid Search

✅ **Use hybrid when:**
- You need both concept AND keywords present
- Semantic search returns too many false positives
- Keyword search misses related code
- You want balanced precision and recall

### Examples

```bash
# Find timeout code with "connection" keyword
ck --hybrid "connection timeout" src/

# Authentication with specific terms
ck --hybrid "jwt authentication" src/

# Performance with keywords
ck --hybrid "cache optimization" src/

# Security patterns
ck --hybrid "sql injection prevention" src/
```

## Comparison with Other Search Modes

| Feature | Keyword | Semantic | Hybrid |
|---------|---------|----------|--------|
| Exact matches | ✅ | ❌ | ✅ |
| Conceptual matches | ❌ | ✅ | ✅ |
| Requires keywords | ✅ | ❌ | ✅ |
| False positives | Low | Medium | Low |
| Speed | Fastest | Fast | Fast |

```bash
# Keyword: Finds exact "error" text
ck "error" src/

# Semantic: Finds error handling concepts
ck --sem "error handling" src/

# Hybrid: Finds error handling concepts with "error" keyword
ck --hybrid "error handling" src/
```

## Advanced Options

### Threshold Filtering

```bash
# High confidence only
ck --hybrid --threshold 0.7 "pattern" src/

# Exploratory (lower confidence)
ck --hybrid --threshold 0.3 "pattern" src/
```

### Result Limiting

```bash
# Top 10 results
ck --hybrid --topk 10 "pattern" src/

# With threshold
ck --hybrid --topk 20 --threshold 0.5 "pattern" src/
```

### Complete Sections

```bash
# Get full functions
ck --hybrid --full-section "retry logic" src/
```

## Best Practices

### Query Formulation

✅ **Good hybrid queries:**
```bash
ck --hybrid "database connection pool"    # Concept + keywords
ck --hybrid "async timeout handler"       # Specific + descriptive
ck --hybrid "jwt token validation"        # Technology + function
```

❌ **Poor hybrid queries:**
```bash
ck --hybrid "code"                        # Too vague
ck --hybrid "function that does stuff"    # No specific keywords
```

### Performance Tips

```bash
# Limit results for speed
ck --hybrid --topk 10 "pattern" src/

# Use threshold to reduce computation
ck --hybrid --threshold 0.6 "pattern" src/

# Search specific paths
ck --hybrid "pattern" src/core/
```

## Use Cases

### Refactoring

```bash
# Find specific implementation patterns
ck --hybrid "singleton pattern" src/

# Find duplication
ck --hybrid "user validation logic" src/
```

### Security Audits

```bash
# SQL injection risks
ck --hybrid "sql injection" src/

# XSS vulnerabilities
ck --hybrid "xss prevention" src/

# Hardcoded secrets
ck --hybrid "api key|password" src/
```

### API Integration

```bash
# Find API calls
ck --hybrid "rest api client" src/

# Find specific endpoints
ck --hybrid "payment gateway" src/
```

## Troubleshooting

### Too Many Results

```bash
# Increase threshold
ck --hybrid --threshold 0.7 "pattern" src/

# Limit count
ck --hybrid --topk 5 "pattern" src/

# Use more specific query
ck --hybrid "specific detailed pattern" src/
```

### Too Few Results

```bash
# Lower threshold
ck --hybrid --threshold 0.3 "pattern" src/

# Try semantic-only
ck --sem "pattern" src/

# Try keyword-only
ck "pattern" src/
```

### Unexpected Results

```bash
# Check with scores
ck --hybrid --scores "pattern" src/

# Try semantic alone
ck --sem --scores "pattern" src/

# Try keyword alone
ck "pattern" src/
```

## Next Steps

- Learn about [semantic search](/features/semantic-search)
- Explore [grep compatibility](/features/grep-compatibility)
- Try [MCP integration](/features/mcp-integration)
- Check [CLI reference](/reference/cli)
