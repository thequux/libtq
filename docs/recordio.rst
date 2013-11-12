=============
RecordIO v1.0
=============

Features
========

- Streaming output of records
- Multiple record types
- If records are ASCII, the file is human-readable

Structure
=========

recordio ::= header record*
header   ::= magic `\n` kvpair* `\n`
magic    ::= `RecordIO ` version
version  ::= `v` num `.` num
num      ::= `0` | [1-9][0-9]*    # Number must have decimal value <=2^32-1
kvpair   ::= key `: ` value `\n`
key      ::= [A-Z][a-z*](-[A-Z][a-z]*)*
value    ::= [^\n]*
record   ::= record-type `:` length rhterm@(`:`|`+`) length*byte `\n`
record-type ::= [.]?[A-Za-z0-9]+
length   ::= num

Description
===========

This specification is sufficient to extract all user data from any
v1.x file.  Future v1.x versions of this specification MAY add new
features, but they will remain completely compatible with v1.0
parsers.  Files written according to this spec MUST be written with
a version of v1.0. 

[Where the above BNF and the following two paragraphs disagree, the
BNF is authoritative]

A RecordIO file has two parts: a header, and then a collection of
contiguous segments. The header consits of a magic string
("`RecordIO`" followed by a version number) followed by a sequence of
key/value pairs, along the lines of HTTP of RFC822, but simpler. Each
key is a dash-separated sequence of capitalized words with no
whitespace. Keys are followed by a colon, at least one space, and a
value that extends until the end of the line (modulo any whitespace at
the beginning or end).

Following the header is a sequence of contiguous record segments, each
consisting of a segment header, a segment body, and a terminating
newline.  The segment header consists of a record type (a sequence of
alphanumeric characters), a colon, an ASCII length, and either a colon
or a plus.  The segment body is simply the number of bytes contained
in the segment header. 

All keys and values in the header must be ASCII; future versions of
this spec may loosen this restriction. No segment may be more than
2^32-1 (2147483647) bytes long. Values do not include any leading or
trailing whitespace.

Segments may be either partial or terminating; a partial segment's
header is terminated by a `+`, whereas a termanting segment's header
is terminated by a `:`.  Each partial segment MUST be followed by
another segment of the same type.  The sequence of partial segments
and final terminating segment SHOULD be reassembled into a contiguous
record before being passed to the higher level application, unless the
application specifically requests the receipt of partial segments.
The division of a record into partial segments is simply an artifact
of the encoding into RecordIO format, and holds no semantic
significance.

Records whose types begin with a `.` are reserved for library-internal
use and MUST NOT be written with application-provided contents or
passed to the application when encountered while reading.

Headers
=======

While all of these headers are optional, it is highly recommended to
include at least Application.  Multiple headers with the same name MAY
be provided; all provided values SHOULD be collected and passed to the
application.  Unkown headers MUST be preserved for the application but
not be interpreted by the library in any way.  Headers not listed in
any version of this spec SHOULD have names starting with `X-` followed
by some identifier of the organization that created the header.  For
example, Mozilla-specific headers may have names starting with
`X-Moz-`, while UpstandingHackers headers will start with `X-Uh-`.

Date::
  An ISO 8601 formatted representation of the time this file was written.
Application::
  The name and version of the application which created this RecordIO file.
Content-Type::
  A MIME type for the file as a whole.
Record-Content-Type::
  `<record-type>: <mime-type-of-record>`

Example
=======

RecordIO v1.0
Date: 2013-11-11T23:50-06:00
Description: Example RecordIO file
Record

Continued:31+These two records have the same
Continued:9: content.
Single:40:These two records have the same content.
