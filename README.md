[![Cirrus CI - Default Branch Build Status](https://img.shields.io/cirrus/github/containers/podman-machine-wsl-os)](https://cirrus-ci.com/github/containers/podman-machine-wsl-os)

# Podman Machine WSL OS

Source code to build and release the Fedora-based VM image for WSL.

The image is distributed as a [zstd compressed](https://facebook.github.io/zstd/) `.tar` file for both `amd64` and `arm64` architectures.

The CI is triggered every 3 hours **and** at every push to the main branch.

At every update of the disk image:
- a new OCI artifact is pushed to [quay.io/podman/machine-os-wsl](https://quay.io/repository/podman/machine-os-wsl?tab=tags)

## Build Process Details

The source code that builds the WSL image for podman is in this [Cirrus CI manifest](https://github.com/containers/podman-machine-wsl-os/blob/main/.cirrus.yml).

To build the content of the Fedora-based disk the CI execute the following steps:
- Pulls the latest `docker.io/library/fedora`
- Starts a container named `fedora-update` and execute the commands to install Podman and other packages

After that, if the are no changes compared to the last time the CI has been executed (i.e. neither the `docker.io/library/fedora` image nor Podman and other packages have been updated), the build stops. When there are some changes, then a new image disk is built:
- Export the `fedora-update` container content to the file `rootfs.tar` (`podman export --output rootfs.tar fedora-update`)
- Remove the content of the file `/etc/resolv.conf` in `rootfs.tar` and compress it using zstd

If at least one of the 2 images (`amd64` and `arm64`) has been updated then the CI publishes a new release:
- Creates [a new GitHub release](https://github.com/containers/podman-machine-wsl-os/releases), using the build timestamp as version, and uploads the    
- Push the OCI artifact to [quay.io/podman/machine-os-wsl](https://quay.io/repository/podman/machine-os-wsl?tab=tags) using the next version of Podman as the image tag

That's done for both `amd64` and `arm64` architectures. And v5.1 and v5.2 zstd files are identical.

## The Delay with Podman Releases

WSL disk images build is disconnected to Podman release process. 

The WSL disk image build is triggered every 3 hours and it looks for a new version of Podman. But it looks for an update on Fedora stable and that's usually updated a few days after a Podman release.

For example [Podman v5.2.2 has been released on the 2024-08-21](https://github.com/containers/podman/releases/tag/v5.2.2) but [has been included in Fedora 40 stable](https://bodhi.fedoraproject.org/updates/FEDORA-2024-435a743cf7) one week after that, and the updated WSL disk image [has been released on the 2024-08-27](https://github.com/containers/podman-machine-wsl-os/releases/tag/v20240827181401).

## When a new dev version of Podman is bumped on main branch

Before bumping the version of Podman in c/podman, the corresponding WSL image should be pushed to quay.io:

```diff
-    IMAGE_TAG_DEV: "5.4"
+    IMAGE_TAG_DEV: "5.5"
```

## When a new version of Podman is released

After a new version of Podman has been released (i.e. when the fedora package has been updated), the OCI artifact tags should be updated using development Podman version:

```diff
-    IMAGE_TAG_LATEST: "5.3"
-    IMAGE_TAG_NEXT: "5.4"
+    IMAGE_TAG_LATEST: "5.4"
+    IMAGE_TAG_NEXT: "5.5"
```

Note that at that point `IMAGE_TAG_NEXT` and `IMAGE_TAG_DEV` will match.
