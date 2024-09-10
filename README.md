![Cirrus CI - Default Branch Build Status](https://img.shields.io/cirrus/github/containers/podman-machine-wsl-os)

# Podman Machine WSL OS

Source code to build and release the Fedora-based VM image for WSL.

The image is distributed as a [zstd compressed](https://facebook.github.io/zstd/) `.tar` file for both `amd64` and `arm64` architectures.

The CI is triggered every 3 hours **and** at every push to the main branch.

At every update of the disk image:
- a new [GitHub release](https://github.com/containers/podman-machine-wsl-os/releases) is created and the new zstd-compressed disk images is added.
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

## Always publish current and old version of the disk on GitHub 

Podman `v5.y.z` downloads `v5.y` of the WSL OS image from the **latest** release on this GitHub repository. For example Podman `v5.1.2` downloads the file `5.1-rootfs-amd64.tar.zst` (or the arm64 equivalent) from the latest release.

That means that every GitHub release needs to include WSL disk images for old and new versions of Podman 5.

## When a new version of Podman is released

When there is a new version of Podman, a new version of the Disk Image should be included in the releases:

```diff
-    # temporary 5.1, 5.2, 5.3 clone until divergance occurs
+    # temporary 5.1, 5.2, 5.3, 5.4 clone until divergance occurs
    for arch in amd64 arm64; do
      cp $VER_PFX-rootfs-$arch.tar.zst 5.1-rootfs-$arch.tar.zst
      cp $VER_PFX-rootfs-$arch.tar.zst 5.2-rootfs-$arch.tar.zst
      cp $VER_PFX-rootfs-$arch.tar.zst 5.3-rootfs-$arch.tar.zst
+     cp $VER_PFX-rootfs-$arch.tar.zst 5.4-rootfs-$arch.tar.zst
      cp $VER_PFX-latest-$arch 5.1-latest-$arch
      cp $VER_PFX-latest-$arch 5.2-latest-$arch
      cp $VER_PFX-latest-$arch 5.3-latest-$arch
+     cp $VER_PFX-latest-$arch 5.4-latest-$arch
```

And the OCI artifact tag should be updated using development Podman version:

```diff
-    IMAGE_TAG: "5.3"
+    IMAGE_TAG: "5.4"
```
