# Running Zephium on GitHub Actions

This project already ships with its workflows in `.github/workflows/`. You do
not need to write any. You only need to put the code in a GitHub repository.

## 1. Put the code on GitHub

1. Create a new **empty** repository on github.com (no README, no .gitignore,
   no license, because the project already has them). Public is easiest:
   Actions minutes are free on public repositories.
2. Unzip the project, open a terminal **inside the `Zephium-main` folder**
   (the one that contains `Cargo.toml`) and run:

```bash
git init -b main
git add -A
git commit -m "Initial commit"
git remote add origin https://github.com/<your-username>/<your-repo>.git
git push -u origin main
```

Important: `.github/` has to sit at the **top level** of the repository, so
push the contents of `Zephium-main`, not the folder that contains it. If it is
one level too deep, GitHub will not see any workflows.

## 2. Turn Actions on

Repository → **Actions** tab. If GitHub asks, click
*"I understand my workflows, go ahead and enable them"*.
Pushing to `main` starts the **CI** workflow by itself.

## 3. What each workflow does and how to run it

| Workflow | Starts when | What you need |
| --- | --- | --- |
| **CI** (`ci.yml`) | Every push to `main` and every pull request | Nothing. Runs checks on Linux, macOS and Windows. |
| **Windows dev build** (`windows-dev-build.yml`) | You press the button | Nothing. Produces an **unsigned** Windows installer (`.exe`). |
| **CodeQL** (`codeql.yml`) | Push to `main`, weekly, or by hand | Nothing. Security scan. |
| **Security review** (`security-review.yml`) | Weekly, or by hand | Nothing. Opens an issue when the engine-security review deadlines pass (see note below). |
| **Nightly** (`nightly.yml`) | Daily, or by hand | Nothing. Fuzzing, fork-security checks, extended tests and coverage (slow). |
| **Release / Publish stable** | A `v*` tag, or by hand | Apple, SignPath and Tauri signing secrets (see below). |

### Get a Windows installer (the simplest "run it")

1. **Actions** tab → **Windows dev build** (left sidebar).
2. **Run workflow** → pick `main` → **Run workflow**.
3. Wait about 30–60 minutes (the first Rust build is slow; later runs are
   faster because of the cache).
4. Open the finished run → **Artifacts** → download `zephium-windows-setup`.

The installer is not code-signed, so Windows SmartScreen will warn you. Choose
*More info → Run anyway*.

### Run the checks by hand

Actions tab → **CI** → the run for your latest commit. To re-run it, press
**Re-run all jobs**.

### Release builds (only if you want signed releases)

`release.yml` needs these under Settings → Secrets and variables → Actions:

* Secrets: `APPLE_CERTIFICATE`, `APPLE_CERTIFICATE_PASSWORD`,
  `APPLE_SIGNING_IDENTITY`, `APPLE_TEAM_ID`, `APPLE_API_ISSUER`, `APPLE_API_KEY`,
  `APPLE_API_KEY_PATH`, `TAURI_SIGNING_PRIVATE_KEY`,
  `TAURI_SIGNING_PRIVATE_KEY_PASSWORD`, `SIGNPATH_API_TOKEN`,
  `SIGNPATH_ORGANIZATION_ID`
* Variables: `SIGNPATH_RELEASE_CERT_THUMBPRINT`, `SIGNPATH_RELEASE_MODE`,
  `SIGNPATH_SIGNER_SUBJECT`

You also have to replace the updater public key in `desktop/tauri.conf.json`
(`plugins.updater.pubkey`) with your own, generated with
`pnpm tauri signer generate`, otherwise installed apps will only accept updates
signed with the original author's private key. If you only want to try the app,
skip releases and use the Windows dev build above.

## 4. Things to know

* **Linux is not a supported target.** The app builds on macOS and Windows
  only. On Linux, CI runs only the platform-neutral checks (formatting,
  frontend, policy). That is intentional.
* **Security-review deadlines have expired.** The project keeps a dated list
  of minimum WebView2 / WebKit versions, and its review deadline was
  2026-11-17. The weekly *Security review* workflow will open an issue until
  someone re-reviews the advisories and updates the dates (see
  `docs/security-maintenance.md`). CI itself only warns; it does not fail.
  I did not change these dates because a person needs to actually check the
  advisories first.
* **Branch protection** (optional): `.github/REPOSITORY.md` lists the
  required checks (`Changes`, `Workflows`, `Frontend`, `Rust policy`,
  `Rust (macOS)`, `Rust (Windows)`).
* **Pinned actions.** Every action is pinned to a commit hash on purpose.
  Dependabot is configured to update them.
