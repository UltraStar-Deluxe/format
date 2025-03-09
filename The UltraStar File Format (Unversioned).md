# The UltraStar File Format

The UltraStar file format is a timed text format for describing karaoke songs where singing performance is scored by the karaoke software.

## Status of this Document

This document describes the unversioned UltraStar file format.
The unversioned UtraStar file format is widely used without a formal specification.
This document aims to provide a reference for existing songs as well as a baseline for future versions of the file format.
Note that version 1.0.0 of the file format is generally backward-compatible and can be used in almost all legacy applications.

> [!IMPORTANT]
>
> The unversioned UltraStar file format is archived.
> According to the [versioning policy](https://github.com/UltraStar-Deluxe/format/blob/main/VERSIONING.md) no new features and changes will be introduced into this document.

[GitHub Issues](https://github.com/ultrastar-deluxe/format/issues) are preferred for discussion of this specification.
Alternatively, you can discuss comments on our Discord server.

## 1. Introduction

The UltraStar file format is the standard file format for many open source karaoke games, such as [UltraStar Deluxe](https://github.com/UltraStar-Deluxe/USDX), [Performous](https://github.com/performous/performous), or [Vocaluxe](https://github.com/Vocaluxe/Vocaluxe).
There exists an ecosystem of supporting applications for hosting, editing, and managing songs.
However, for the longest time there has been no formal specification of the file format, slowly leading to fragmentation and potential incompatibilities.
This document aims to capture the current state of the UltraStar file format as it is supported by most implementations.
This specification serves as a baseline for future development of the file format and for implementations wanting to support legacy files that have not been updated to one of the versioned file formats.

### 1.1. Conventions in this Document

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

The grammatical rules in this document are to be interpreted as described in [RFC 5234](https://datatracker.ietf.org/doc/html/rfc5234).
The following core rules are being used:

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

Songs are plain text files.
The UTF-8 encoding MUST be used.
Implementations MUST NOT add a byte order mark to the beginning of a file.
In the interests of interoperability, implementations MAY ignore the presence of a byte order mark rather than treating it as an error.
The canonical file extension songs is `.txt`.

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
WSP         = %x20 / %x09
end-of-line = ( CR / LF / CRLF )
empty-line  = *WSP end-of-line
```

Implementations SHOULD use a single Line Feed (`%x0A`) as line terminator.
Implementations MUST accept the end of input (`EOF`) as a valid line terminator.
Empty lines are ignored throughout the entire file (note that a line consisting only of whitespace characters is considered empty).

Whitespace is used as a separator in many places of the format.
Valid whitespace characters are the ASCII space and tab.
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

The following sections describe the standardized headers that have been defined.
If a syntax for a header is specified it applies to the `single-value`.
If no syntax is specified any valid `single-value` is valid.

### 3.1. File References

Some headers reference other files, most notably `#MP3`, `#VIDEO`, `#COVER`, and `#BACKGROUND`.
These file references are always relative to the song from which they are referenced.
As a security measure implementations SHOULD NOT allow the use of absolute paths.

> [!IMPORTANT]
>
> In Windows file names are case-insensitive (i.e. there cannot be two files in a folder that differ only by their case).
> Linux and macOS however, use fully case-sensitive file systems.
> Implementations might need to pay special attention to this fact to ensure that files are compatible across all systems.

### 3.2. The `#VERSION` Header

The `#VERSION` header is **not** a standardized header in the Unversioned UltraStar file format.
It is included here only as a reference.
The versioned UltraStar file formats use this header to indicate which version of the file format is being used.
The presence of this header is enough to indicate that a specific file format version is being used.

### 3.3. The `#BPM` Header

```
Required: Yes
Syntax:   1*DIGIT [ (period / comma) 1*DIGIT ]
```

The notes in a song are quantized in beats.
The `BPM` header indicates the number of beats per minute as a decimal value.
The value is implicitly quadrupled, i.e. a value of `10,5` indicates `42` beats per minute.
A single beat is the smallest unit of time that can be present in a song.
The value of this tag is arbitrary in the sense that it is usually 1 to 2 times higher than the actual BPM of a song.

### 3.4. The `#MP3` Header

```
Required: Yes
```

The `MP3` header contains a file reference (as defined in [Section 3.1.](#31-file-references)) to an audio file.
Supported audio formats are an implementation detail.

### 3.5. The `#TITLE` Header

```
Required: Yes
```

The `TITLE` header indicates the title of the song.

### 3.6. The `#ARTIST` Header

```
Required: Yes
```

The `ARTIST` header indicates the artist of the song.

### 3.7. The `#COVER`, `#BACKGROUND`, and `#VIDEO` Headers

```
Required: No
```

The headers `#COVER`, `#BACKGROUND`, and `#VIDEO` contain file references to image files or in case of `#VIDEO` video files.
Implementations MAY use these files to display cover artwork and background graphics during gameplay.
Supported image and video formats are an implementation detail.

### 3.8. The `#GAP` Header

```
Required: No
Syntax:   1*DIGIT [ (period / comma) 1*DIGIT ]
```

The `GAP` header indicates an amount of time in milliseconds from the beginning of the audio track until beat 0.
This effectively offsets all notes in a song by this amount of time relative to the audio track.

### 3.9. The `#VIDEOGAP` Header

```
Required: No
Syntax:   [ minus ] 1*DIGIT [ (period / comma) 1*DIGIT ]
```

The `#VIDEOGAP` header indicates the number of seconds that the background video will be delayed relative to the audio of a song.
Negative values indicate that the indicated amount of time should be skipped at the beginning of the video.

### 3.10. The `#START` and `#END` Headers

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

### 3.11. The `#PREVIEWSTART` Header

```
Required: No
Syntax:   1*DIGIT [ (period / comma) 1*DIGIT ]
```

The `#PREVIEWSTART` header indicates a time offset in seconds relative to the start of the audio where the preview starts.
Implementations MAY use this value when playing a song in a preview setting (e.g. during song selection).
In its absence implementations SHOULD default to the start of the medley section (if available).

### 3.12. The `#MEDLEYSTARTBEAT` and `#MEDLEYENDBEAT` Headers

```
Required: No
Syntax:   1*DIGIT
```

The `#MEDLEYSTARTBEAT` and `#MEDLEYENDBEAT` headers indicate in beats the start and end of the medley section of a song.
Implementations MUST respect the `#GAP` value when calculating the medley start and end times.

### 3.13. The `#YEAR` Header

```
Required: No
Syntax:   4DIGIT
```

The `#YEAR` indicates the year in which the song was released.
The value is a positive integer.

### 3.14. The `#GENRE` Header

```
Required: No
```

The `#GENRE` defines the genre(s) of the song.
For consistency, it is usually best to capitalize genres.

### 3.15. The `#LANGUAGE` Header

```
Required: No
```

The `#LANGUAGE` header indicates the spoken or sung language(s) of a song.

> [!NOTE]
>
> Later versions of the file format impose restrictions on the set of valid values for this header.

### 3.16. The `#EDITION` Header

```
Required: No
```

The `#EDITION` is an arbitrary categorization value.

> [!NOTE]
>
> Later versions of the file format impose restrictions on the set of valid values for this header.

### 3.17. The `#P1` and `#P2` Headers

```
Required: No
```

The headers `#P1` and `#P2` indicate the names of the voices of a song.
These names correspond to the voices indicated by the `#P1` and `#P2` voice changes (see [Section 3.3](#33-voice-changes)).
If the voices correspond to different singers in the original song, the header values often indicate the names of the original singers.

> [!NOTE]
>
> As `P0` is not a valid voice change, the header `P0` is not specified.

### 3.18. The `#DUETSINGERP1` and `#DUETSINGERP2` Headers

```
Required: No
```

The headers `#DUETSINGERP1` and `#DUETSINGERP2` are aliases for [`#P1` and `#P2`](#317-the-p1-and-p2-headers).
If both are specified [`#P1` and `#P2`](#317-the-p1-and-p2-headers) take precedence.

### 3.19. The `#CREATOR` Header

```
Required: No
```

The `#CREATOR` indicates who created the UltraStar song.
Values are usually usernames or gamer tags.

> [!NOTE]
>
> Some implementations are known to use an application-specific header `#AUTHOR` in place of `#CREATOR`.
> The semantics of the `AUTHOR` header are not part of this specification.

### 3.20. The `#COMMENT` Header

```
Required: No
```

The `#COMMENT` header can include arbitrary text.
Implementations MUST NOT assign semantics to the value of this header.

### 3.21. The `#ENCODING` Header

```
Required: No
Syntax:   "UTF-8" / "CP1252" / "CP1250"
```

The `ENCODING` header specifies the encoding used for text values in a song.
If present implementations MUST apply this encoding to all header values and all note texts.
Implementations MAY support additional encodings.
Names of encodings are compared in a case-insensitive manner.

> [!IMPORTANT]
>
> Many implementations only apply the specified encoding to **subsequent** headers and note texts.
> Although this is technically not spec-compliant it is usually best to put the `ENCODING` header first.

> [!WARNING]
>
> Use of the `#ENCODING` tag is highly discouraged.
> Songs must always use the UTF-8 encoding.

### 3.22. The `#RELATIVE` Header

```
Required: No
Syntax:   "yes" / "no"
```

The `#RELATIVE` header enables [Relative Mode](#a-relative-mode) (see Appendix A).

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
Both are indicated in beats (see section 3.3.) relative to the offset indicated by the `GAP` header.
The end beat of a note is calculated as its start beat plus its duration.
Notes SHOULD NOT overlap, i.e. the start beat of a note being between the start beat (inclusive) and end beat (exclusive) of another note.

The pitch of a note is encoded as the number of half-steps relative to `C4` (also referred to as middle C).
So a pitch of `5` represent an `F4` and a pitch of `-2` represents an `A#3`.

> [!NOTE]
>
> The pitches in this paragraph use [scientific pitch notation](https://en.wikipedia.org/wiki/Scientific_pitch_notation).

#### 4.1.1. Regular Notes `:`

A regular note is indicated by the note type `:` (colon, `%x3A`).
A regular note indicates that a certain pitch is to be held for a certain duration.
Game implementations MAY decide to compare pitches independently of the octave (i.e. compare pitches module 12).

#### 4.1.2. Golden Notes `*`

A golden note is indicated by the note type `*` (asterisk, `%x2A`).
Golden note have the same semantics as regular notes.
However, during scoring game implementations SHOULD award more points for golden notes.
The exact scoring algorithm is an implementation detail.

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
voice-number   = %x31 / %x32  ; 1 / 2
```

A voice change indicates that all notes and end-of-phrase markers following this line belong to the voice indicated by the `voice-number`.
If a body of a song contains a voice change, it must begin with a voice change.
To improve readability notes for different voices should not be interlaced.
Each voice change SHOULD only appear once in ascending order of `voice-number`.

> [!NOTE]
>
> A voice change does NOT implicitly add an end-of-phrase indicator.

> [!TIP]
>
> A song that makes use of voice changes is referred to as a “duet”.

## Appendix

### A. Relative Mode

Relative mode is a special input mode that affects parsing and interpreting songs significantly.
Relative mode is enabled by the [`#RELATIVE`](#330-the-relative-header) header being set to `yes` (case-insensitive).

#### Syntax

When relative mode is enabled, the syntax of end-of-phrase markers changes:

```abnf
end-of-phrase =/ dash
                 WSP start-beat
                 WSP rel-offset
                 *WSP line-break
rel-offset    = 1*DIGIT
```

Note that the syntax in relative mode is incompatible with the normal syntax.
Implementations MUST NOT try to rectify a missing `#RELATIVE` header based on the end-of-phrase markers encountered.

#### Semantics

In relative mode the semantics of start times changes for notes and end-of-phrase markers.

- At the start of the body a relative offset `rel` is initialized to the value of the `#GAP` header (or `0` if no `GAP` header exists).
- The start times of notes and end-of-phrase markers are relative to the current `rel` value. The absolute start time is calculated as `rel + start-beat`.
- End-of-phrase markers in relative mode include a `rel-offset`.
  After the start time of the end-of-phrase marker has been interpreted, the `rel-offset` value is added to `rel` for subsequent lines.

> [!IMPORTANT]
>
> In relative mode the order of notes and end-of-phrase markers within a file is significant.

In files with multiple voices each voice has its own `rel` value which is independent of other voices.
The `rel` value for a voice does not reset when a voice change is encountered.

## Revision History

- 2025-03-07: First revision
