# AI Generated Release Notes

**Experimental showcase.** This repository demonstrates how to generate GitHub release notes with an LLM inside [GitHub Actions](https://docs.github.com/en/actions). It is not a supported product or library—copy ideas into your own workflows and adapt them to your policies, budgets, and review process.

## What it does

When a pull request is **merged** into `main`, a workflow:

1. Reads the latest SemVer tag (or assumes `v0.0.0` if none exist).
2. Requires **exactly one** release label on the PR: `release:patch`, `release:minor`, or `release:major`, and computes the next version from it.
3. Collects **commits** since the last tag, a **diff summary**, and a **unified diff** (with lockfiles, build artifacts, coverage, and snapshots excluded).
4. Truncates oversized diffs (currently about 60 KB) so prompts stay bounded.
5. Calls the [OpenAI Responses API](https://platform.openai.com/docs/api-reference/responses) (`gpt-4o-mini`) with the PR title, body, commits, and diff context.
6. Pushes a new **git tag** for the version and creates a **GitHub Release** whose notes are the generated Markdown.

The model is instructed to emphasize **user-facing** changes from the diff and to follow a fixed Markdown template: a version line with injected SemVer and UTC date/time, then **New**, **Changes**, and **Bug Fixes** sections with prescribed bullet patterns; empty sections are omitted.

## Setup

1. Add this workflow under `.github/workflows/` in a repository that releases from `main`.
2. Create a repository secret named **`OPENAI_API_KEY`** with a key that can call the OpenAI API ([API keys](https://platform.openai.com/api-keys)).
3. Ensure the default **`GITHUB_TOKEN`** can create releases (`permissions.contents: write` is already set in the workflow; adjust if your org uses stricter policies).

No application code is required in this repo—the value is the workflow file alone.

## Cutting a release

1. Open a PR targeting `main`.
2. Add **one** label: `release:patch`, `release:minor`, or `release:major`.
3. Merge the PR. The workflow runs on `pull_request` `closed` when `merged == true`.

If the label is missing or duplicated, the job fails before tagging or calling the API.

## Customization ideas

- Change **`model`** or the prompt in `.github/workflows/ai-release-notes.yaml` to match your tone, languages, or corporate template.
- Tune **`MAX_BYTES`** or diff path exclusions if your repos have different noise patterns.
- Replace `curl` + `jq` with a small script or an official SDK if you prefer maintainability over minimal dependencies.

## Limitations and caveats

- **Cost and rate limits** apply for every merged release PR; monitor usage in your OpenAI account.
- **Model output can be wrong or vague**; treat generated notes as a draft and review before sharing widely.
- **Large changesets** are truncated; notes may miss details present only in omitted diff content.
- First-time behavior when there is no prior tag uses a narrower diff window (`HEAD~1..HEAD`) by design in the workflow—you may want to change that for greenfield repos.

Use this pattern only where automated summaries align with your compliance and security requirements (including sending code snippets to a third-party API).

## Repository layout

| Path | Purpose |
|------|---------|
| `.github/workflows/ai-release-notes.yaml` | Release automation and AI prompt |
