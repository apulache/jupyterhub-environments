# JupyterHub environments

Builds ARM64 JupyterHub images for `k3s-apulache`. The existing vanilla Python
3.12, 3.13, and 3.14 profiles remain supported. `Dockerfile.workbench` adds a
separate preview profile with shared and personal kernels selected inside
JupyterLab.

## Shared scientific kernel

`kernels/scientific-py313/environment.yml` follows the current Jupyter Docker
Stacks SciPy Notebook package catalog. Its checked-in
`scientific-py313.conda-lock.yml` is resolved only for `linux-aarch64` and is
the sole shared-environment input accepted by `Dockerfile.workbench`.

To update a shared kernel, change its `environment.yml`, regenerate its lock on
the ARM64 platform, add a smoke-test notebook, and submit a reviewed pull
request. GitHub Actions validates the lock, builds/tests an ARM64 image, and
publishes an immutable full-commit tag to GHCR. Copy both the verified tag and
its OCI digest from the workflow summary into the Helm values and runbook.

## Personal kernels

The workbench image includes `jhub-kernel` for environments stored on the
user's Longhorn home PVC:

```bash
jhub-kernel init analytics-py313
# Edit ~/.jupyterhub-kernels/analytics-py313/environment.yml
jhub-kernel lock analytics-py313
jhub-kernel install analytics-py313
jhub-kernel list
```

The registered kernel appears as `Personal: analytics-py313` in JupyterLab.
Commit both `environment.yml` and the generated lockfile to Git if it must be
recreated elsewhere. `jhub-kernel remove analytics-py313` deletes only the
installed environment and kernelspec after confirmation; it keeps the source
specification and lockfile.

Personal environments are not shared automatically. To create a shared kernel,
open a pull request with the environment specification, ARM64 lockfile, README
use case, and smoke-test notebook; an administrator must approve the release.

No CPU or memory limits are configured by design. A package solve or scientific
workload can exhaust an 8 GiB worker, so use one environment build at a time
and monitor worker memory in Rancher.
