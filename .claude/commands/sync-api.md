Check for API and model drift across all 3 Alities repos.

The engine is a Swift CLI with an internal NIO control server (port 9847). There is no OpenAPI spec generation — sync is done by comparing source code directly.

## Steps

1. **Inventory engine HTTP endpoints**: Read `~/alities-engine/Sources/AlitiesEngine/Services/ControlServer.swift` and list all HTTP routes (method, path, description).

2. **Check studio API client**: Search `~/alities-studio/src/` for any fetch calls, API URLs, or endpoint references. Compare against the engine's actual endpoints.

3. **Check mobile model structs**: Read `~/alities-mobile/` for model structs (e.g., `GameModels.swift`, `TriviaQuestion.swift`, or similar). Compare field names and types against the engine's models in `~/alities-engine/Sources/AlitiesEngine/Models/`.

4. **Report drift table**:

   | Area | Engine | Studio | Mobile | Status |
   |------|--------|--------|--------|--------|
   | Control endpoints | ... | ... | ... | aligned / drifted / missing |
   | Question model | ... | ... | ... | aligned / drifted / missing |
   | GameData model | ... | ... | ... | aligned / drifted / missing |

5. **Run studio tests** to verify nothing is broken:
   ```bash
   cd ~/alities-studio && npx vitest run
   ```

6. If anything changed, commit and push the affected repos.

7. Report a summary table:

   | Step | Status |
   |------|--------|
   | Engine endpoints inventoried | ... |
   | Studio API drift check | ... |
   | Mobile model drift check | ... |
   | Studio tests | ... |

$ARGUMENTS
