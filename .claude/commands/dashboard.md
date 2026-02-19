Show ecosystem stats: working directory, lines of code, test counts, and git status.

## Instructions

1. **Run a single Bash command** that gathers all stats at once:

```bash
echo "CWD:$(pwd):$(git branch --show-current 2>/dev/null || echo 'no-git')" && echo "SRC_ENGINE:$(find ~/alities-engine/Sources -name '*.swift' | xargs wc -l | tail -1 | awk '{print $1}')" && echo "TST_ENGINE:$(find ~/alities-engine/Tests -name '*.swift' | xargs wc -l | tail -1 | awk '{print $1}')" && echo "SRC_STUDIO:$(find ~/alities-studio/src \( -name '*.ts' -o -name '*.tsx' \) ! -path '*__tests__*' | xargs wc -l | tail -1 | awk '{print $1}')" && echo "TST_STUDIO:$(find ~/alities-studio/src -path '*__tests__*' \( -name '*.ts' -o -name '*.tsx' \) 2>/dev/null | xargs wc -l 2>/dev/null | tail -1 | awk '{print $1+0}')" && echo "SRC_MOBILE:$(find ~/alities-mobile -path '*/Sources/*.swift' | xargs wc -l | tail -1 | awk '{print $1}')" && echo "TST_MOBILE:$(find ~/alities-mobile -path '*/Tests/*.swift' | xargs wc -l | tail -1 | awk '{print $1}')" && echo "GIT_HUB:$(cd ~/alities && git status --porcelain | wc -l | awk '{print ($1==0)?"clean":"dirty"}')" && echo "GIT_ENGINE:$(cd ~/alities-engine && git status --porcelain | wc -l | awk '{print ($1==0)?"clean":"dirty"}')" && echo "GIT_STUDIO:$(cd ~/alities-studio && git status --porcelain | wc -l | awk '{print ($1==0)?"clean":"dirty"}')" && echo "GIT_MOBILE:$(cd ~/alities-mobile && git status --porcelain | wc -l | awk '{print ($1==0)?"clean":"dirty"}')" && for f in ~/alities/.claude/commands/*.md; do echo "SKILL:$(basename "$f" .md):$(head -1 "$f")"; done
```

2. **Parse the output** and present as markdown. Format numbers with commas.

3. **Output format:**

**Working directory:** `/path/to/dir` (branch: `main`)

| Repo | Source LoC | Test LoC | Tests | Git Status |
|------|-----------|----------|-------|------------|
| alities-engine | X,XXX | X,XXX | 74 | clean/dirty |
| alities-studio | X,XXX | X,XXX | 1 | clean/dirty |
| alities-mobile | X,XXX | X,XXX | 3 | clean/dirty |
| **Total** | **XX,XXX** | **X,XXX** | **78** | |

| Skill | Description |
|-------|-------------|
| `/skill-name` | First line of skill file |

4. **Get test counts** from each repo's CLAUDE.md (use Grep, search for test count in parentheses — do this in parallel with step 1 if possible, or read from memory).
