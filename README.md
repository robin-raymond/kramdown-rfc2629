# kramdown-rfc2629

[kramdown][] is a [markdown][] parser by Thomas Leitner, which has a
number of backends for generating HTML, Latex, and markdown again.

**kramdown-rfc2629** is an additional backend to that: It allows the
generation of [XML2RFC][] XML markup (also known as [RFC 2629][]
compliant markup).

Who would care?  Anybody who is writing Internet-Drafts and RFCs in
the [IETF][] and prefers (or has co-authors who prefer) to do part of
their work in markdown.

# Usage

Start by installing the kramdown-rfc2629 gem (this automatically
installs kramdown version 1.0.x as well):

    sudo gem install kramdown-rfc2629

The guts of kramdown-rfc2629 are in one Ruby file,
`lib/kramdown-rfc2629.rb` --- this melds nicely into the extension
structure provided by kramdown.  `bin/kramdown-rfc2629` started out as
a simple command-line program showing how to use this, but can now do
much more (see below).

To use kramdown-rfc2629, you'll need a Ruby 1.9 or 2.0, the command
"wget" (if you want to use the offline feature), and maybe [XML2RFC][]
if you want to see the fruits of your work.

    kramdown-rfc2629 mydraft.mkd >mydraft.xml
    xml2rfc mydraft.xml

# Examples

`stupid.mkd` was an early markdown version of an actual Internet-Draft
(for a protocol called [STuPiD][] \[sic!]).  This demonstrated some,
but not all features of kramdown-rfc2629.  Since markdown/kramdown
does not cater for all the structure of an RFC 2629 style document,
some of the markup is in XML, and the example switches between XML and
markdown using kramdown's `{::nomarkdown}` and `{:/nomarkdown}` (this
is ugly, but works well enough).  `stupid.xml` and `stupid.txt` show
what kramdown-rfc2629 and xml2rfc make out of this.

`stupid-s.mkd` is the same document in the new sectionized format
supported by kramdown-rfc2629.  The document metadata are in a short
piece of YAML at the start, and from there, `abstract`, `middle`,
references (`normative` and `informative`) and `back` are sections
delimited in the markdown file.  See the example for how this works.
The sections `normative` and `informative` can be populated right from
the metadata, so there is never a need to write XML any more.
Much less scary, and no `{:/nomarkdown}` etc. is needed any more.
Similarly, `stupid-s.xml` and `stupid-s.txt` show what
kramdown-rfc2629 and xml2rfc make out of this.

`draft-ietf-core-block-xx.mkd` is a real-world example of a current
Internet-Draft done this way.  For RFC and Internet-Draft references,
it uses document prolog entities instead of caching the references in
the XML (i.e., not standalone mode, this is easier to handle when
collaborating with XML-only co-authors).  See the `bibxml` metadata.

# The YAML header

Please consult the examples for the structure of the YAML header, this should be mostly
obvious.  The `standalone` attribute controls whether the RFC/I-D
references are inserted into the document (yes) or entity-referenced
(no), the latter leads to increased build time, but may be more
palatable for a final XML conversion.
The author entry can be a single hash or a list, as in:

    author:
      ins: C. Bormann
      name: Carsten Bormann
      org: Universität Bremen TZI
      abbrev: TZI
      street: Bibliothekstr. 1
      city: Bremen
      code: D-28359
      country: Germany
      phone: +49-421-218-63921
      email: cabo@tzi.org

or

    author:
      -
        ins: C. Bormann
        name: Carsten Bormann
        org: Universität Bremen TZI
        email: cabo@tzi.org
      -
        ins: Z. Shelby
        name: Zach Shelby
        org: Sensinode
        role: editor
        street: Kidekuja 2
        city: Vuokatti
        code: 88600
        country: Finland
        phone: "+358407796297"
        email: zach@sensinode.com

(the hash keys are the XML GIs from RFC 2629, with a flattened
structure.  As RFC 2629 requires giving both the name and
surname/initials, we use `ins` as an abbreviation for
"initials/surname".  Yes, the toolchain is Unicode-capable, even if
the final RFC output is still in ASCII.)


## References

The references section is built from the references listed in the YAML
header and from references made inline to RFCs and I-Ds in the
markdown text.  Since kramdown-rfc2629 cannot know whether a reference
is normative or informative, no entry is generated by default in the
references section.  By indicating a normative reference as in
`{{!RFC2119}}` or an informative one as in `{{?RFC1925}}`, you can
completely automate the referencing, without the need to write
anything in the header.  Alternatively, you can write something like:

    informative:
      RFC1925:
    normative:
      RFC2119:

and then just write `{{RFC2119}}` or `{{RFC1925}}`.  (Yes, there is a
colon in the YAML, because this is a hash that could provide other
information.)

If your references are not in the [XML2RFC][] databases, you need to
spell it out like this:

    informative:
      RFC1925:
      WEI:
        title: "6LoWPAN: the Wireless Embedded Internet"
        author:
          -
            ins: Z. Shelby
            name: Zach Shelby
          -
            ins: C. Bormann
            name: Carsten Bormann
        date: 2009
        seriesinfo:
          ISBN: 9780470747995
        ann: This is a really good reference on 6LoWPAN.
      ASN.1:
        title: >
          Information Technology — ASN.1 encoding rules:
          Specification of Basic Encoding Rules (BER), Canonical Encoding
          Rules (CER) and Distinguished Encoding Rules (DER)
        author:
          org: International Telecommunications Union
        date: 1994
        seriesinfo:
          ITU-T: Recommendation X.690
      REST:
        target: http://www.ics.uci.edu/~fielding/pubs/dissertation/fielding_dissertation.pdf
        title: Architectural Styles and the Design of Network-based Software Architectures
        author:
          ins: R. Fielding
          name: Roy Thomas Fielding
          org: University of California, Irvine
        date: 2000
        seriesinfo:
          "Ph.D.": "Dissertation, University of California, Irvine"
        format:
          PDF: http://www.ics.uci.edu/~fielding/pubs/dissertation/fielding_dissertation.pdf
    normative:
      ECMA262:
        author:
          org: European Computer Manufacturers Association
        title: ECMAScript Language Specification 5.1 Edition
        date: 2011-06
        target: http://www.ecma-international.org/publications/files/ecma-st/ECMA-262.pdf
        seriesinfo:
          ECMA: Standard ECMA-262
      RFC2119:
      RFC6690:

(as in the author list, `ins` is an abbreviation for
"initials/surname".) Then you can simply reference `{{ASN.1}}` and
`{{ECMA262}}` in the text.  (Make sure the reference keys are valid XML
names, though.)

# Risks and Side-Effects

The code is not very polished, but it has been successfully used for a
number of non-trivial Internet-Drafts.  You probably still need to
skim [RFC 2629][] if you want to write an Internet-Draft, but you
don't really need to understand XML very much.  Knowing the basics of
YAML helps with the metadata (but you'll understand it from the
examples).

# Related Work

Moving from XML to Markdown for RFC writing apparently is a
no-brainer, so I'm not the only one who has written code for this.

[Miek Gieben][] has done a [similar thing][pandoc2rfc] employing
pandoc/asciidoc.  He uses multiple input files instead of
kramdown-rfc2629's sectionized input format.  He keeps the metadata in
a separate XML file, similar to the way the previous version of
kramdown-rfc2629 stored (and still can store) the metadata in XML in
the markdown document.  He also uses a slightly different referencing
syntax, which is closer to what markdown does elsewhere but more
verbose.

# License

With kramdown version 1.0, kramdown itself now is MIT licensed, so it
is finally possible to license kramdown-rfc2629 under the same
license.

[kramdown]: http://kramdown.rubyforge.org/
[stupid]: http://tools.ietf.org/id/draft-hartke-xmpp-stupid-00
[RFC 2629]: http://xml.resource.org/public/rfc/html/rfc2629.html
[markdown]: http://en.wikipedia.org/wiki/Markdown
[IETF]: http://www.ietf.org
[Miek Gieben]: http://www.miek.nl/
[pandoc2rfc]: https://github.com/miekg/pandoc2rfc/
[XML2RFC]: http://xml.resource.org
