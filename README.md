# The UltraStar File Format Specification

This project defines the UltraStar file format.
The UltraStar file format is the standard file format for open source karaoke software.

## Motivation

The UltraStar file format has been used by different karaoke applications for a long time.
However, it was never formally standardized so each application interpreted the format in its own way, leading to fragmentation and differences in feature implementation.
This project is an effort to unite developers in the open source karaoke ecosystem and standardize a common file format based on the original UltraStar file format.

## Goals of this Project

The main goal of this project is to define a standardized file format with clear extension points so that the multitude of different games and applications can be made interoperable.
We aim to reduce the thousands of UltraStar songs that use proprietary or undefined headers or features and make the songs accessible to everyone in the karaoke community.
We hope that this can be a step to unite the various communities that currently each "do their own thing". Let's bond together and see if we can make the best out of this format we all love.

- [x] Formally standardize the format widely used by karaoke games. See [The UltraStar File Format (Unversioned).md](https://github.com/Ultrastar-Deluxe/format/blob/main/The%20UltraStar%20File%20Format%20(Unversioned).md).
- [x] Introduce a versioning strategy for the file format as well as processes for the standardization of new features.
- [ ] Be the authoritative source of the standardized features (headers, note types, ...) of the UltraStar file format.
- [ ] Standardize a modern variant of the format that solves long-standing shortcomings of the original UltraStar format.
- [ ] Unite the fragmented community over a common file format, ensuring interoperability between the various karaoke games and supporting software.

## Join the Discussion

If you are interest in contributing to the karaoke community, you might want to check out the [UltraStar* Creators Community Discord](https://discord.gg/tNEXZw2QJX).
If you want to contribute to this project, check out [CONTRIBUTING.md](https://github.com/UltraStar-Deluxe/format/blob/main/.github/CONTRIBUTING.md).
We are looking forward to your input.

You can also check out our [public status and release board](https://github.com/orgs/UltraStar-Deluxe/projects/3/views/1) and [milestones](https://github.com/UltraStar-Deluxe/format/milestones)!

## Involved parties

There is a lot of software support for the original UltraStar file format.
This overview is not exhaustive, feel free to open a PR to add any missing project.
We always appreciate from the maintainers of projects in the karaoke community looking to improve the file format together.

### Karaoke software
* [UltraStar Online](http://ultrastaronline.com/)
* [UltraStar Deluxe](https://github.com/UltraStar-Deluxe/USDX)
* [Performous](https://github.com/performous/performous)
* [UltraStar World Party](https://github.com/ultrastares/ultrastar-worldparty)
* [Vocaluxe](https://github.com/Vocaluxe/Vocaluxe)
* [MyLittleKaraoke](https://www.mylittlekaraoke.com/)
* [UltraStar Play / Melody Mania](https://github.com/UltraStar-Deluxe/Play)

### UltraStar Song Creator software
* [Yass](https://github.com/SarutaSan72/Yass)
* [Yass Reloaded](https://github.com/DoubleDee73/Yass)
* [Composer](https://github.com/performous/composer)
* [Karedi](https://github.com/Nianna/Karedi)
* [UltraStar Creator](https://github.com/UltraStar-Deluxe/UltraStar-Creator)
* [UltraSinger](https://github.com/rakuri255/UltraSinger)

### UltraStar Textfiles hosting
* [USDB](https://usdb.animux.de)
* [UltraStar-ES](http://ultrastar-es.org/)

### Management Software
* [USDB Syncer](https://github.com/bohning/usdb_syncer)
* [UltraStar Manager](https://github.com/UltraStar-Deluxe/UltraStar-Manager)
