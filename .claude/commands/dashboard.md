Show ecosystem stats: working directory, lines of code, test counts, and git status.

## Instructions

1. **Gather stats in parallel** using separate Bash calls for each repo:

   - Current working directory and branch: `pwd` and `git branch --show-current`
   - Engine source LoC: `find ~/alities-engine/Sources -name '*.swift' | xargs wc -l | tail -1`
   - Engine test LoC: `find ~/alities-engine/Tests -name '*.swift' | xargs wc -l | tail -1`
   - Studio source LoC: `find ~/alities-studio/src \( -name '*.ts' -o -name '*.tsx' \) ! -path '*__tests__*' | xargs wc -l | tail -1`
   - Studio test LoC: `find ~/alities-studio/src -path '*__tests__*' \( -name '*.ts' -o -name '*.tsx' \) 2>/dev/null | xargs wc -l 2>/dev/null | tail -1`
   - Mobile source LoC: `find ~/alities-mobile -path '*/Sources/*.swift' | xargs wc -l | tail -1`
   - Mobile test LoC: `find ~/alities-mobile -path '*/Tests/*.swift' | xargs wc -l | tail -1`
   - Git status for each repo: `git -C ~/alities status --porcelain`, `git -C ~/alities-engine status --porcelain`, `git -C ~/alities-studio status --porcelain`, `git -C ~/alities-mobile status --porcelain`

2. **List available skills**: Read file names from `~/alities/.claude/commands/` and the first line of each.

3. **Get test counts** from `~/alities/CLAUDE.md` (the Test Suites table).

4. **Output format:**

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

Format numbers with commas.
