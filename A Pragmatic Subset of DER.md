# A Pragmatic Subset of DER

## Introduction

Cryptographic protocols and message formats such as X.509 certificates
and CRLs use the Distinguished Encoding Rules, or DER, to encode and
decode messages. While elements can be encoded multiple ways with BER,
DER defines a single way to encode an element. By mandating
that senders use a single, deterministic encoding for a given element,
receivers do not need to normalize messages prior to verifying
signature. 

Many cryptographic protocols utilize only a subset of the full
functionality presented by DER. Many of the features have never been
exercised, so it is not useful to describe them to implementers who
are seeking solely to interoperate with these protocols and are not
interested in developing a full DER implementation. This primer on
DER is focused on such an implementer.

## Tag-Length-Value

Elements in DER all contain three fields: the tag, the length, and the
value. This encoding format of specifying type information followed by
the length of the value and the value itself is commonly referred to
as Tag-Length-Value, or TLV, for short. In this primer, the 3-tuple
of tag, length, and value is referred to as "TLV".

### Tag

Tags are always one octet in length. Within the tag, there are three
fields: the class, the format, and the number. 

The class occupies the two most significant bits, the format is the
next most significant bit, and the number occupies the bottom 5 bits.
The value of each of these fields is represented in big-endian format.

#### Class

The class is used to inform the meaning of the tag number.

There are two classes that are used in cryptographic protocols:
universal, which is represented by a "0" value, and context-specific,
which is represented by a "2" value.

The universal class indicates that the tag number corresponds to a
number that is allocated by X.680 itself. For example, the ASN.1 type
INTEGER has a tag number of 2. When the class is universal/0 and the tag
number is 2, then the tag indicates that the element is an INTEGER.

The context-specific class indicates that the tag number corresponds to
a number defined within the ASN.1 schema for the TLV. Since a
context-specific tag needs a context to determine the meaning of the
tag number, context-specific tags only appear within other ASN.1
TLVs that have a constructed format.

There are two sub-categories of context-specific tags: IMPLICIT and
EXPLICIT. IMPLICIT tags are used to override the universal tag number
of a given TLV. This is useful in certain ASN.1 types which require
a unique tag number for each field (see the section on CHOICE for
more details).

Conversely, EXPLICIT tags create a wrapper to contain a
single TLV. The tag number of the EXPLICITly tagged TLV is
specified in the ASN.1 schema, and the "inner" TLV retains the
tag that it would have normally.

#### Format

There are two formats: primitive, which is represented by a "0" value,
and constructed, which is represented by a "1" value.

Primitive values are essentially scalar values: the value of the element
does not contain other ASN.1 elements, absent some other
decoding/encoding rule outside of the DER specification. Conversely,
constructed values contain one or more ASN.1 values.

If the class of the tag is universal, then the format value is always
set to the format as defined by the type indicated by the tag number.
If the class of the tag is context-specific, then there are two
possibilities, depending on whether the context-specific tag is IMPLICIT
or EXPLICIT. If IMPLICIT, then the format is always the same for the
corresponding universal type under DER rules. For example, the BIT
STRING type always uses the primitive format under DER rules. Thus, an
IMPLICITly tagged BIT STRING will always indicate the primitive format.

The situation is easier for EXPLICITly tagged elements: they are always
constructed, since they always contain a TLV within.

#### Number

For tags with a universal class, the number indicates the type of data
as defined in X.680.

For tags with a context-specific class, the number indicates the type
of data as defined within the ASN.1 schema. It is common practice for
a given constructed element that contains context-specific fields to
start numbering at 0, but this is not required.

# XXX: add example of IMPLICT and EXPLICIT tagging

### Length

There are two formats used for encoding the length. The format is
determined by the length of the value. For values that are less than 128
octets, the length is encoded as a big-endian integer in a single octet.
For values that are 128 octets or larger, the first length octet is
has the most significant bit asserted and the lower 7 bits encode the
number of octets used to encode the length of the value as a big-endian
integer. The length of the value then follows this initial length octet.

### Value

The value field contains the actual ASN.1 value. As mentioned above,
the type of the value is determined by the class and tag number.

## The Primitive Types

The following sections describe the primitive types that are commonly
used in cryptographic protocols. Each of these types are in the
universal class, and they are primitive format. As a result, they
differ by only the tag number and the actual value.

### BOOLEAN

__Tag number:__ 1

BOOLEAN values have only two possible values: true and false. The
length is always one octet, and "false" is always represented by a
single octet "0x00". "True" is always represented by a single octet
"0xFF".

This is the encoding of a "true" BOOLEAN:

```01 01 FF``

This is the encoding of a "true" BOOLEAN:

```01 01 00``

### INTEGER

__Tag number:__ 2

INTEGER values are signed, 2-complement, big-endian integers. There is
one quirk in the encoding of the value: if the integer is positive and
the most significant bit of its leading octet is asserted, then a
leading "0x00" is encoded before encoding the integer. This is done so
that there is no confusion between a positive value that so happens to
have the most significant bit of its leading octet asserted and a
negative value (which will always have this bit asserted).

This is the encoding of the integer 1:

```02 01 01```

This is the encoding of the integer -1:

```02 01 FF```

This is the encoding of the integer 255:

```02 02 00 FF```


