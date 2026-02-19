Compare specs in Docs/ against actual implementations across all 3 Alities repos.

## Steps

1. Read `~/alities/Docs/CATALOG.md` to get the list of all spec documents and feature areas.

2. For each spec area, check whether it is implemented in:
   - **Engine**: Look in `~/alities-engine/Sources/AlitiesEngine/` for commands, models, and services
   - **Studio**: Look in `~/alities-studio/src/` for components, pages, and API calls
   - **Mobile**: Look in `~/alities-mobile/Alities/` for views, models, and services

3. For each feature, determine status:
   - `done` — fully implemented matching the spec
   - `partial` — some parts implemented, gaps remain
   - `missing` — not implemented at all
   - `n/a` — not applicable to that repo
   - `no spec` — implemented but no corresponding spec doc

4. Present results as a markdown table:

   | Feature | Spec Doc | Engine | Studio | Mobile | Notes |
   |---------|----------|--------|--------|--------|-------|
   | ... | ... | ... | ... | ... | ... |

5. List any implementations that exist without a corresponding spec ("no spec" items).

6. Suggest priority actions for closing the most critical gaps.

$ARGUMENTS
