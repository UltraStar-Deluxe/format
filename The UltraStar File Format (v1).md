# The UltraStar File Format

The UltraStar file format is a timed text format for describing karaoke songs where singing performance is scored by the karaoke software.

## Status of this Document

This document describes the major version 1 of the UltraStar file format.
Version 1.0.0 of the UltraStar file format has been officially published in 2023 and can be used.
Any future revisions to this document will only introduce backward-compatible changes.

> [!NOTE]
>
> For historical reasons version 1.1.0 is equivalent to version 1.0.0.

[GitHub Issues](https://github.com/ultrastar-deluxe/format/issues) are preferred for discussion of this specification.
Alternatively, you can discuss comments on our Discord server.

## 1. Introduction

The UltraStar file format is the standard file format for many open source karaoke games, such as [UltraStar Deluxe](https://github.com/UltraStar-Deluxe/USDX), [Performous](https://github.com/performous/performous), or [Vocaluxe](https://github.com/Vocaluxe/Vocaluxe).
There exists an ecosystem of supporting applications for hosting, editing, and managing songs.
The UltraStar file format is designed to be edited by machines and humans alike
and is intended to be easily understood and edited by technical and non-technical users.
This guiding principle is influential for many decisions made during the design process.

### 1.1. Conventions in this Document

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

The grammatical rules in this document are to be interpreted as described in [RFC 5234](https://datatracker.ietf.org/doc/html/rfc5234).
We are using the following core rules:

```abnf
CR    = %x0D     ; carriage return
LF    = %x0A     ; line feed
CRLF  = CR LF
DIGIT = %x30-39  ; 0-9
```

### 1.2. Terminology

The following terminology is used throughout this document:

**Song**: A song refers to a file in the UltraStar file format.
In unambiguous contexts **song** can refer to linked media files as well.

**Medley**: A medley is a short, recognizable excerpt from a song.
Many games include a designated medley mode.

**Implementation**: An implementation is any program or software that interacts with the file format.
Most common types of implementations are **games** (which allow users to sing karaoke songs and potentially score their singing performance), and **editors** (which allow users to create and edit their own songs).

## 2. General Structure

Songs are plain text files
The UTF-8 encoding MUST be used.
Implementations MUST NOT add a byte order mark to the beginning of a file.
In the interests of interoperability, implementations MAY ignore the presence of a byte order mark rather than treating it as an error.
The canonical file extension for textfiles is `.txt`.

Songs consist of a header and a body.
The header contains metadata about the song, the body contains the musical data.
A file SHOULD end with an `E` on a single line.
Everything after a trailing `E` MUST be ignored.

```abnf
song = file-header
       file-body
       [ %x45 end-of-line *char ]  ; E
char = %x00-10FFFF
```

### 2.1. Line Endings and Whitespace

Both the header and the body of a file are defined in terms of lines.
A line is a string of text that is terminated with an end-of-line sequence.

```abnf
WSP         = <any unicode character with White_Space=yes>
end-of-line = ( CR / LF / CRLF )
empty-line  = *WSP end-of-line
```

Implementations SHOULD use a single Line Feed (`%x0A`) as line terminator.
Implementations MUST accept the end of input (`EOF`) as a valid line terminator.
Empty lines are ignored throughout the entire file (note that a line consisting only of whitespace characters is considered empty).

Whitespace is used as a separator in many places of the format.
Any unicode character with the property `White_Space=yes` is a valid whitespace character (except for the carriage return `%x0D` and line feed `%x0A` both of which are considered line breaks). See [Wikipedia](https://en.wikipedia.org/wiki/Whitespace_character) for a list of whitespace characters.
In the interests of interoperability implementations SHOULD use ASCII spaces (`%x20`) as whitespace.

## 3. The File Header

The header of a song consists of a sequence of key-value pairs.
Each line in the header section starts with a hash.
The key and value of a header are separated by a colon.
Whitespace around key and value is ignored.

```abnf
file-header  = *( header-line / empty-line )
header-line  = hash *WSP header-key *WSP colon *WSP header-value *WSP line-break
header-key   = 1*header-char
header-value = *( header-char / colon )

header-char  = %x00-09 /  ; exclude line feed
               %x0B-0C /  ; exclude carriage return
               %x0E-39 /  ; exclude colon
               %x3B-10FFFF
hash         = %x23  ; #
colon        = %x3A  ; :
comma        = %x2C  ; ,
period       = %x2E  ; .
```

Comparisons of header keys is case-insensitive.
For the sake of consistency header keys SHOULD use only capital letters.
Header values are generally case-sensitive unless otherwise specified.
An empty value is equivalent to the header being absent.
Implementations MAY remove leading and trailing whitespace in header keys and values without changing semantics.
This specification imposes no limit on the size of a single header value.
Note that implementations may have technical limitations so it is recommended that neither a header key nor a header value exceeds 2048 bytes.

### 3.1. Types of Headers

Headers are classified into three categories: Core Headers, Extended Headers, and Application-specific headers.

**Core Headers** are headers that are essential to the semantics of the file format.
Implementations MUST implement support for all core headers.
Note that core headers may still be optional.
However, an implementation that ignores a core header that is present will not be able to process the song correctly.

**Extended Headers** are headers that can supplement the karaoke experience.
Extended headers can be safely ignored without fundamentally breaking the semantics of a song.
Implementations SHOULD still implement support for extended headers.

**Application-specific Headers** MAY be defined by implementations.
An implementation that defines and use application-specific headers
SHOULD prefix those headers with the application name and a hyphen (`%x2D`) to avoid conflicts with future standardized headers.
For example an application `Foo Bar` might use the application-specific header `#FOO_BAR-SPEED`.
Future core and extended headers are guaranteed to not include a hyphen.
If an application-specific header at some point becomes standardized, the standardized header takes precedence.

Implementations MUST ignore headers they do not recognize.
The order of headers is irrelevant although standardized headers should precede any application-specific headers.

### 3.2. File References

Some headers reference other files, most notably `#MP3`, `#VIDEO`, `#COVER`, and `#BACKGROUND`.
These file references are always relative to the song from which they are referenced.
As a security measure implementations SHOULD NOT allow the use of absolute paths.

> [!IMPORTANT]
>
> In Windows file names are case-insensitive (i.e. there cannot be two files in a folder that differ only by their case).
> Linux and macOS however, use fully case-sensitive file systems.
> Implementations might need to pay special attention to this fact to ensure that files are compatible across all systems.


### 3.3. Core Headers

This section describes the core headers of the file format.
If a syntax for a header is specified it applies to the `header-value`.
If no syntax is specified any valid `header-value` is valid.

#### 3.3.1. The `#VERSION` Header

```
Required: Yes
Syntax:   "1" period 1*DIGIT period 1*DIGIT
```

The `#VERSION` header indicates the version of this specification that a file complies to.
The value of the header is a triplet of numbers separated by periods.
Similar to [semantic versioning ´](https://semver.org) the version number implicates a certain level of compatibility.

An implementation supporting a specific version X.Y.Z of this specification can safely process all files that indicate the same major version X.Y'.Z' (although newer features might not be supported).
Implementations SHOULD NOT attempt to process files with a higher major version than they were designed to work with.

Implementations SHOULD reject a file based on the value of the `#VERSION` header,
in particular if the value is syntactically invalid.

The `#VERSION` header SHOULD be the first header in a file.

> [!NOTE]
>
> The absence of the `#VERSION` header indicates use of the unversioned UltraStar file format.

#### 3.3.2. The `#BPM` Header

```
Required: Yes
Syntax:   1*DIGIT [ (period / comma) 1*DIGIT ]
```

The notes in a song are quantized in beats.
The `BPM` header indicates the number of beats per minute as a decimal value.
The value is implicitly quadrupled, i.e. a value of `10,5` indicates `42` beats per minute.
A single beat is the smallest unit of time that can be present in a song.
The value of this tag is arbitrary in the sense that it is usually 1 to 2 times higher than the actual BPM of a song.

#### 3.3.3. The `#MP3` Header

```
Required: Yes
```

The `MP3` header contains a file reference (as defined in [Section 3.2](#32-file-references)) to an audio file.
This file contains the full version of a song (including instrumentals and vocals).
Supported audio formats are an implementation detail.

#### 3.3.4. The `#TITLE` Header

```
Required: Yes
```

The `TITLE` header contains the title of the song. Additional information such as recording specification or
categorization is provided using dedicated headers like `RENDITION`, `EDITION`, and `TAGS`.

#### 3.3.5. The `#ARTIST` Header

```
Required: Yes
```

The `#ARTIST` header indicates the artist of the song.

#### 3.3.6. The `#GAP` Header

```
Required: No
Syntax:   1*DIGIT [ (period / comma) 1*DIGIT ]
```

The `GAP` header indicates an amount of time in milliseconds from the beginning of the audio track until beat 0.
This effectively offsets all notes in a song by this amount of time relative to the audio track.

#### 3.3.7. The `#START` and `#END` Headers

```
Required: No
Syntax:   1*DIGIT [ (period / comma) 1*DIGIT ]
```

The `#START` and `#END` header specify two time points in seconds relative to the start of the audio data that indicate a start and end point for the song.
Game implementations SHOULD start and end the song at the specified points and scale scoring accordingly.

> [!NOTE]
>
> The `#START` and `#END` values do not affect the placement of notes nor any other time codes relative to the audio.
> They simply indicate that a song should be started or stopped a certain amount of time into the audio file.

#### 3.3.8. The `#P1` thru `#P9` Headers

```
Required: Yes for multi-voice songs, No for single-voice songs
```

The headers `#P1`, `#P2`, …, `#P9` indicate the names of the voices of a song.
These names correspond to the voices indicated by the `#P1`, `#P2`, …, `#P9` voice changes (see [Section 4.3](#43-voice-changes)).
If the voices correspond to different singers in the original song, the header values often indicate the names of the original singers.

The association of header values to voices is defined by the numerical value after each `P` respectively,
i.e. the header `#P2` indicates the name of the voice whose notes are introduced by the `P2` voice change.
If a song uses the voice change `Pn` the corresponding `#Pn` header is required.

> [!NOTE]
>
> As `P0` is not a valid voice change, the header `#P0` is not specified.

## 4. The File Body

The body of a file consists of a sequence of notes, end-of-phrase markers, and voice changes.

```abnf
body = *( note /
          end-of-phrase /
          voice-change /
          empty-line )
```
The sequence of notes and end-of-phrase markers for each voice SHOULD appear in ascending order by their start beats.

### 4.1. Notes

A note is a musical element in a song.
Each note is defined by its type, start beat, duration, pitch, and text.

```abnf
note = note-type
       WSP start-beat
       WSP duration
       WSP pitch
       WSP note-text
       line-break

note-type  = %x21-22 / %x24-7E  ; Visible ASCII-characters except space and #
start-beat = 1*DIGIT
duration   = 1*DIGIT
pitch      = [ minus ] 1*DIGIT
note-text  = 1*( %x20-10FFFF )

minus   = %x2D  ; -
```

The note type indicates how singing the correct or wrong note should affect scoring.
The following sections define standard note types.

The start beat and duration define the time when a note appears in a song.
Both are indicated in beats relative to offset indicated by the `#GAP` header.
The end beat of a note is calculated as its start beat plus its duration.
Notes SHOULD NOT overlap, i.e. the start beat of a note being between the start beat (inclusive) and end beat (exclusive) of another note.

The pitch of a note is encoded as the number of half-steps relative to `C4` (also referred to as middle C).
So a pitch of `5` represent an `F4` and a pitch of `-2` represents an `A#3`.

> [!NOTE]
>
> The pitches in this paragraph use [scientific pitch notation](https://en.wikipedia.org/wiki/Scientific_pitch_notation).

The text of a note contains the spoken words of that note (usually a single syllable). There are no restrictions on the specific text values, in particular spaces can appear at the beginning or end of the note text (or both).

E.g. before

```abnf
0 4 0 Hello|
8 4 0  World|
```

and after

```abnf
0 4 0 Hello |
8 4 0 World|
```

are semantically equivalent. (the `|` is used to mark the end of the line for visualization purposes in the above example)


#### 4.1.1. Regular Notes `:`

A regular note is indicated by the note type `:` (colon, `%x3A`).
A regular note indicates that a certain pitch is to be held for a certain duration.
Game implementations MAY decide to compare pitches independently of the octave (i.e. compare pitches module 12).

#### 4.1.2. Golden Notes `*`

A golden note is indicated by the note type `*` (asterisk, `%x2A`).
Golden note have the same semantics as regular notes.
However, during scoring game implementations SHOULD award more points for golden notes.
The exact scoring behavior is an implementation detail.

#### 4.1.3. Rap Notes `R`

A rap note is indicated by the note type `R` (the letter R, `%x52`).
Rap notes have the same timing semantics as regular notes but are intended or spoken phrases that do not have a defined pitch.
Implementations MUST ignore pitch information on rap notes.

#### 4.1.4. Golden Rap Notes `G`

A golden rap note is indicated by the note type `G` (the letter G, `%x47`).
Golden rap notes have the same semantics as rap notes.
However, during scoring game implementations SHOULD award more points for golden rap notes.
The exact scoring behavior is an implementation detail.

#### 4.1.5. Freestyle Notes `F`

A freestyle note is indicated by the note type `F` (the letter F, `%x46`).
Similar to rap notes, freestyle notes do not carry pitch information.
Additionally, game implementations MUST NOT award points for freestyle notes.

### 4.2. End-of-Phrase Markers

End-of-Phrase markers are indicated by a `-` character (hyphen/minus, `%x2D`).

```abnf
end-of-phrase = %x2D  ; -
                WSP start-beat
                *WSP line-break
```

An end-of-phrase marker carries no musical information but indicates the end of a phrase in the song.
This is usually interpreted as a line break in the lyrics.

An end-of-phrase SHOULD NOT appear between the start beat (inclusive) of a note and its end beat (exclusive).
An end-of-phrase marker SHOULD NOT appear before the start time of the first note or after the start time of the last note.

An end-of-phrase marker MUST NOT immediately follow another end-of-phrase marker.
In the interests of interoperability implementations MAY ignore subsequent end-of-phrase markers.

### 4.3. Voice Changes

A voice change (also referred to as a “player change”) is indicated by a `P` (the letter P, `%x50`), immediately followed by a single digit.

```abnf
voice-change   = p voice-numer
                *WSP line-break
p              = %x50  ; P
voice-number   = DIGIT
```

A voice change indicates that all notes and end-of-phrase markers following this line belong to the voice indicated by the `voice-number`.
Implementations MAY choose to limit the number of voices.
If the body of a song does not start with a voice change, `P1` is assumed implicitly.
To improve readability notes for different voices should not be interlaced.

> [!NOTE]
>
> A voice change does NOT implicitly add an end-of-phrase indicator.

Voice changes SHOULD appear in ascending order of `voice-number` and there SHOULD be no gaps (i.e. a song having notes for `P1` and `P3`, but not `P2`).
The exact `voice-number` carries no semantics other than its relative order with other `voice-number` and its association with the corresponding header (see [Section 3.3.8](#338-the-p1-thru-p9-headers)).
In particular a file that uses `P3` and `P5` can be rewritten using `P1` and `P2` with no change in semantics.

> [!TIP]
>
> A song that makes use of voice changes is referred to as a “duet”.

A song that uses voice changes MUST also include the appropriate [`#P1` thru `#P9`](#338-the-p1-thru-p9-headers) headers indicating the names of the voices.

## Appendix

### A. Extended Headers

Extended headers are standardized headers used for additional metadata and features in songs.
All extended headers are optional.

#### A.1. The `#AUDIO` Header

The `#AUDIO` header contains a file reference (as defined in [Section 3.2](#32-file-references)) to an audio file.
This file contains the full version of a song (including instrumentals and vocals).
Supported audio formats are an implementation detail.
Implementations SHOULD disregard the [`#MP3`](#333-the-mp3-header) header if an `#AUDIO` header is present (even if the specified file cannot be found or processed).

#### A.2. The `#COVER`, `#BACKGROUND`, and `#VIDEO` Headers

The headers `#COVER`, `#BACKGROUND`, and `#VIDEO` contain file references (as defined in [Section 3.2](#32-file-references)) to image files or in case of `#VIDEO` video files.
Implementations MAY use these files to display cover artwork and background graphics during gameplay.
Supported image and video formats are an implementation detail.

#### A.3. The `#VOCALS` and `#INSTRUMENTAL` Headers

The `#VOCALS` and `#INSTRUMENTAL` header contain file references to audio files (as defined in [Section 3.2](#32-file-references)).
These files contain the a cappella and instrumental versions of the song respectively.
Implementations MAY use these instead of [`#MP3`](#333-the-mp3-header)
to give users the option of changing the volume of vocal and instrumental tracks separately.

#### A.4. The `#VIDEOGAP` Header

```
Syntax: [ minus ] 1*DIGIT [ (period / comma) 1*DIGIT ]
```

The `#VIDEOGAP` header indicates an amount of time in seconds that the background video will be delayed relative to the audio of a song.
Negative values indicate that the indicated amount of time should be skipped at the beginning of the video.

#### A.5. The `#PREVIEWSTART` Header

```
Syntax: 1*DIGIT [ (period / comma) 1*DIGIT ]
```

The `#PREVIEWSTART` header indicates a time offset in seconds relative to the start of the audio where the preview starts.
Implementations MAY use this value when playing a song in a preview setting (e.g. during song selection).
In its absence implementations SHOULD default to the start of the medley section (if available).

#### A.6. The `#MEDLEYSTARTBEAT` and `#MEDLEYENDBEAT` Headers

```
Syntax: 1*DIGIT
```

The `#MEDLEYSTARTBEAT` and `#MEDLEYENDBEAT` headers indicate in beats the start and end of the medley section of a song.
Implementations MUST respect the `GAP` value when calculating the medley start and end times.

#### A.7. The `#YEAR` Header

```
Syntax: 4DIGIT
```

The `YEAR` indicates the year in which the song was released.
The value must be a positive integer.

#### A.8. The `#GENRE` Header

The `#GENRE` defines the genre(s) of the song.
Multiple values can be separated by a comma (`%x2C`).
Individual genre values MUST be compared case-insensitively.
For consistency, it is usually best to capitalize genres.

#### A.9. The `#LANGUAGE` Header

The `#LANGUAGE` header indicates the spoken or sung language(s) of a song.
Multiple values can be separated by a comma (`%x2C`).
Valid values for this header are the english language names according to [ISO 639-2](https://www.loc.gov/standards/iso639-2/php/code_list.php).
`#LANGUAGE` values are compared case-insensitively.

#### A.10. The `#EDITION` Header

The `#EDITION` indicates a curated list of where a song belongs to.
Multiple values can be separated by a comma (`%x2C`).
Implementations MUST NOT reject a file based on the value of this header.
The curated list contains:

- Charts
- Christmas
- Club
- Cover
- Disney
- Duet
- Eurovision Song Contest
- Explicit
- Fan Song
- Feel-Good
- Funny
- Guilty Pleasure
- Halloween
- Heartbreak
- Live
- Love Song
- Mainstream
- Movies
- Musical
- Party
- Pride/LGBTQ
- Relaxed
- Slow
- Song-checked
- Special interest
- Summer
- TV Show
- Underground
- Underrated
- Video Game
- Viral Hit

A list of eligable SingStar editions is available [here](https://github.com/bohning/usdb_syncer/wiki/SingStar-Editions).
A list of eligable RockBand editions is available [here](https://github.com/bohning/usdb_syncer/wiki/RockBand-Editions).
A list of eligable Guitar Hero editions is available [here](https://github.com/bohning/usdb_syncer/wiki/GuitarHero-Editions).
For arbitrary keywords see the [`#TAGS`](#a11-the-tags-header) header.

#### A.11. The `#TAGS` Header

The `#TAGS` allow association of any reasonable keyword with a song.
Multiple values can be separated by a comma (`%x2C`).
Tags are compared in a case-insensitive manner.

#### A.12. The `#CREATOR` Header

The `#CREATOR` indicates who created the UltraStar song.
Multiple values can be separated by a comma (`%x2C`).
Values are usually usernames or gamer tags.

> [!NOTE]
>
> Some implementations are known to use an application-specific header `#AUTHOR` in place of `#CREATOR`.
> The semantics of the `#AUTHOR` header are not part of this specification.

#### A.13. The `#PROVIDEDBY` Header

The `#PROVIDEDBY` header indicates the source of a particular song.
Implementations concerned with providing songs to many users (sometimes referred to as "hosters") SHOULD set this value automatically.
Values SHOULD be valid URLs according to [RFC 1738](https://datatracker.ietf.org/doc/html/rfc1738) using the HTTP or HTTPS scheme.

#### A.14. The `#COMMENT` Header

The `#COMMENT` header can include arbitrary text.
Implementations MUST NOT assign semantics to the value of this header.

#### A.15. The `#AUDIOURL`, `#VIDEOURL`, `#COVERURL` and `#BACKGROUNDURL` Header

```
Syntax: URL
```

The `#AUDIOURL`, `#VIDEOURL`, `#COVERURL` and `#BACKGROUNDURL` contain URLs according to [RFC 1738](https://datatracker.ietf.org/doc/html/rfc1738).
Implementations MAY use the values of these headers to locate missing file references.

### A.16. The `RENDITION` Header

The `RENDITION` header allows a song to be precisely associated with a specific recording.
Artist and title information alone are insufficient, as multiple performances of the same song by
the same artist—or even different edits of the same performance—are common.

Examples:

-   video version
-   album version
-   live
-   live (Malmö 2024)
-   radio edit
-   extended version
-   uncensored

## Revision History

- 2023-12-01: Publication of the UltraStar file format v1.0.0
- 2024-03-03: First revision of this document
- 2024-05-05: Clarification of the `#VERSION` header
- 2025-01-22: Some paragraphs have been rewritten for clarification
- 2025-02-01: Standardization of application-specific headers
- 2025-03-07: Major revision wrt wording and structure
- 2025-04-20: The `RENDITION` header
- 2025-04-22: Clarification of the meaning of spaces before/after syllables
- 2025-04-22: Fixed section linking
