# GitHub Add Control

Purpose: safe GitHub read/patch/apply through a Railway service.

Tools:
- githubAddHealth
- githubReadFile
- githubPatchPreview
- githubPatchApply

Architecture:
```mermaid
flowchart LR
  GPT[Custom GPT] -->|Action auth| GHADD[github-add API on Railway]
  GHADD -->|GITHUB_TOKEN server-side| GITHUB[GitHub API]
  GITHUB --> REPO[Repositories]
```

Runtime evidence:
```yaml
status: ok
service: github-add
version: 0.2.4
github_token_present: true
github_token_prefix: github_pat
github_content_read_ok: true
```

Rules:
- read file first and capture SHA
- preview patch before apply
- apply only with expected SHA and commit message
- no secrets in commits
- no destructive rewrites
