# Neptu public installation distribution

This repository distributes Global and Site installer releases and public installation guides. Installer implementations remain in their source repositories; image build/publishing tools remain in neptu-ai-images.

- [Installation and version selection](docs/installers.md)
- [Published releases](https://github.com/joysun99/neptu-ai-web/releases)

Release assets are downloadable without GitHub authentication. Private GHCR images still require Docker registry login. Select an existing fixed release from GitHub Releases or the installer version query; replace placeholder version URLs before downloading. New deployment commands require a release explicitly marked `deploymentWorkflow: neptu-deployment-workflow-v1`.

This repository does not contain production configuration, credentials, business source code or runtime data.
