Test all, commit dirty repos, and push everything.

## Steps

1. **Check git status** of all 4 repos in parallel:
   - `cd ~/alities && git status`
   - `cd ~/alities-engine && git status`
   - `cd ~/alities-studio && git status`
   - `cd ~/alities-mobile && git status`

2. **Run all 3 test suites** in parallel:
   - `cd ~/alities-engine && uv run pytest`
   - `cd ~/alities-studio && npx vitest run`
   - `cd ~/alities-mobile && xcodebuild test -project Alities.xcodeproj -scheme Alities -destination 'platform=iOS Simulator,name=iPhone 16 Pro Max,OS=18.5'`

3. **If ANY test suite fails — STOP.** Report the failures and do NOT commit or push anything.

4. **If all tests pass**, for each repo that has uncommitted changes:
   - `git add` the relevant files (not -A; be specific)
   - `git commit` with a descriptive message ending with:
     ```
     Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
     ```
   - `git push`

5. **Report final status table**:

   | Repo | Dirty? | Tests | Committed? | Pushed? |
   |------|--------|-------|------------|---------|
   | alities | ... | n/a | ... | ... |
   | alities-engine | ... | ... | ... | ... |
   | alities-studio | ... | ... | ... | ... |
   | alities-mobile | ... | ... | ... | ... |

$ARGUMENTS
