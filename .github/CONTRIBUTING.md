# Contributing to the UltraStar File Format

First off, thanks for taking the time to contribute! ❤️

All types of contributions are encouraged and valued.
See the [Table of Contents](#table-of-contents) for different ways to help and details about how this project handles them.
Please make sure to read the relevant section before making your contribution.
It will make it a lot easier for us maintainers and smooth out the experience for all involved.
The community looks forward to your contributions. 🎉

> And if you like the project, but just don't have time to contribute, that's fine. There are other easy ways to support the project and show your appreciation, which we would also be very happy about:
> - Star the project
> - Tweet about it
> - Refer this project in your project's readme
> - Mention the project at local meetups and tell your friends/colleagues

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [I Have a Question](#i-have-a-question)
- [I Want To Contribute](#i-want-to-contribute)
    - [Join the Discussion & Vote](#join-the-discussion-vote)
    - [Suggesting Enhancements](#suggesting-enhancements)
    - [Improving The Website](#improving-the-website)
- [Repository Overview](#repository-overview)
- [Styleguides](#styleguides)
    - [Writing Style](#writing-style)
    - [Formatting](#formatting)
    - [Terminology](#terminology)
    - [Commit Messages](#commit-messages)

## Code of Conduct

Please review our [Code of Conduct](https://github.com/ultrastar-deluxe/format/blob/main/CODE_OF_CONDUCT.md).
It is in effect at all times.
We expect it to be honored by everyone who contributes to this project.
Acting like an asshole will not be tolerated.

## I Have a Question

This project concerns itself with the development of the UltraStar file format.
If you have a question about a specific program that uses the UltraStar file format, you are probably better off to ask your question there.

If you have a question about the file format, it is best to search for existing [Issues](https://github.com/ultrastar-deluxe/format/issues) that might help you.
In case you have found a suitable issue and still need clarification, you can write your question in this issue.
If you can't find a suitable issue, you can [open a new issue](https://github.com/ultrastar-deluxe/format/issues/new) or ask us on the [UltraStar discord server](https://discord.gg/tNEXZw2QJX).

## I Want To Contribute

> ### Legal Notice
> When contributing to this project, you must agree that you have authored 100% of the content, that you have the necessary rights to the content and that the content you contribute may be provided under the project license.

### Join the Discussion & Vote

The easiest way to contribute to the development of the file format is to join the discussion via [GitHub issues](https://github.com/ultrastar-deluxe/format/issues).
All changes to the specification are openly discussed by the community and every input is valuable.
If you have an opinion on a specific suggestion, leave a reaction or a comment to show your support.
Or maybe there is a caveat to a suggestion that hasn't been discussed and you can provide insight.

Before changes to the specification are officially implemented there are two more steps where contribution is possible:
- At the end of the discussion about a change there will be a vote if and how the change should be implemented.
  You can participate in the vote by adding your reaction.
  Votes are also announced in the `#song-format` [discord channel](https://discord.gg/tNEXZw2QJX).
- The specific change is then opened as a pull request which you can review and comment on.
  You can comment on the changes, for example to suggest a different wording.

### Suggesting Enhancements

Enhancement suggestions and other potential changes to the UltraStar file format are tracked as [GitHub issues](https://github.com/ultrastar-deluxe/format/issues).
Before submitting an enhancement, perform a [search](https://github.com/ultrastar-deluxe/format/issues) to see if the enhancement has already been suggested.
If it has, add a comment to the existing issue instead of opening a new one.

Your enhancement suggestion should include the following:
- Use a **clear and descriptive title** for the issue to identify the suggestion.
- Include a **motivation** why the change would improve the file format.
- Pay attention to the **impact on existing implementations**. How hard would it be to implement your suggestion?

<!-- TODO: Link issue templates -->

### Improving The Website

In addition to the formal specification this repository also contains the source code for the [Website](https://usdx.eu/format/) in the `website` branch.
The website contains a condensed version of the currently available features.
You can suggest changes to the website by opening an issue.
You can submit suggestions for improving the website by opening a pull request with your changes.

## Repository Overview

This repository consists of multiple active branches:

- Branch `main` contains the most recent published revision of the UltraStar file format.
- Branch `develop` contains the most recent changes to the UltraStar file format, including unpublished revisions.
- Branch `website` contains the source code for the [Website](https://usdx.eu/format/).

Pull requests that target the website can directly target the `website` branch.
Pull requests that introduce changes to the specification should target the `develop` branch.
After a change has been implemented it will usually not be published to `main` immediately.
Instead there is a grace period to allow the latest changes to be revisited, updated, or, in some cases, reverted.

At some point, the current changes in the `develop` branch will be _published_ by the specification team.
Publication includes:
- Updating the revision history of the specification files.
- Creating a pull request targeting the `main` branch.
  Usually this is a merge from `develop` into `main` but can also be an independent PR that only includes a subset of the changes in `develop`.
- Merging the changes into `main`.

As soon as changes are merged into `main` (and only then) they become official.

## Styleguides

### Writing Style

The UltraStar file format specification is targeted at developers that implement software interacting with the file format.
Although song creators might find the specification useful as well,
we expect that editors will abstract away most of the technical details and offer its users a simplified view on the features of the file format.
We expect the [website](https://usdx.eu/format/) to be a more useful resource for song creators.

The UltraStar file format specification is written in a neutral style sometimes referred to as [technical writing](https://en.wikipedia.org/wiki/Technical_writing):
- Try to keep sentences short and simple.
- Don't be verbose but avoid ambiguities.
- It is better to err on the side of clarity than conciseness.
- Each paragraph should only relate to a single subject or aspect.
There are many online resources to learn technical writing such as Google's [Technical Writing Courses](https://developers.google.com/tech-writing).

Syntactic elements of the specification use the [Augmented Backus-Naur form](https://en.wikipedia.org/wiki/Augmented_Backus–Naur_form) (ABNF).
All syntax definitions should include the corresponding ABNF definition as well.
If there are syntax elements that are very verbose or cannot be expressed in ABNF you can include a more intuitive description as well.

### Formatting

The specification is written using [GitHub Flavored Markdown](https://docs.github.com/en/get-started/writing-on-github).
When writing please follow these guidelines:
- Ensure that headings are numbered consistently (i.e. no skipped numbers or inconsistent numbers between headings and sub-headings).
- When referring to headers, use the code font and include the `#` sign, i.e. `#BPM` instead of `BPM`.
- Use highlights such as bold font or italics sparingly.

### Terminology

The UltraStar file format uses terminology that should be easily understandable for most developers.
It is important that the specification uses a consistent terminology.
Try to follow these guidelines when writing parts of the specification:
- Use the term **song** when referring to the contents of an UltraStar `.txt` file.
- Use the term **implementation** instead of application, app, or program.
  If you need to distinguish between implementations that read or write songs, you can use the terms **game** (reading) or **editor** (writing).
- Use the term **version** when referring to specific versions of the UltraStar file format.
  Use the term **revision** when referring to a specific revision of a version.
  See the [versioning strategy](https://github.com/UltraStar-Deluxe/format/blob/main/VERSIONING.md) for details.
- Avoid the term _released_ and _release_.
  Use the term **published** and **publication** when referring to versions and revisions of the specification instead.

### Commit Messages

Please [write a great commit message](https://cbea.ms/git-commit/).

1. Separate subject from body with a blank line
2. Limit the subject line to 50 characters
3. Capitalize the subject line
4. Do not end the subject line with a period
5. Use the imperative mood in the subject line (example: "Fix grammar issue")
6. Wrap the body at about 72 characters
7. Use the body to explain **why**, not _what and how_ (the code shows that)

```
[TAG] Short summary of changes in 50 chars or less

Add a more detailed explanation here, if necessary. Possibly give
some background about the issue being fixed, etc. The body of the
commit message can be several paragraphs. Further paragraphs come
after blank lines and please do proper word-wrap.

Wrap it to about 72 characters or so. In some contexts,
the first line is treated as the subject of the commit and the
rest of the text as the body. The blank line separating the summary
from the body is critical (unless you omit the body entirely);
various tools like `log`, `shortlog` and `rebase` can get confused
if you run the two together.

Explain the problem that this commit is solving. Focus on why you
are making this change as opposed to how or what. The code explains
how or what. Reviewers and your future self can read the patch,
but might not understand why a particular solution was implemented.
Are there side effects or other unintuitive consequences of this
change? Here's the place to explain them.

 - Bullet points are okay, too

 - A hyphen or asterisk should be used for the bullet, preceded
   by a single space, with blank lines in between

Note the fixed or relevant GitHub issues at the end:

Resolves: #123
See also: #456, #789
```

## Attribution
This guide is based on the [contributing.md](https://contributing.md/generator)!
