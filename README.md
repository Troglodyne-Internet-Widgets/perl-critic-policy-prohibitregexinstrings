# NAME

Perl::Critic::Policy::RegularExpressions::ProhibitRegexInStrings - Write a regular expression as qr//, not as a string that somebody compiles later.

# VERSION

version 0.001

# Perl::Critic::Policy::RegularExpressions::ProhibitRegexInStrings

A regular expression held in a string is compiled somewhere else, later, under
the rules of that place and not of the file that wrote it.  Two things go
wrong with it, and nothing reports either:

- The modifiers of the file do not reach it.  Under `use re '/aa'`,
`[[:alnum:]]`, `\d` and `\w` match only ASCII in the code of the file.  In a
string that JSON::Validator compiles as the `pattern` of a schema, they match
any letter or digit of Unicode.
- The policies of Perl::Critic do not read it.  They read `m//`,
`qr//`, `s///` and `split`, and a string is a string to them.  So
`'^[A-Za-z0-9]+$'` passes `ProhibitEnumeratedClasses` and
`RequireExtendedFormatting`, and its `$`, which also matches before a
newline at the end, passes everything:

    ```perl
    pattern => '^[a-z_][a-z0-9_-]*$'    # reported: accepts "bob\n"
    ```

Write the expression as `qr//`, where the file and the policies can see it.
Where a string is what the other end takes, give it the `qr//` in a string.
A compiled expression keeps its modifiers when it becomes one, as
`(?^aa:...)`:

```perl
my $USER = qr/\A [[:lower:]_] [[:lower:][:digit:]_-]* \z/x;
...
pattern => "$USER"
```

This policy reports two kinds of string.

## PROHIBITED

The value of a key that names a regular expression, `pattern`, `regex` or
`regexp` by default, in a list of pairs:

```perl
{ type => 'string', pattern => '^[a-z]+$' }
validate( value => $v, regex => q{\A\d+\z} )
{ 'regexp' => "^$prefix-" }
```

A lexical scalar declared with a string, and used as a regular expression in
its scope: interpolated into a match, a substitution or a `qr//`, bound with
`=~` or `!~`, or given to `split`:

```perl
my $sep = '\s*,\s*';
my @parts = split $sep, $line;

my $name = '[A-Za-z]+';
return unless $value =~ m/\A$name\z/;
```

## ALLOWED

```perl
pattern => "$USER"                  # a qr// in a string
pattern => "${USER}"
pattern => qr/\A[a-z]+\z/
name    => '^not a pattern$'        # a key that names no expression
my $sep = qr/\s*,\s*/;              # a qr//
my $text = 'abc';                   # a string that no match uses
$value =~ m/\Q$literal\E/;          # see CAVEATS
```

## CONFIGURATION

`keys` is the list of the keys whose value is a regular expression, separated
by spaces.  It replaces the default, `pattern regex regexp`:

```
[RegularExpressions::ProhibitRegexInStrings]
keys = pattern regex regexp match_re
```

## CAVEATS

The second kind is found by its name, in the block that declares it and after
the declaration.  So a scalar that the block declares again with another value
can be reported for a match that uses the second one, and a package variable
that another file matches with is not seen.

A string that a match quotes with `\Q` is a literal, and not a pattern, but
the policy does not look inside `\Q`.  Say `## no critic
(ProhibitRegexInStrings)` there, or use `index`.

A key is matched by its name, whatever the hash is for.  A `pattern` key that
holds a glob or a `sprintf` format is reported too.  Take the key out of
`keys` for such a code base.

## METHODS

### supported\_parameters

### default\_severity

### default\_themes

### applies\_to

### violates

# BUGS

Please report any bugs or feature requests on the bugtracker website
[https://github.com/Troglodyne-Internet-Widgets/perl-critic-policy-prohibitregexinstrings/issues](https://github.com/Troglodyne-Internet-Widgets/perl-critic-policy-prohibitregexinstrings/issues)

When submitting a bug or request, please include a test-file or a
patch to an existing test-file that illustrates the bug or desired
feature.

# AUTHORS

Current Maintainers:

- George S. Baugh <george@troglodyne.net>

# CONTRIBUTOR

troglodyne-robot <trogbot@troglodyne.net>

# COPYRIGHT AND LICENSE

Copyright (c) 2026 Troglodyne LLC

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:
The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
