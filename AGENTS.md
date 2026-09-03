# Repository Guidelines for AI Agents

## Dependabot Security Alerts Workflow

When asked to check, resolve, or audit Dependabot security alerts for this repository, follow this standardized procedure:

### 1. Synchronize with Remote First
Always fetch and rebase against `origin/master` before inspecting or modifying dependencies to ensure you are working on the latest merged PRs:
```bash
git fetch origin
git pull --rebase origin master
```

### 2. Query Open Dependabot Alerts
Use GitHub CLI to list all open Dependabot alerts:
```bash
gh api 'repos/melastmohican/micronaut-angular-spa/dependabot/alerts?state=open' \
  --jq '.[] | {number: .number, severity: .security_advisory.severity, package: .security_vulnerability.package.name, vulnerable_range: .security_vulnerability.vulnerable_version_range, patched_version: .security_vulnerability.first_patched_version.identifier, manifest: .dependency.manifest_path, summary: .security_advisory.summary}'
```

Check the total count of open alerts:
```bash
gh api 'repos/melastmohican/micronaut-angular-spa/dependabot/alerts?state=open' --jq 'length'
```

### 3. Resolving Webapp Vulnerabilities (`src/main/webapp`)
Most frontend dependencies are resolved via `overrides` in `src/main/webapp/package.json`.

- **Check Current Dependency Tree**:
  ```bash
  cd src/main/webapp
  npm ls <package-name>
  ```
- **Update Overrides in `package.json`**:
  - Add or update the package in the `"overrides"` block with a semver constraint satisfying the patched version (e.g., `"browserslist": "^4.28.8"`).
  - Use scoped overrides if different branches of the tree require different major versions (e.g. `@istanbuljs/load-nyc-config: { "js-yaml": "^3.15.2" }` alongside top-level `"js-yaml": "^4.3.2"`).
  - Check for CommonJS/ESM export incompatibilities with legacy test runners (for instance, Karma requires `karma: { "minimatch": "^3.1.2" }` to prevent `TypeError: mm is not a function`).
- **Regenerate Lockfile**:
  ```bash
  cd src/main/webapp
  npm install
  ```
- **Audit Check**:
  ```bash
  npm audit
  ```

### 4. Verification
Always verify build and test suites before committing:
- **Build Frontend**:
  ```bash
  cd src/main/webapp
  npm run build
  ```
- **Run Unit Tests (Headless Chrome)**:
  ```bash
  cd src/main/webapp
  CHROME_BIN="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" npm test -- --watch=false --browsers=ChromeHeadless
  ```

### 5. Git Commit and Push Conventions
- Commit message format:
  ```bash
  git add src/main/webapp/package.json src/main/webapp/package-lock.json src/main/webapp/.gitignore
  git commit -m "fix(security): update dependencies and overrides to resolve remaining Dependabot alerts"
  git push origin master
  ```
- Verify Dependabot alert closure on GitHub after pushing:
  ```bash
  gh api 'repos/melastmohican/micronaut-angular-spa/dependabot/alerts?state=open' --jq 'length'
  ```
  Confirm the count is `0`.
