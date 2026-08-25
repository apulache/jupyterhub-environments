# JupyterHub environments

Builds the ARM64 hub and Python 3.12/3.13/3.14 single-user images used by `k3s-apulache`.

GitHub Actions publishes immutable image tags to GitHub Container Registry. After the first release, copy each immutable digest into JupyterHub Helm values; do not deploy mutable tags.
