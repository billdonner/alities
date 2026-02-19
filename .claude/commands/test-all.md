Run all 3 Alities test suites in parallel and report results.

## Steps

1. Launch these 3 test commands in parallel using the Task tool or parallel Bash calls:
   - `cd ~/alities-engine && swift test`
   - `cd ~/alities-studio && npx vitest run`
   - `cd ~/alities-mobile && swift test`

2. Wait for all 3 to complete.

3. Parse the output of each to extract: total tests, passed, failed, duration.

4. Present results as a markdown table:

   | Repo | Tests | Pass | Fail | Duration |
   |------|-------|------|------|----------|
   | alities-engine | ... | ... | ... | ... |
   | alities-studio | ... | ... | ... | ... |
   | alities-mobile | ... | ... | ... | ... |
   | **Total** | **...** | **...** | **...** | |

5. If any test counts changed from what's recorded in `~/alities/CLAUDE.md`, update the test count table there.

6. Report overall status: ALL PASS or FAILURES DETECTED.

$ARGUMENTS
