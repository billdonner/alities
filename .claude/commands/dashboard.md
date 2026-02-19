Show ecosystem stats: working directory, lines of code, test counts, and git status.

**Do not prompt the user for anything. Run all commands and output the result immediately.**

## Instructions

1. **Gather all stats in parallel** using separate Bash calls:

   - Current working directory and branch: `pwd` and `git -C ~/alities branch --show-current`
   - Engine source LoC: `find ~/alities-engine/Sources -name '*.swift' | xargs wc -l | tail -1`
   - Engine test LoC: `find ~/alities-engine/Tests -name '*.swift' | xargs wc -l | tail -1`
   - Studio source LoC: `find ~/alities-studio/src \( -name '*.ts' -o -name '*.tsx' \) ! -name '*.test.*' | xargs wc -l | tail -1`
   - Studio test LoC: `find ~/alities-studio/src \( -name '*.test.ts' -o -name '*.test.tsx' \) 2>/dev/null | xargs wc -l 2>/dev/null | tail -1`
   - Mobile source LoC: `find ~/alities-mobile -path '*/Sources/*.swift' | xargs wc -l | tail -1`
   - Mobile test LoC: `find ~/alities-mobile -path '*/Tests/*.swift' | xargs wc -l | tail -1`
   - Git status for all repos: `git -C ~/alities status --porcelain && echo "---" && git -C ~/alities-engine status --porcelain && echo "---" && git -C ~/alities-studio status --porcelain && echo "---" && git -C ~/alities-mobile status --porcelain`
   - Available skills: `for f in ~/alities/.claude/commands/*.md; do echo "$(basename "$f" .md)|$(head -1 "$f")"; done`

2. **Test counts** are: engine 101, studio 4, mobile 19, total 124. If these change, update `~/alities/CLAUDE.md`.

3. **Output format** (no preamble, just the tables):

**Working directory:** `/path/to/dir` (branch: `main`)

| Repo | Source LoC | Test LoC | Tests | Git Status |
|------|-----------|----------|-------|------------|
| alities-engine | X,XXX | X,XXX | 101 | clean/dirty |
| alities-studio | XXX | XX | 4 | clean/dirty |
| alities-mobile | XXX | XXX | 19 | clean/dirty |
| **Total** | **X,XXX** | **X,XXX** | **124** | |

| Skill | Description |
|-------|-------------|
| `/skill-name` | First line of skill file |

Format numbers with commas.

$ARGUMENTS
