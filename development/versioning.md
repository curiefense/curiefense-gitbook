---
description: >-
  This page describes Curiefense's versioning and release process.
---

Curiefense uses [SemVer](https://semver.org/) as its versioning strategy. Find below a quick summary from the SemVer specification:

> Given a version number MAJOR.MINOR.PATCH, increment the:
>
> * MAJOR version when you make incompatible API changes,
> * MINOR version when you add functionality in a backwards compatible manner, and
> * PATCH version when you make backwards compatible bug fixes.
>
> Additional labels for pre-release and build metadata are available as extensions to the MAJOR.MINOR.PATCH format.

The Curiefense architecture depends on the interoperability of multiple components, which requires
there to be a common language and alignment between them in order to guarantee the stability of the
system. Furthermore, it's important for Curiefense's versioning to not make day to day operations
any harder. Therefore, the current Curiefense versioning works as follows:

Major and minor versions are aligned across all components, while patch versions are independent.
This allows for working on bugs, CVEs, and other type of fixes for each component in isolation,
while still keeping an aligned versioning for the overall architecture.

Curiefense's release workflow takes care of updating the `$MAJOR.$MINOR` tags as well as the
`$MAJOR.$MINOR.$PATCH` tags of each component as new releases are cut.
