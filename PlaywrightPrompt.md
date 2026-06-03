# Playwright BDD Test Generator — AI Prompt

---

## INPUTS

1. **Caplink UI Project Path** — Root path of the Angular UI application source code
2. **Regression Testing Project Path** — Root path of the existing Playwright + Cucumber automation project
3. **Port / URL where the UI is running** — e.g., `http://localhost:4200`
4. **Gherkin Feature File** — The `.feature` file containing the BDD scenarios to automate

---

## OUTPUTS

1. **Page Object File** — A new (or updated) TypeScript page class under the existing `pages/` directory, modelled after existing page objects in the project. Only create a new file if no suitable page object already exists.
2. **Step Definition File** — A new (or updated) TypeScript step-definition file under the existing `step-definitions/` directory, reusing existing step bindings wherever an exact or near-exact match exists.
3. **No duplicate test cases** — If a scenario (or an equivalent one) already exists in the project, skip it and log a note explaining why it was skipped.
4. **Endpoint / API catalogue (if applicable)** — If the scenario touches API calls, capture the endpoint paths observed in the UI source or network layer and document them inline as comments.

---

## TASK

### Step 1 — Analyse the UI Project
- Traverse the **UI project path** and identify:
  - Angular components and services relevant to the Gherkin scenarios
  - CSS selectors, `data-testid` attributes, ARIA roles, and other stable locators
  - Any REST/GraphQL API endpoints called by the components (from service files or interceptors)
- Do **not** guess locators. If a reliable locator cannot be found in the source, flag it with a `// TODO: Verify locator` comment.

### Step 2 — Analyse the Automation Project
- Traverse the **automation project path** and build an internal map of:
  - All existing **Page Object** classes and their public methods
  - All existing **step definitions** and their Cucumber expressions / regex patterns
  - Shared **utilities**, **fixtures**, **hooks**, and **helper functions**
  - The `cucumber.json` / `playwright.config.ts` configuration to understand report paths, test tags, and browser setup
- Identify reusable artefacts before writing a single line of new code.

### Step 3 — Read the Gherkin File
- Parse every `Feature`, `Scenario`, `Scenario Outline`, `Background`, and `Examples` block.
- For each step (`Given` / `When` / `Then` / `And` / `But`):
  - **Check** whether a matching step definition already exists (exact or parameterised match).
  - If it exists → **reuse** it; do not re-implement.
  - If it does not exist → **generate** a new step definition.

### Step 4 — Generate Code
Apply the following rules strictly:

| Rule | Detail |
|------|--------|
| **Mirror existing style** | Copy the naming conventions, import order, class structure, and assertion style from existing files. |
| **Page Object pattern** | All UI interactions go through a Page Object method. No raw `page.locator()` calls inside step definitions. |
| **Locator strategy** | Prefer `data-testid` → ARIA role → CSS class (scoped) → XPath (last resort). |
| **Async / await** | Every Playwright call must be properly awaited. |
| **No magic strings** | Repeated selectors or URLs must be stored as constants or constructor properties. |
| **Assertions** | Use `expect` from `@playwright/test`. Soft assertions are allowed only if the existing project uses them. |
| **Tags** | Preserve all Cucumber tags (`@smoke`, `@regression`, etc.) from the Gherkin file. |
| **Imports** | Import only what is used. Tree-shake unused imports. |
| **Skip duplicates** | If the scenario already has coverage, output a comment block: `// SKIPPED: Covered by <existing-file>#<line>` |

### Step 5 — Output Format
Return code in clearly labelled fenced blocks:

```
### FILE: tests/pages/<PageName>.page.ts
<full file content>

### FILE: tests/step-definitions/<feature-name>.steps.ts
<full file content>
```

After the code blocks, append a **Summary** section:
- List of new step definitions created
- List of reused step definitions (with source file references)
- List of skipped scenarios (with reason)
- List of locators flagged as `// TODO: Verify locator`
- List of API endpoints captured

---

## CONSTRAINTS & QUALITY GATES

- **Never overwrite** an existing file without first showing a diff of what changes and why.
- **Never duplicate** a step definition that already exists — import or reuse it.
- **Never hardcode** the base URL; read it from the Playwright config or an environment variable.
- **Never use** `page.waitForTimeout()` for synchronisation — use Playwright's built-in auto-waiting or `expect(...).toBeVisible()`.
- **Always** follow the `async/await` pattern consistently; no floating promises.
- **Always** add a JSDoc comment above each new Page Object method describing its purpose.

---

## EXAMPLE INVOCATION

```
UI Project Path       : C:/Workspace/Venky/caplink-ui/src
QA Project Path       : C:/Workspace/Venky/caplink-regression/
App URL               : http://localhost:4200
Gherkin Feature File  : features/login.feature
```

---

*Generated with the Playwright BDD Test Generator prompt — version 1.0*
