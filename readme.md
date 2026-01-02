# GitHub PR Deep-Link & Routing Validator (ExecuteCommand + GitHub Comment)

### 🚀 Quick-Start TL;DR

1.  **Import** the workflow JSON into n8n (Cloud or self-hosted).
2.  **Create a GitHub Personal Access Token** with `repo:public_repo` (or `repo`) scope and add it to n8n credentials.
3.  Open the **“CONFIG - Variables”** node and tweak:
    - `manifestPath` – path to your deep-link manifest (AndroidManifest.xml, Info.plist, etc.).
    - `scriptPath` – helper script that boots the emulator & checks each route.
4.  **Enable** the workflow. Every push to a PR branch triggers validation and posts a Markdown pass/fail matrix back to the PR.

---

### What It Does

This workflow delivers an automated, CI-friendly smoke-test of every deep link defined in your mobile app. On each push to an open GitHub PR, it:

- Clones the PR branch.
- Runs a lightweight validation script (provided) that spins up an emulator/simulator, attempts to open each declared URI, and records OK/FAIL.
- Generates a Markdown table summarizing the results.
- Comments that table in the PR, letting reviewers spot broken schemes at a glance.

---

### Who's It For

- **Mobile teams** maintaining Android or iOS deep-link manifests.
- **CI engineers** who need a simple, language-agnostic check they can publish to each PR.
- **OSS maintainers** wanting a template-library-ready n8n recipe.

---

### Requirements

| Requirement                      | Notes                                                                                         |
| :------------------------------- | :-------------------------------------------------------------------------------------------- |
| **n8n Cloud / CE**               | Works everywhere; self-hosted users need Docker with Android / Xcode if validating on-runner. |
| **GitHub Personal Access Token** | Used for posting PR comments.                                                                 |
| **Emulator-capable runner**      | Local dev hardware or CI image that can run `adb` / `xcrun simctl`.                           |

---

### How It Works

1.  **GitHub Trigger** fires on `pull_request` → `synchronize` (i.e., each push to the PR branch).
2.  **Set (CONFIG - Variables)** centralises repo URL, manifest path, script path, timeout, and comment mode.
3.  **ExecuteCommand** clones the repo and calls the validation script.
4.  **Function** converts CLI CSV output into a Markdown table.
5.  **GitHub node** posts (or appends) the results as a comment on the PR.

---

### How To Set Up

1.  **Auth:** In n8n, add a GitHub credential with your PAT named “GitHub Personal Access Token”.
2.  **Import:** Settings → Import workflow and paste the JSON above.
3.  **Edit Config:** Double-click **CONFIG - Variables** and change any default values.
4.  **Validation Script:** Commit `scripts/validate_deeplinks.sh` into your repo (see sample below).
5.  **Enable** the workflow. Push to any PR branch and watch the comment appear.

---

### Sample `validate_deeplinks.sh`

```bash
#!/usr/bin/env bash
set -e

while getopts "m:" opt; do
  case $opt in
    m) MANIFEST="$OPTARG" ;;
  esac
done

echo "⇨ Parsing deep links from $MANIFEST"

# rudimentary parser—replace with something smarter for XML/Plist
grep -oE 'http[s]?://[^" ]+' "$MANIFEST" | while read -r uri; do
  if adb shell am start -W -a android.intent.action.VIEW -d "$uri" >/dev/null 2>&1; then
    echo "$uri,OK"
  else
    echo "$uri,FAIL"
  fi
done
```
