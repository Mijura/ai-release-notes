# ai-release-notes

This repository provides a GitHub Actions workflow that, when a pull request is merged into `main` with a `release:major`, `release:minor`, or `release:patch` label, bumps the semantic version tag, uses OpenAI to turn commits and PR context into Markdown release notes, and publishes a GitHub release.
