---
title: "How to submit a PKGBUILD to the AUR"
date: 2023-02-04T01:00:00-03:00
draft: false
---

# How to submit a PKGBUILD to the AUR

This document provides instructions on creating and sharing a PKGBUILD script
using the AUR (Arch User Repository). While not a comprehensive guide, it aims
to assist individuals with a basic understanding of Arch Linux in sharing their
PKGBUILD scripts..

The manual page for PKGBUILD and the Arch wiki are a good starting point. I
encourage you to read the following articles and maybe you do not need my post:

- https://wiki.archlinux.org/title/PKGBUILD
- https://wiki.archlinux.org/title/Creating_packages
- https://wiki.archlinux.org/title/Arch_package_guidelines
- https://wiki.archlinux.org/title/AUR_submission_guidelines

## Step 1: create an AUR user account

If you intend to share your PKGBUILD (not binary files) through the AUR, it is
necessary to have an account and upload your SSH public key. To create an
account, simply follow the registration steps [here](https://aur.archlinux.org/register).

## Step 2: create a repostitory

This step can be confusing because unlike other registries, the AUR does not
provide an option to create a Git repository. However, you can still share your
PKGBUILD script by following the alternative steps outlined below.

To accomplish this, you can clone an empty repository from the AUR using the
name of your package. Here's how you can proceed:

```sh
git clone ssh://aur@aur.archlinux.org/PKG_NAME.git
```

A good practice is to create the `.gitignore` file that ignores everything
within the repository:

```sh
echo "*" > .gitignore
```

This approach requires explicitly using the `-f` flag to `add` files to your
repository.

```sh
git add -f <file>
```

## Step 3: Write your PKGBUILD

A PKGBUILD script is a shell script that includes the necessary information to
build a package. In Arch Linux, packages are built using the `makepkg` utility.
When `makepkg` is executed, it searches for a `PKGBUILD` file in the current
directory and follows the instructions specified within it to compile or obtain
the files required for creating a package archive, typically named
`PKG_NAME.pkg.tar.zst`. The resulting package consists of binary files along
with installation instructions, which can be easily installed using the command
`pacman -U PKG_NAME.pkg.tar.zst`.

In this document, I do not provide a comprehensive guide on how to write a
`PKGBUILD`. However, you can find more information in the manual pages or refer
to the previous link I mentioned. Additionally, you can use `/usr/share/pacman/PKGBUILD.proto`
as a starting point. It is important to note that this template includes
numerous components that you may not necessarily require for your specific
PKGBUILD.

**_Note_**: You will notice that many other PKGBUILDs, in the `package()`
function, use the `install` command to copy things into the file system. This
is because it can copy files and set permissions in one go.

To update the package checksums in the PKGBUILD file, run `updpkgsums`, which
belongs to the `pacman-contrib` package.

## Step 4: build the package

To build the package, navigate to the directory containing the PKGBUILD file
and execute the following command:

```sh
makepkg -s
```

This command will download the sources and create a `.tar.zst` file along with
the necessary directory structure. Here, the `.gitignore` file plays a crucial
role in preventing irrelevant files from being pushed to the repository. If you
wish to test or install the package, you can execute the following command:

```sh
sudo pacman -U PKG_NAME.pkg.tar.zst
```

## Step 5: Create the .SRCINFO file

The `.SRCINFO` file contains package metadata in a simple and unambiguous
format. This allows tools like the AUR's web back-end or AUR helpers to
retrieve a package's metadata without directly parsing the PKGBUILD file.

To create this file, in the working directory you need to execute the following
command:

```sh
makepkg --printsrcinfo > .SRCINFO
```

## Step 6: Publish to the AUR

```sh
git add -f .SRCINFO PKGBUILD
git commit .SRCINFO PKGBUILD -m "Some cool commit message"
git push origin master
```

 **_Note_**: whenever you make modifications to the PKGBUILD file, it is
 essential to update both the checksums and the `.SRCINFO` file.
