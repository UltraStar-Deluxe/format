# The UltraStar File Format

The UltraStar file format is a timed text format for describing karaoke songs where singing performance is scored by the karaoke software.

## Status of this document

This document aims to describe the UltraStar File Format in its most recent version 2.0.0.

> [!IMPORTANT]
>
> This document is currently a work-in-progress.
> Parts of the final specification may change significantly from the current state of this document.

GitHub Issues are preferred for discussion of this specification.
Alternatively, you can discuss comments on our Discord server.

## 1. Introduction

The UltraStar file format provides a standardized representation of karaoke songs.
The format has been used for a long time by numerous karaoke games such as [UltraStar Deluxe](https://github.com/UltraStar-Deluxe/USDX), [Performous](https://github.com/performous/performous), or [Vocaluxe](https://github.com/Vocaluxe/Vocaluxe).
There exists an ecosystem of supporting applications for hosting, editing, and managing songs.
However, due to the lack of an official file format specification implementations differ and new features cannot be added consistently.
This document aims to fix this problem by providing a formal specification for the syntax and semantics of the UltraStar file format.

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
HTAB  = %x09     ; horizontal tab
SP    = %x20     ; space
```

### 1.2. Terminology

The following terminology is used throughout this document:

**Song**: A song refers to a file in the UltraStar file format.
In some contexts **song** can refer to linked media files as well. In those cases **textfile** is used to disambiguate individual files.

**Medley**: A medley is a short, recognizable excerpt from a song.
Many games include a designated medley mode.

**Implementation**: An implementation is any program or software that interacts with the file format.

**Game Implementation**: A game implementation is an implementation that allows users to sing karaoke songs
and potentially scores singing performance.

> [!CAUTION]
>
> The exact terminology has not been decided yet.

## 2. General Structure

Songs are plain text files
The UTF-8 encoding MUST be used.
Implementations MUST NOT add a byte order mark to the beginning of a file.
In the interests of interoperability, implementations MAY ignore the presence of a byte order mark rather than treating it as an error.

> [!CAUTION]
>
> Whether files with a BOM may be accepted is an open discussion ([#46](https://github.com/UltraStar-Deluxe/format/issues/46)).

The canonical file extension for textfiles is `.txt`.

Songs consist of a header and a body.
The header contains metadata about the song.
The body contains the musical data.
A file SHOULD end with an `E` on a single line.
Everything after a trailing `E` MUST be ignored.

> [!CAUTION]
>
> Whether the trailing `E` is optional or not is open for discussion ([#44](https://github.com/UltraStar-Deluxe/format/issues/44)).

```abnf
song = file-header
       file-body
       [ %x45 *char ]  ; E
char = %x00-10FFFF
```

### 2.1. Line Endings and Whitespace

Both the header and the body of a file are defined in terms of lines.
A line is a string of text that is terminated with an end-of-line sequence.

```abnf
end-of-line = ( CR / LF / CRLF )
empty-line  = end-of-line
```

Implementations SHOULD use a single Line Feed (`%x0A`) as line terminator.
Implementations MUST accept the end of input (`EOF`) as a valid line terminator.
Empty lines are ignored throughout the entire file.

> [!CAUTION]
>
> The allowed end-of-line sequences as well as potential recommendations for implementations are currently open for discussion ([#43](https://github.com/UltraStar-Deluxe/format/issues/43)).

> [!CAUTION]
>
> Whether empty lines are allowed or not is currently open for discussion ([#43](https://github.com/UltraStar-Deluxe/format/issues/43)).
> Whether a line that consists only of whitespace is recognized as an empty line has not been decided yet.

Whitespace is used as a separator in many places of the format.
Only space (`%x20`) and horizontal tab (`%x09`) are recognized as whitespace characters.
In particular other unicode characters with the property `White_Space=yes` MUST NOT be treated as whitespace characters in the context of this specification.

```abnf
WSP = ( SP / HTAB )
```

> [!CAUTION]
>
> Definition and handling of whitespace characters is currently open for discussion ([#46](https://github.com/UltraStar-Deluxe/format/issues/46)).

## 2. The File Header

The header of a song consists of a sequence of key-value pairs.
Each line in the header section starts with a hash.
The key and value of a header are separated by a colon.
Whitespace around key and value is ignored.

> [!CAUTION]
>
> The terminology “header” is not decided yet.
> Other terms used for the same concept are “tag”, “attribute” or “field”.

```abnf
file-header  = *( header / empty-line )
header       = hash *WSP header-key *WSP colon *WSP header-value *WSP line-break
header-key   = header-char
               [ *( WSP / header-char )
                 header-char ]
header-value = single-value / multi-value
multi-value  = single-value [ comma single-value ] [ comma ]
single-value = *( header-char / colon )

header-char  = %x00-09 /  ; exclude line feed
               %x0B-0C /  ; exclude carriage return
               %x0E-39 /  ; exclude colon
               %x3B-10FFFF
hash         = %x23  ; #
colon        = %x3A  ; :
comma        = %x2C  ; ,
period       = %x2E  ; .
```

> [!CAUTION]
>
> The use of trailing commas in comma-separated header lists has not been decided yet.

Comparisons of header keys is case-insensitive.
For the sake of consistency header keys SHOULD use only capital letters.
Header values are generally case-sensitive unless otherwise specified.
An empty value is equivalent to the header being absent.
Implementations MAY remove leading and trailing whitespace in header keys and values without changing semantics.
Header values may not exceed 255 characters.

Implementations MAY define application-specific headers
but SHOULD prefix those headers with the application name to avoid conflicts with future standardized headers.
Implementations MUST ignore headers they do not recognize.
The order of headers is irrelevant (note the exception in section 3.1.)
although standardized headers should precede any application-specific headers.

>  [!CAUTION]
>
> Handling of application-specific headers has not been decided yet.

The following sections describe the standardized headers that have been defined.
If a syntax for a header is specified it applies to the `single-value`.
If no syntax is specified any valid `single-value` is valid.
Some headers are marked as deprecated or removed from a certain version onward.
Implementations MUST continue to apply the defined semantics to a header if it is deprecated in the file format version indicated by the file.
Implementations MUST NOT apply semantics to a header if a file indicates a version where the header has been removed.

### 3.1. Single-Valued and Multi-Valued Headers

Headers can be single-valued or multi-valued.
Single-valued headers can only be specified once and can only contain a single value.
For the sake of robustness implementations SHOULD ignore multiple occurrences of single-valued headers (which value is chosen in such a case is an implementation detail).

Multi-valued headers can contain multiple values separated by a comma (`%x2C`).
Additionally, multiple occurrences of a multi-valued header are semantically equivalent to a single occurrence
where all values are concatenated by commas in order of occurrence.
In this way the order of multi-valued headers is significant.

Empty values within a multi-valued header can be removed without changing semantics.

> [!WARNING]
>
> **Backwards-Compatible Change** in version 1.1.0
>
> Multi-Valued Headers were introduced in version 1.1.0 of the format.
> Headers indicated as multi-valued were single-valued in previous versions.

> [!WARNING]
>
> **Backwards-Compatible Change** in version ???
>
> Concatenating multiple occurrences of multi-valued headers was introduced in version ??? of the format.
> Previously handling of repeated headers was not covered by this specification.

>  [!CAUTION]
>
> The use of multi-valued headers has not been decided yet ([#22](https://github.com/UltraStar-Deluxe/format/issues/22)).

### 3.2. File References

Some headers reference other files, most notably `AUDIO`, `VIDEO`, `COVER`, and `BACKGROUND`.
These file references are always relative to the textfile from which they are referenced.
As a security measure file references MUST NOT use absolute paths.
Implementations SHOULD refuse to load absolutely referenced files.

> [!CAUTION]
>
> Whether absolute paths are allowed or not hasn't been decided yet.

> [!IMPORTANT]
>
> In Windows file names are case-insensitive (i.e. there cannot be two files in a folder that differ only by their case).
> Linux and macOS however, use fully case-sensitive file systems.
> Implementations might need to pay special attention to this fact to ensure that files are compatible across all systems.

### 3.3. The `VERSION` Header

```
Required:     Yes
Multi-Valued: No
Syntax:       <valid semver>
Since:        1.0.0
```

The `VERSION` header indicates the version of this specification that a file complies to.
The value of the tag is a [semantic version](https://semver.org),
meaning that an implementation supporting version 1.0.0 of this spec should be able to process files using a version of 1.1.0 without any changes (although new features might not be supported).
Implementations SHOULD NOT attempt to process files with a higher major version than they were designed to work with.

In absence of the `VERSION` header implementations SHOULD assume the version 0.3.0.
Implementations SHOULD reject a file based on the value of the `VERSION` header,
in particular if the value is syntactically invalid.
Applications MAY reject prerelease-versions altogether.

The `VERSION` header SHOULD be the first header in a file.

### 3.4. The `BPM` Header

```
Required:     Yes
Multi-Valued: No
Syntax:       1*DIGIT [ period 1*DIGIT ]
Since:        0.1.0
```

The `BPM` header indicates the number of beats per minute.
The notes in a song are quantized in beats.
A single beat is the smallest unit of time that can be present in a song.
The value is a 32 bit floating point number.
The value of this tag is arbitrary in the sense that it is usually 4 to 8 times higher than the actual BPM of a song.

> [!WARNING]
>
> **Breaking Change** in version 2.0.0
>
> In versions 0.x and 1.x the value of the `BPM` header was implicitly quadrupled.
> This is **not** the case for version 2.0.0 and above.

> [!CAUTION]
>
> The removal of the implicit quadrupling has not been decided yet.

> [!WARNING]
>
> **Breaking Change** in version 2.0.0
>
> In versions 0.x and 1.x the comma was allowed as decimal separator in addition to the period.
> This feature has been removed in version 2.0.0 and above.

> [!CAUTION]
>
> The removal of the comma as a valid decimal separator has not been decided yet.

### 3.5. The `AUDIO` Header

```
Required:     Yes
Multi-Valued: No
Since:        1.1.0
```

The `AUDIO` header contains a file reference (as defined in section 3.2.) to an audio file.
This file contains the full version of a song (including instrumentals and vocals).
Supported audio formats are an implementation detail.
Implementations MUST disregard the `MP3` header if an `AUDIO` header is present (even if the specified file cannot be found or processed).

> [!CAUTION]
>
> If media files should follow a specific naming scheme has not been decided yet.

### 3.6. The `MP3` Header

```
Required:     Yes (Versions 0.x and 1.x)
Multi-Valued: No
Since:        0.1.0
Deprecated:   1.1.0
Removed:      2.0.0
```

The `MP3` header contains a file reference (as defined in section 3.2.) to an audio file.
This header has been replaced by the `AUDIO` header.

### 3.7. The `COVER`, `BACKGROUND`, and `VIDEO` Headers

```
Required:     No
Multi-Valued: No
Since:        0.2.0
```

The headers `COVER`, `BACKGROUND`, and `VIDEO` contain file references to image files or in case of `VIDEO` video files.
Implementations MAY use these files to display cover artwork and background graphics during gameplay.
Supported image and video formats are an implementation detail.

> [!CAUTION]
>
> If a minimum requirement concerning media types should be part of the spec has not been decided yet.

> [!CAUTION]
>
> If media files should follow a specific naming scheme has not been decided yet.

### 3.8. The `VOCALS` and `INSTRUMENTAL` Headers

```
Required:     No
Multi-Valued: No
Since:        1.1.0
```

The `VOCALS` and `INSTRUMENTAL` header contain file references to audio files.
These files contain the a cappella and instrumental versions of the song respectively.
Implementations MAY use these instead of `AUDIO`
to give users the option of changing the volume of vocal and instrumental tracks separately.

> [!CAUTION]
>
> Whether the inclusion of `VOCALS` requires the inclusion of `INSTRUMENTAL` is currently not decided.

### 3.9. The `GAP` Header

```
Required:     No
Multi-Valued: No
Syntax:       1*DIGIT
Since:        0.2.0
```

The `GAP` header indicates an amount of time in milliseconds from the beginning of the audio track until beat 0.
This effectively offsets all notes in a song by this amount of time relative to the audio track.
The `GAP` value is an integer.

> [!CAUTION]
>
> Whether negative `GAP` values are allowed or disallowed is not decided yet.

> [!WARNING]
>
> **Breaking Change** in version 2.0.0
>
> In versions 0.x and 1.x the value of `GAP` could also be a floating point number.
> Since version 2.0.0 this is not allowed anymore.

### 3.10. The `VIDEOGAP` Header

```
Required:     No
Multi-Valued: No
Syntax:       [ minus ] 1*DIGIT
Since:        0.2.0
```

The `VIDEOGAP` header indicates an amount of time in milliseconds that the background video will be delayed relative to the audio of a song.
The `VIDEOGAP` value is an integer.

> [!WARNING]
>
> **Breaking Change** in version 2.0.0
>
> In versions 0.x and 1.x the value of `VIDEOGAP` was specified in seconds as a floating point number.
> Version 2.0.0 changes this to an integer and milliseconds.

### 3.11. The `NOTESGAP` Header

```
Required:     No
Multi-Valued: No
Syntax:       [ minus ] 1*DIGIT
Since:        0.2.0
Deprecated:   0.3.0
Removed:      1.0.0
```

The `NOTESGAP` header is deprecated and MUST NOT be used.
Implementations MUST ignore the field if present.

> [!NOTE]
>
> The `NOTESGAP` header was defined in version 0.1.0 But its semantics were never fully specified.

### 3.12. The `START` and `END` Headers

```
Required:     No
Multi-Valued: No
Syntax:       1*DIGIT
Since:        0.2.0
```

The `START` and `END` header specify two time points in milliseconds relative to the start of the audio data that indicate a start and end point for the song.
Game implementations SHOULD start and end the song at the specified points and scale scoring accordingly.
Both `START` and `END` values are integers.

> [!NOTE]
>
> The `START` and `END` values do not affect the placement of notes nor any other time codes relative to the audio.
> They simply indicate that a song should be started or stopped a certain amount of time into the audio file.

> [!WARNING]
>
> **Breaking Change** in version 2.0.0
>
> In versions 0.x and 1.x `START` was specified in seconds as a floating point number.
> Version 2.0.0 changes this to an integer and milliseconds.

### 3.13. The `PREVIEWSTART` Header

```
Required:     No
Multi-Valued: No
Syntax:       1*DIGIT
Since:        0.2.0
```

The `PREVIEWSTART` header indicates a time offset in milliseconds relative to the start of the audio where the preview starts.
Implementations MAY use this value when playing a song in a preview setting (e.g. during song selection).
In its absence implementations SHOULD default to the start of the medley section (if available).

> [!WARNING]
>
> **Breaking Change** in version 2.0.0
>
> In versions 0.x and 1.x `PREVIEWSTART` was specified in seconds as a floating point number.
> Version 2.0.0 of this specification changes this to an integer and milliseconds.

### 3.14. The `MEDLEYSTART` and `MEDLEYEND` Headers

```
Required:     No
Multi-Valued: No
Syntax:       1*DIGIT
Since:        2.0.0
```

The `MEDLEYSTART` and `MEDLEYEND` headers indicate in milliseconds the start and end of the medley section of a song relative to the start of the audio.
These tags replace `MEDLEYSTARTBEAT` and `MEDLEYENDBEAT` in version 2.0.0 of this specification.

### 3.15. The `MEDLEYSTARTBEAT` and `MEDLEYENDBEAT` Headers

```
Required:     No
Multi-Valued: No
Syntax:       1*DIGIT
Since:        0.2.0
Removed:      2.0.0
```

The `MEDLEYSTARTBEAT` and `MEDLEYENDBEAT` headers indicate in beats the start and end of the medley section of a song.
Implementations MUST respect the `GAP` value when calculating the medley start and end times.

> [!WARNING]
>
> **Breaking Change** in version 2.0.0
>
> The headers `MEDLEYSTARTBEAT` and `MEDLEYENDBEAT` have been replaced by `MEDLEYSTART` and `MEDLEYEND` in version 2.0.0 of this specification.

### 3.16. The `CALCMEDLEY` Header

```
Required:     No
Multi-Valued: No
Syntax:       "on" / "off"
Since:        0.2.0
```

If `MEDLEYSTART` or `MEDLEYEND` (`MEDLEYSTARTBEAT` or `MEDLEYENDBEAT` for versions before 2.0.0) are not specified,
the `CALCMEDLEY` header indicates whether an implementation is supposed to determine the start and end of the medley section automatically.
The value of this header is compared case-insensitively.

> [!CAUTION]
>
> The exact semantics of the `CALCMEDLEY` header have not been defined yet.

### 3.17. The `TITLE` Header

```
Required:     Yes
Multi-Valued: No
Since:        0.1.0
```

The `TITLE` header contains the title of the song.

### 3.18. The `ARTIST` Header

```
Required:     Yes
Multi-Valued: No
Since:        0.1.0
```

The `ARTIST` header contains the artist of the song.

### 3.19. The `YEAR` Header

```
Required:     No
Multi-Valued: No
Syntax:       4DIGIT
Since:        0.2.0
```

The `YEAR` indicates the year in which the song was released.
The value must be a positive integer.

### 3.20. The `GENRE` Header

```
Required:     No
Multi-Valued: Yes
Since:        0.2.0
```

The `GENRE` defines the genre(s) of the song.
Individual genre values MUST be compared case-insensitively.
For consistency, it is usually best to capitalize genres.

> [!CAUTION]
>
> Whether genres should be compared case-insensitively or not hasn't been decided yet.

### 3.21. The `LANGUAGE` Header

```
Required:     No
Multi-Valued: Yes
Since:        0.2.0
```

The `LANGUAGE` header indicates the spoken or sung language(s) of a song.

### 3.22. The `EDITION` Header

```
Required:     No
Multi-Valued: Yes
Since:        0.2.0
```

The `EDITION` indicates what edition of games a song belongs to.
While this header is intended to hold commercially available editions (e.g. SingStar Pop Hits, GuitarHero Live) implementations MUST NOT reject a file based on the value of this header.
A list of SingStar editions is available [here](https://github.com/bohning/usdb_syncer/wiki/SingStar-Editions).
For arbitrary keywords see the `TAGS` header.

### 3.23. The `TAGS` Header

```
Required:     No
Multi-Valued: Yes
Since:        1.1.0
```

The `TAGS` allow association of any reasonable keyword with a song.
Implementations SHOULD compare tags in a case-insensitive manner.

> [!CAUTION]
>
> Whether tags should be compared case-insensitively or not hasn't been decided yet.

### 3.24. The `P1`, `P2`, … Headers

```
Required:     No
Multi-Valued: No
Since:        0.2.0
```

The headers `P1`, `P2`, … indicate the names of the voices of a song.
These names correspond to the voices indicated by the `P1`, `P2`, … voice changes (see [section 3.3](#33-voice-changes)).
If the voices correspond to different singers in the original song, the header values often indicate the names of the original singers.

The association of header values to voices is defined by the numerical value after each `P` respectively,
i.e. the header `P2` indicates the name of the voice whose notes are introduced by the `P2` voice change.
Leading zeroes are insignificant, i.e. the headers `P1` and `P001` both refer to the same voice.

> [!CAUTION]
>
> The exact semantics of the `P` headers have not been decided yet.

### 3.25. The `DUETSINGER1`, `DUETSINGER2`, … Headers

```
Required:     No
Multi-Valued: No
Since:        0.2.0
Deprecated:   0.3.0
Removed:      1.0.0
```

The headers `DUETSINGER1`, `DUETSINGER2`, etc. are aliases for `P1`, `P2`, etc.
If both are specified `P1`, `P2`, etc. take precedence.

### 3.26. The `CREATOR` Header

```
Required:     No
Multi-Valued: Yes
Since:        0.2.0
```

The `CREATOR` indicates who created the textfile.
Values are usually usernames or gamer tags.

> [!NOTE]
>
> Some implementations are known to use an application-specific header `AUTHOR` in place of `CREATOR`.
> The semantics of the `AUTHOR` header are not part of this specification.

### 3.27. The `PROVIDEDBY` Header

```
Required:     No
Multi-Valued: No
Since:        1.1.0
```

The `PROVIDEDBY` header indicates the source of a particular textfile.
Implementations concerned with providing textfiles to many users (sometimes referred to as "hosters") SHOULD set this value automatically.
Values SHOULD be valid URLs according to [RFC 1738](https://datatracker.ietf.org/doc/html/rfc1738) using the HTTP or HTTPS scheme.

> [!NOTE]
>
> Some implementations are known to use an application-specific header `SOURCE` in place of `PROVIDEDBY`.
> The semantics of the `SOURCE` header are not part of this specification.

### 3.28. The `COMMENT` Header

```
Required:     No
Multi-Valued: No
Since:        0.2.0
```

The `COMMENT` header can include arbitrary text.
Implementations MUST NOT assign semantics to the value of this header.

### 3.29. The `ENCODING` Header

```
Required:     No
Multi-Valued: No
Syntax:       "UTF-8" / "CP1252" / "CP1250"
Since:        0.2.0
Deprecated:   0.3.0
Removed:      1.0.0
```

The `ENCODING` header specifies the encoding used for text values in a textfile.
If present implementations MUST apply this encoding to all header values and all note texts.
Implementations MAY support additional encodings.
Names of encodings are compared in a case-insensitive manner.

> [!IMPORTANT]
>
> Many implementations only apply the specified encoding to **subsequent** headers and note texts.
> Although this is technically not spec-compliant it is usually best to put the `ENCODING` header first.

> [!WARNING]
>
> The use of the `ENCODING` tag is highly discouraged.
> Songs must always use the UTF-8 encoding.

### 3.30. The `RELATIVE` Header

```
Required:     No
Multi-Valued: No
Syntax:       "yes" / "no"
Since:         0.2.0
Deprecated:    0.3.0
Removed:       1.0.0
```

The `RELATIVE` header enables [Relative Mode](#a-relative-mode) (see Appendix A).

## 3. The File Body

The body of a file consists of a sequence of notes, end-of-phrase markers, and voice changes.

```abnf
body = *( note /
          end-of-phrase /
          voice-change /
          empty-line )
```
The sequence of notes and end-of-phrase markers SHOULD appear in ascending order by their start beats.

> [!CAUTION]
>
> Whether the body of a file must or may be sorted is not decided yet.

### 3.1. Notes

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
start-beat = *DIGIT
duration   = *DIGIT
pitch      = [ minus ] *DIGIT
note-text  = 1*( %x20-10FFFF )

minus   = %x2D  ; -
```

> [!CAUTION]
>
> Whether only a single or multiple whitespace character in a row are allowed is currently up for discussion ([#46](https://github.com/UltraStar-Deluxe/format/issues/46)).

The note type indicates how singing the correct or wrong note should affect scoring.
The following sections define standard note types.
Implementations MAY substitute unknown note types with freestyle notes (`F`).
Implementations MUST NOT attach semantics to note types not covered by this specification.

The start beat and duration define the time when a note appears in a song.
Both are indicated in beats (see section 3.3.) relative to offset indicated by the `GAP` header.
The end beat of a note is calculated as its start beat plus its duration.
Notes SHOULD NOT overlap, i.e. the start beat of a note being between the start beat (inclusive) and end beat (exclusive) of another note.

> [!CAUTION]
>
> Whether and how applications may define custom note types hasn't been decided yet.

> [!CAUTION]
>
> Whether negative note starts and/or durations are valid is, hasn't been decided yet.

The pitch of a note is encoded as the number of half-steps relative to C4 (also referred to as middle C).
So a pitch of `5` represent an F4 and a pitch of `-2` represents an A#3.

#### 3.1.1. Regular Notes `:`

A regular note is indicated by the note type `:` (colon, `%x3A`).
A regular note indicates that a certain pitch is to be held for a certain duration.
Game implementations MAY decide to compare pitches independently of the octave (i.e. compare pitches module 12).

#### 3.1.2. Golden Notes `*`

A golden note is indicated by the note type `*` (asterisk, `%x2A`).
Golden note have the same semantics as regular notes.
However, during scoring game implementations SHOULD award more points for golden notes.
The exact scoring behavior is an implementation detail.

#### 3.1.3. Rap Notes `R`

> [!WARNING]
>
> Rap notes are standardized in version 0.2.0 of this specification.

A rap note is indicated by the note type `R` (the letter R, `%x52`).
Rap notes have the same timing semantics as regular notes but are intended or spoken phrases that do not have a defined pitch.
Implementations MUST ignore pitch information on rap notes.

#### 3.1.4. Golden Rap Notes `G`

>  [!WARNING]
>
> Golden rap notes are standardized in version 0.2.0 of this specification.

A golden rap note is indicated by the note type `G` (the letter G, `%x47`).
Golden rap notes have the same semantics as rap notes.
However, during scoring game implementations SHOULD award more points for golden rap notes.
The exact scoring behavior is an implementation detail.

#### 3.1.5. Freestyle Notes `F`

>  [!WARNING]
>
> Freestyle notes are standardized in version 0.2.0 of this specification.

A freestyle note is indicated by the note type `F` (the letter F, `%x46`).
Similar to rap notes, freestyle notes do not carry pitch information.
Additionally, game implementations MUST NOT award points for freestyle notes.

### 3.2. End-of-Phrase Markers

End-of-Phrase markers are indicated by a `-` character (hyphen/minus, `%x2D`).

```abnf
end-of-phrase = dash
                WSP start-beat
                *WSP line-break
```

An end-of-phrase marker carries no musical information but indicates the end of a phrase in the song.
This is usually interpreted as a line break in the lyrics.

An end-of-phrase SHOULD NOT appear between the start beat (inclusive) of a note and its end beat (exclusive).
An end-of-phrase marker SHOULD NOT appear before the start time of the first note or after the start time of the last note.

> [!CAUTION]
>
> Whether leading or trailing end-of-phrase markers are allowed is currently up for discussion ([#44](https://github.com/UltraStar-Deluxe/format/issues/44)).

> [!CAUTION]
>
> Whether there can be non-whitespace text following an end-of-phrase indicator has not been decided yet.

### 3.3. Voice Changes

> [!WARNING]
>
> **Breaking Change** in version 0.2.0
>
> In version 0.1.0 only single-voice songs were defined.
> Voice changes are specified since version 0.2.0.

A voice change (also referred to as a “player change”) is indicated by a `P` (the letter P, `%x50`), immediately followed by a number.

```abnf
voice-change  = p voice-numer
                *WSP line-break
p              = %x50  ; P
voice-number  = positive-digit *DIGIT
positive-digit = %x31-39  ; 1-9
```

A voice change indicates that all notes and end-of-phrase markers following this line belong to the voice indicated by the `voice-number`.
Implementations MAY choose to limit the number of voices.
If the body of a song does not start with a voice change, `P1` is assumed implicitly.
To improve readability notes for different voices should not be interlaced.

> [!NOTE]
>
> A voice change does NOT implicitly add an end-of-phrase indicator.

Voice changes SHOULD appear in ascending order of `voice-number` and there SHOULD be no gaps (i.e. a song having notes for `P1` and `P3`, but not `P2`).
The exact `voice-number` carries no semantics other than its relative order with other `voice-number` and its association with the corresponding header (see [section 3.25](#325-the-p1-and-p2-headers)).
In particular a file that uses `P3` and `P5` can be rewritten using `P1` and `P2` with no change in semantics.

> [!TIP]
>
> A song that makes use of voice changes is referred to as a “duet”.

> [!NOTE]
>
> There exists a legacy behavior where an indicated `P3` would start a sequence of notes that apply to both voices.
> This behavior is explicitly NOT compliant with this specification.

> [!CAUTION]
>
> Whether songs that make use of voice changes need to start their body with a voice change has not been decided yet.

> [!CAUTION]
>
> Whether single-voice songs can have voice changes (only `P1`) has not been decided yet.

> [!CAUTION]
>
> Whether gaps in `voice-number`s are allowed has not been decided yet.

> [!CAUTION]
>
> Whether there may be a whitespace between the `P` and the indicated voice is currently open for discussion ([#46](https://github.com/UltraStar-Deluxe/format/issues/46)).

## Appendix

### A. Relative Mode

> [!WARNING]
>
> Relative mode is deprecated and has been removed in version 1.0.0 of this specification.

Relative mode is a special input mode that affects parsing and interpreting songs significantly.
Relative mode is enabled by the `RELATIVE` header being set to `yes` (case-insensitive).

#### Syntax

When relative mode is enabled, the syntax of end-of-phrase markers changes:

```abnf
end-of-phrase =/ dash
                 WSP start-beat
                 WSP rel-offset
                 *WSP line-break
rel-offset    = *DIGIT
```

Note that the syntax in relative mode is incompatible with the normal syntax.
Implementations MUST NOT try to rectify a missing `RELATIVE` header based on the end-of-phrase markers encountered.

#### Semantics

In relative mode the semantics of start times changes for notes and end-of-phrase markers.

- At the start of the body a relative offset `rel` is initialized to the value of the `GAP` header (or `0` if no `GAP` header exists).
- The start times of notes and end-of-phrase markers are relative to the current `rel` value. The absolute start time is calculated as `rel + start-beat`.
- End-of-phrase markers in relative mode include a `rel-offset`.
  After the start time of the end-of-phrase marker has been interpreted, the `rel-offset` value is added to `rel` for subsequent lines.

> [!IMPORTANT]
>
> In relative mode the order of notes and end-of-phrase markers within a file is significant.

In files with multiple voices each voice has its own `rel` value which is independent of other voices.
The `rel` value for a voice does not reset when a voice change is encountered.
