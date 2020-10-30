# Types and encodings

## Python 2

str: byte string
  - Literal: "this" (or b"that" for 2/3 compat)
  - "ajá" is `"aj\xc3\xa1"` (4 bytes)
  - Has `.decode(codec)` returning `unicode`
unicode: unicode code points string
  - Literal: u"this"
  - "ajá" is `u"aj\xe1"` (3 code points)
  - Has `.encode(codec)` returning `str`

## Python 3

str: unicode code points string
  - Literal: "this" (or u"that" for 2/3 compat)
  - "ajá" is `"aj\xe1"` (3 code points)
  - Has `.encode(codec)` returning `bytes`
bytes: byte string
  - Literal: b"this"
  - "ajá" is `b"aj\xc3\xa1"` (4 bytes)
  - Has `.decode(codec)` returning `str`

# Implicit cohercion and default encoding

## Python 2

When operations expect `unicode`, `str`s are promoted using the default codec, hardcoded to `ascii`.

```python
"a"                 + u"b"
unicode("a")        + u"b"
"a".decode("ascii") + u"b"  # Since `ascii` is the default encoding
u"ab"

                "a".encode("utf-8")  # Note: `str`s should not respond to `encode`!
       unicode("a").encode("utf-8")
"a".decode("ascii").encode("utf-8")  # Since `ascii` is the default encoding
"a"
```

# Python 3

No implicit cohercion is done.



