# The UltraStar File Format

The UltraStar file format is a timed text format for describing karaoke songs where singing performance is scored by the karaoke software.

## Status of this document

This document aims to describe the UltraStar File Format in its most recent version 2.0.0.

> [!IMPORTANT]
>
> This document is currently a work-in-progress. Parts of the final specification may change significantly from the current state of this document.

GitHub Issues are preferred for discussion of this specification. Alternatively, you can discuss comments on our Discord server.

## 1. Introduction

The UltraStar file format provides a standardized representation of karaoke songs. The format has been used for a long time by numerous karaoke games such as [UltraStar Deluxe](https://github.com/UltraStar-Deluxe/USDX), [Performous](https://github.com/performous/performous), or [Vocaluxe](https://github.com/Vocaluxe/Vocaluxe). There exists an ecosystem of supporting applications for hosting, editing, and managing songs. However due to the lack of an official file format specification implementations differ and new features cannot be added in a consistent manner. This document aims to fix this problem by providing a formal specification for the syntax and semantics of the UltraStar file format.

The UltraStar file format is designed to be edited by machines and humans alike and is intended to be easily understood and edited by technical and non-technical users. This guiding principle is influential for many decisions made during the design process.

### 1.1. Conventions in this Document

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119](https://datatracker.ietf.org/doc/html/rfc2119).

The grammatical rules in this document are to be interpreted as described in [RFC5234](https://datatracker.ietf.org/doc/html/rfc5234). We are using the following core rules:

```abnf
CR    = %x0D     ; carriage return
LF    = %x0A     ; line feed
CRLF  = CR LF
DIGIT = %x30-39  ; 0-9
HTAB  = %x09     ; horizontal tab
SP    = %x20     ; space
```



## 2. General Structure

UltraStar files are plain text files. The UTF-8 encoding MUST be used. Implementations MUST NOT add a byte order mark to the beginning of a file. In the interests of interoperability, implementations MAY ignore the presence of a byte order mark rather than treating it as an error.

> [!CAUTION]
>
> Whether files with a BOM may be accepted is an open discussion (#46).

The canonical file extension for the UltraStar file format is `.txt`.

UltraStar files consist of a header and a body. The header contains metadata about the song and the file. The body contains the musical data. A file SHOULD end with an `E` on a single line. Everything after a trailing `E` MUST be ignored.

> [!CAUTION]
>
> Whether the trailing `E` is optional or not is open for discussion (#44).

```abnf
UltraStar-file = file-header
                 file-body
                 [ %x45 *char ]  ; E
char           = %x00-10FFFF
```

### 2.1. Line Endings and White Space

Both the header and the body of a file are defined in terms of lines. A line is a string of text that is terminated with an end-of-line sequence.

```abnf
end-of-line = ( CR / LF / CRLF )
empty-line  = end-of-line
```

Implementations SHOULD use a single Line Feed (`%x0A`) as line terminator. Implementations MUST accept the end of input (`EOF`) as a valid line terminator. Empty lines are ignored throughout the entire file.

> [!CAUTION]
>
> The allowed end-of-line sequences as well as potential recommendations for implementations are currently open for discussion (#43).

> [!CAUTION]
>
> Whether empty lines are allowed or not is currently open for discussion (#43). Whether a line that consists only of whitespace is recognized as an empty line is not yet decided.

Whitespace is used as a separator in many places of the format. Only space (`%x20`) and horizontal tab (`%x09`) are recognized as whitespace characters. In particular other unicode characters with the property `White_Space=yes` MUST NOT be treated as whitespace characters in the context of this specification.

```abnf
WSP = ( SP / HTAB )
```

> [!CAUTION]
>
> Definition and handling of whitespace characters is currently open for discussion (#46).

## 2. The File Header

The header of an UltraStar file consists of a sequence of key-value pairs. Each line in the header section starts with a hash. The key and value of a header are separated by a colon. Whitespace around key and value is ignored.

> [!CAUTION]
>
> The terminology “header” is not decided yet. Other terms used for the same concept are “tag”, “attribute” or “field”.

```abnf
file-header  = *( header / empty-line )
header       = hash *WSP header-key *WSP colon *WSP header-value *WSP line-break
header-key   = header-key-char
               [ *( HTAB / SP / header-key-char )
                 header-key-char ]
header-value = single-value / multi-value
multi-value  = single-value [ comma single-value ] [ comma ]

single-value    = *( %x20-10FFFF )
header-key-char = %x21-3A / %x3B-10FFFF  ; does not include space, tab, or colon

hash  = %x23  ; #
colon = %x3A  ; :
comma = %x2C  ; ,
```

> [!CAUTION]
>
> The use of trailing commas in comma-separated header lists has not been decided yet.

Comparisons of header keys is case-insensitive. For the sake of consistency header keys SHOULD be capitalized. Header values are generally case-sensitive unless otherwise specified. An empty value is equivalent to the header being absent.

Implementations MAY define application-specific headers but SHOULD prefix those headers with the application name to avoid conflicts with future standardized headers. Implementations MUST ignore headers they do not recognize. The order of headers is irrelevant (note the exception in section 3.1.) although standardized headers should precede any application-specific headers.

>  [!CAUTION]
>
> Handling of application-specific headers has not been decided yet.

The following sections describe the standardized headers that have been defined. Some headers are marked as deprecated or removed from a certain version onward. Implementations MUST continue to apply the defined semantics to a headers if it has been deprecated in the file format version indicated by the file. Implementations MUST NOT apply semantics to a header if a file indicates a version where the header has been removed.

### 3.1. Single-Valued and Multi-Valued Headers

Headers can be single-valued or multi-valued. Single-valued headers can only be specified once and can only contain a single value. For the sake of robustness implementations SHOULD ignore multiple occurrences of single-valued headers (the selected header value is implementation specific in this case).

Multi-valued headers can contain multiple values separated by a comma (`%x2C`). Additionally multiple occurrences of a multi-valued header are semantically equivalent to a single occurrence where all values are concatenated by a comma. In this way the order of multi-valued headers is significant.

> [!WARNING]
>
> **Backwards-Compatible Change** in version 1.1.0
>
> Multi-Valued Headers were introduced in version 1.1.0 of the format. Headers indicated as multi-valued were single-valued in previous versions.

> [!WARNING]
>
> **Backwards-Compatible Change** in version ???
>
> Concatenating multiple occurrences of multi-valued headers was introduced in version ??? of the format. Previously handling of repeated headers was not covered by this specification.

>  [!CAUTION]
>
> The use of multi-valued headers has not been decided yet (#22).

### 3.2. The `VERSION` Header

```
Required:     Yes
Multi-Valued: No
Since:        1.0.0
```

The `VERSION` header indicates the version of this specification that a file complies to. The value of the tag is a [semantic version](https://semver.org), meaning that an implementation supporting version 1.0.0 of this spec should be able to process files using a version of 1.1.0 without any changes (although new features might not be supported). Implementations SHOULD NOT attempt to process files with a higher major version than they were designed to work with.

In absence of the `VERSION` header the version 0.3.0 should be assumed. Implementations SHOULD reject a file based on the value of the `VERSION` header, in particular if the value is syntactically invalid. Applications MAY reject prerelease-versions altogether.

The `VERSION` header SHOULD be the first header in a file.

### 3.3. The `BPM` Header

```
Required:     Yes
Multi-Valued: No
Since:        0.1.0
```

The `BPM` header indicates the number of beats per minute. The notes in a song are quantized in beats. A single beat is the smallest unit of time that can be present in a song. The value is a 32 bit floating point number. The value of this tag is arbitrary in the sense that it is usually 4 to 8 times higher than the actual BPM of a song.

> [!WARNING]
>
> **Breaking Change** in version 2.0.0
>
> In versions 0.x and 1.x the value of the `BPM` header was implicitly quadrupled. This is **not** the case for version 2.0.0 and above.

> [!CAUTION]
>
> The removal of the implicit quadrupling has not been decided yet.

> [!WARNING]
>
> **Breaking Change** in version 2.0.0
>
> In versions 0.x and 1.x the comma was allowed as decimal separator in addition to the period. This feature has been removed in version 2.0.0 and above.

> [!CAUTION]
>
> The removal of the comma as a valid decimal separator has not been decided yet.

### 3.4. The `AUDIO` Header

```
Required:     Yes
Multi-Valued: No
Since:        1.1.0
```

The `AUDIO` header indicates the location of an audio file relative to the UltraStar file. This file contains the full version of a song (including instrumentals and vocals).

### 3.5. The `MP3` Header

```
Required:     No (Version 2.x), Yes (Versions 0.x and 1.x)
Multi-Valued: No
Since:        0.1.0
Deprecated:   1.1.0
Removed:      2.0.0
```

The `MP3` header indicates the location of an audio file relative to the UltraStar file. This header has been replaced by the `AUDIO` header. Implementations MUST disregard the `MP3` header if an `AUDIO` header is present (even if the specified file cannot be found or processed).

### 3.6. The `GAP` Header

```
Required:     No
Multi-Valued: No
Since:        0.2.0
```

The `GAP` header indicates an amount of time in milliseconds from the beginning of the audio track until beat 0. This effectively offsets all notes in a song by this amount of time relative to the audio track. The `GAP` value must be an integer.

> [!WARNING]
>
> **Breaking Change** in version 2.0.0
>
> In versions 0.x and 1.x the value of `GAP` could also be a floating point number. Since version 2.0.0 this is not allowed anymore.

### 3.7. The `NOTESGAP` Header

```
Required:     No
Multi-Valued: No
Since:        0.2.0
Deprecated:   0.3.0
Removed:      1.0.0
```

The `NOTESGAP` header is deprecated and MUST NOT be used. Implementations MUST ignore the field if present.

> [!NOTE]
>
> The `NOTESGAP` header was defined in version 0.1.0 But its semantics were never fully specified.

### 3.8. The `TITLE` Header

```
Required:     Yes
Multi-Valued: No
Since:        0.1.0
```

The `TITLE` header contains the title of the song.

### 3.9. The `Artist` Header

```
Required:     No
Multi-Valued: No
Since:        0.1.0
```

The `ARTIST` header contains the artist of the song.

### 3.10. The `GENRE` Header

```
Required:     No
Multi-Valued: Yes
Since:        0.2.0
```

The `GENRE` defines the genre(s) of the song. Individual genre values MUST be compared case-insensitively. For consistency it is usually best to capitalize genres.

> [!CAUTION]
>
> Whether genres should be compared case-insensitively or not has not yet been decided.

### 3.11. The `P1` and `P2` Headers

```
Required:     No
Multi-Valued: No
Since:        0.2.0
```

The headers `P1` and `P2` indicate the names of the artists that originally sang the song. `P1` is the name of the singer of first voice and `P2` is the name of the singer of the second voice. Usually `P2` is only included for duets.

### 3.12. The `DUETSINGER1` and `DUETSINGER2` Headers

```
Required:     No
Multi-Valued: No
Since:        0.2.0
Deprecated:   0.3.0
Removed:      1.0.0
```

The headers `DUETSINGER1` and `DUETSINGER2` are aliases for `P1` and `P2`. If both are specified `P1` and `P2` take precedence.

## 3. The File Body

The body of a file consists of a sequence of notes, end-of-phrase markers, and player changes.

```abnf
body = *( note /
          end-of-phrase /
          player-change /
          empty-line )
```
The sequence of notes and end-of-phrase markers SHOULD appear in ascending order by their start beats.

> [!CAUTION]
>
> Whether the body of a file must or may be sorted is not yet decided.

### 3.1. Notes

A note is a musical element in a song. Each note is defined by its type, start beat, duration, pitch, and text.

```abnf
note = note-type
       WSP start-beat
       WSP duration
       WSP pitch
       WSP note-text
       line-break

note-type  = %x21-22 / %x24-7E  ; ASCII-characters except space and #
start-beat = *DIGIT
duration   = *DIGIT
pitch      = [ minus ] *DIGIT
note-text  = 1*( %x20-10FFFF )

minus   = %x2D    ; -
```

> [!CAUTION]
>
> Whether only a single or multiple whitespaces in a row are allowed is currently up for discussion (#46).

The note type indicates how singing the correct or wrong note should affect scoring. The following sections define standard note types. Implementations MAY substitute unknown note types with freestyle notes (`F`). Implementations MUST NOT attach semantics to note types not covered by this specification.

The start beat and duration define the time when a note appears in a song. Both are indicated in beats (see section 3.3.) relative to offset indicated by the `GAP` header. The end beat of a note is calculated as its start beat plus its duration. Notes SHOULD NOT overlap, i.e. the start beat of a note being between the start beat (inclusive) and end beat (exclusive) of another note.

> [!CAUTION]
>
> Whether and how applications may define custom note types is not yet decided.

> [!CAUTION]
>
> Whether negative note starts and/or durations are valid is not yet decided.

The pitch of a note is encoded as the number of half-steps relative to middle C or C4. So a pitch of `5` represent an F4 and a pitch of `-2` represents an A#3.

#### 3.1.1. Regular Notes `:`

A regular note is indicated by the note type `:` (colon, `%x3A`). A regular note indicates that a certail pitch is to be held for a certain duration. Game implementations MAY decide to compare pitches independently of the octave (i.e. compare pitches module 12).

#### 3.1.2. Golden Notes `*`

A golden note is indicated by the note type `*` (asterist, `%x2A`). Golden note have the same semantics as regular notes. However, during scoring game implementations SHOULD award more points for golden notes. The exact scoring behavior is an implementation detail.

#### 3.1.3. Rap Notes `R`

> [!WARNING]
>
> Rap notes are standardized in version 0.2.0 of this specification.

A rap note is indicated by the note type `R` (the letter R, `%x52`). Rap notes have the same timing semantics as regular notes but are intended or spoken phrases that do not have a defined pitch. Implementations MUST ignore pitch information on rap notes.

#### 3.1.4. Golden Rap Notes `G`

>  [!WARNING]
>
> Golden rap notes are standardized in version 0.2.0 of this specification.

A golden rap note is indicated by the note type `G` (the letter G, `%x47`). Golden rap notes have the same semantics as rap notes. However, during scoring game implementations SHOULD award more points for golden rap notes. The exact scoring behavior is an implementation detail.

#### 3.1.5. Freestyle Notes

>  [!WARNING]
>
> Freestyle notes are standardized in version 0.2.0 of this specification.

A freestyle note is indicated by the note type `F` (the letter F, `%x46`). Similar to rap notes, freestyle notes do not carry pitch information. Additionally game implementations MUST NOT award points for freestyle notes.

### 3.2. End-of-Phrase Markers

End-of-Phrase markers are indicated by a `-` character (hyphen/minus, `%x2D`).

```abnf
end-of-phrase = dash
                WSP start-beat
                *WSP line-break
```

An end-of-phrase marker carries no musical information but indicates the end of a phrase in the song. This is usually interpreted as a line break in the lyrics.

An end-of-phrase SHOULD NOT appear between the start beat (inclusive) of a note and its end beat (exclusive). An end-of-phrase marker SHOULD NOT appear before the start time of the first note or after the start time of the last note.

> [!CAUTION]
>
> Whether leading or trailing end-of-phrase markers are allowed is currently up for discussion (#44).

> [!CAUTION]
>
> Whether there can be non-whitespace text following an end-of-phrase indicator is not yet decided.

### 3.3. Player Changes

A player change is indicated by a `P` (the letter P, `%x50`), immediately followed by a number.

```abnf
player-change = p player-numer
                *WSP line-break
p             = %x50    ; P
player-number = "1" / "2"
```

A player change indicates that all notes and end-of-phrase markers following this line belong to the player, indicated by the `player-number`. This allows inclusion of up to 2 voices. If the body of a song does not start with a player change, `P1` is assumed implicitly. Implementations SHOULD NOT include the same player change more than once (i.e. notes for different players should not be interlaced).

> [!NOTE]
>
> A player change does NOT implicitly add an end-of-phrase indicator.

> [!TIP]
>
> An UltraStar file that makes use of player changes is referred to as a “duet”.

> [!WARNING]
>
> **Breaking Change** in version 0.2.0
>
> In version 0.1.0 only single-player songs were defined. Player changes are specified since version 0.2.0.

> [!WARNING]
>
> There exists a legacy behavior where an indicated `P3` would start a sequence of notes that apply to both players. This behavior is explicitly NOT compliant with this specification.

> [!CAUTION]
>
> Whether songs that make use of player changes need to start their body with a player change is not yet decided.

> [!CAUTION]
>
> Whether single-voice songs can have player changes (only `P1`) is not yet decided.

> [!CAUTION]
>
> Whether there may be a whitespace between the “P” and the indicated player is not yet decided.

## Appendix

### A. Relative mode

How does player change work in relative mode?
