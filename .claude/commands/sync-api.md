Regenerate the API client after alities-engine changes and check for iOS drift.

## Steps

1. **Regenerate OpenAPI spec** from the engine:
   ```bash
   cd ~/alities-engine && uv run python -c "
   from alities.server.main import create_app
   import json
   app = create_app()
   print(json.dumps(app.openapi(), indent=2))
   " > docs/openapi.json
   ```

2. **Copy to alities-studio**:
   ```bash
   cp ~/alities-engine/docs/openapi.json ~/alities-studio/openapi.json
   ```

3. **Regenerate TypeScript client**:
   ```bash
   cd ~/alities-studio && npm run api:generate
   ```

4. **Check iOS model drift**: Compare the OpenAPI spec against:
   - `~/alities-mobile/Alities/Services/APIEndpoint.swift`
   - `~/alities-mobile/Alities/Models/*.swift`

   Report any endpoints or model fields present in the spec but missing from iOS, or vice versa.

5. **Run alities-studio tests** to verify the regenerated client:
   ```bash
   cd ~/alities-studio && npx vitest run
   ```

6. If anything changed, commit and push the affected repos.

7. Report a summary table:

   | Step | Status |
   |------|--------|
   | OpenAPI regenerated | ... |
   | TS client regenerated | ... |
   | iOS drift check | ... |
   | Studio tests | ... |

$ARGUMENTS
