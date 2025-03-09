# Versioning

The UltraStar file format is a versioned file format, i.e. songs indicate which version of the file format they conform to.
This is an essential mechanism to future-proof the file format and safely implement changes in the future.
This document describes the semantics of version numbers and how they relate to the introduction of new features.

## History

The original UltraStar file format was never formally specified.
Its specification only existed in the form of different implementations using a similar format.
This original format did not include an explicit file format version, but it serves as the base for the file format defined in this repository.

The community has defined the original UltraStar file format retroactively as the so-called _unversioned_ UltraStar file format (because it does not include an explicit version number).
The unversioned UltraStar file format is the predecessor to all later versions of the file format.

In 2023 the UltraStar file format v1.0.0 was published including a new `#VERSION` header.
The `#VERSION` header indicates the version of the file format a song conforms to.
To improve interoperability with [semver](https://semver.org/lang/de/) tooling the header uses a major.minor.patch triplet to indicate the version, even though it is unlikely that there will ever by a patch version of the file format.

## Revision vs. Version

_Versions_ and _revisions_ describe similar concepts that should not be confused.

The UltraStar file format defines several **versions**.
A version of the file format defines a set of syntax rules and semantics under which a song can be parsed and interpreted.
A song always conforms to a specific _version_ of the UltraStar file format.

The document that defines a specific file format version is not set in stone.
After a version of the file format has been published, limited changes to the specification are still possible (as long as they are backwards-compatible).
A **revision** is a change to the specification after is has already been published.
_Revisions_ are published to amend the specification for a specific version of the UltraStar file format.

### Active and Archived Versions

The multiple versions of the UltraStar file format are maintained simultaneously.
New features and changes are usually introduced to multiple versions.
However, maintaining many different versions of a file format indefinitely becomes unrealistic.

**Active** versions of the file format are those versions that are being actively maintained.
All active versions are eligible for new features.
Song creators and editors are encouraged to always create songs using the latest available version of the file format.
As older versions become less and less relevant, the specification team may decide to transition a version into the _archived_ state.

**Archived** versions of the file format are not being maintained anymore.
Archived versions will not receive any new features and use of archive versions for new songs is discouraged.
Implementations are free to support archived versions as long as they want.
It is not recommended to use newer features with archived versions as their behavior is undefined.

<details>
<summary>Example</summary>

The unversioned UltraStar file format is archived.
Version 1.0.0 of the UltraStar file format introduces the `#PROVIDEDBY` header.
Using the `#PROVIDEDBY` with the unversioned file format is not recommended.
The behavior of the header in the unversioned file format is unsepcified.
Some implementations might respect the header, some might ignore it.

</details>

> [!NOTE]
>
> Archived versions might still receive new revisions with editorial changes.

## Versions indicate Compatibility

The purpose of the `#VERSION` header is to ensure compatibility between songs and implementations.
In face of changes to the format and new features being added this header basically answers the question: "Can software XY open this song"?

Implementations may safely load any song that uses a major version of the file format known to the implementation.
The following guarantees are upheld by the specification team:
- Revisions to a published version of the UltraStar file format will only make backwards-compatible changes.
- Minor versions of the UltraStar file format will only introduce backward-compatible changes compared to the respective major version.

Backward-compatible changes include (but are not limited to):
- Editorial changes
- Standardization of metadata headers
- Increasing the strictness on syntax requirements

An implementation can **not** safely open songs that use a different major version than what they were designed to work with.
Different major versions of the UltraStar file format may have a different syntax or semantics potentially causing any number of problems.
This applies both to higher as well as lower major versions (i.e. an implementation designed to work only with the UltraStar file format version 2.0.0 cannot safely open a song using version 1.0.0).

## Changes & New Features

The specification of the UltraStar file format is constantly evolving.
New enhancements are being proposed and implemented continuously.
During the discussion, changes are classified into 3 categories:

<details>
<summary>Does not impact compatibility</summary>
This type of change adds functionality without impacting existing implementations.

**Example**: Adding a new metadata header (let's say `#COMPOSER`).
A song that uses this header can be opened in a game that does not support the header without experiencing any degradation.
The song works as intended and the header can be safely ignored.
</details>

<details>
<summary>Impacts Compatibility</summary>
This type of change adds functionality that can lead to a degraded experience in older implementations that do not have support for it.

**Example**: Adding a new note type (let's say a vibrato note).
A song that uses this note type can be opened in a game that does not support it and can be sung, but the vibrato notes will not work.
They will be either missing or will be replaced by Freestyle notes.
The song can still be sung, but players will experience some restrictions when newer features are used.
</details>

<details>
<summary>Breaking Change</summary>
This type of change adds functionality that is not backwards compatible.

**Example**: Changing the syntax or semantics of a required header (e.g. removing the implicit quadrupling of the `#BPM` header).
A song that uses the new syntax/semantics cannot be used in a game that does not have support for it.
The song would be completely broken on the old game (in this example because everything would be 4 times as fast).
</details>

Changes are then implemented as follows:

1. Non-impacting changes are introduced in a new **revision** to existing versions of the specification.
   The change is introduced to all active versions it is compatible with.
   In the above example the `#COMPOSER` header could be introduced to all versions of the file format.
   Implementations can choose to support the new header or choose not to.
2. Impacting changes are introduced in a new **minor version** of the file format.
   Although backwards-compatible, these changes can impact the playability of songs.
   The use of minor versions allows implementations to emit warnings while maximizing compatibility.
3. Breaking changes are introduced in a new **major version** of the file format.
   Implementations are expected to reject files that use a different major version than what they are compatible with.

Pull requests should include all active versions a change applies to.

## Practical Guidance for Developers

This section is aimed at developers writing software that interacts with the UltraStar file format.

The versioning strategy is defined such that it is usually sufficient to inspect the major component of the `#VERSION` header.
The major component is indicative of the compatibility guarantees made by the file format.
A typical version check might look like this:

1. Find and parse the `#VERSION` header.
   If it does not match the expected major.minor.patch pattern the file is invalid.
2. Extract the major component of the version.
   Check if the major version is supported by the implementation.
   If it is not supported, stop processing the file altogether.
3. (Optional) Extract the minor component of the version.
   If it is higher than the latest minor version supported by the implementation, emit a warning to the user that some features of the song might not work correctly and load the file.

> [!TIP]
>
> Currently the patch component of the file format has no meaning and is always 0.
> This may change in the future.
> A change in the patch component will, however, at most be as impactful as a change in the minor component.
