# Difference between language and script

## Top exp

A language refers to a method of spoken communication. It has its own vocabulary, grammar, etc.

A script refers to a method of written communication. It has its own collection of symbols or characters. These symbols
represent a concept, a sound or a word of a language. They are combined together to transcribe the language.

English, French, German, Italian are different languages. They are all written using the Roman Script.

Similarly, Sanskrit, Hindu, Marathi, Nepali are different languages. They are written using the Devanagari Script.

Most languages are commonly written using a single script, but not always. E.g. Konkani is a language which is written
in 5–6 different scripts (including Devanagari and Roman).

If you can understand Hindi language and read Devanagari script, these examples will make the difference between
language and script clear:

- This is English language written in Roman Script.
- Ye Roman script mein likhi hui Hindi Bhasha hai.
- दिस इ़ज़ इन्गलिश लैन्गुएज रिट्टन इन देवनागरी स्क्रिप्ट।
- ये देवनागरी स्क्रिप्ट में लिखी हुई हिन्दी भाषा है।

## Another exp

Language is a medium of communication between human beings. We are required to learn the particular language in spoken &
or in written form to convey our views & understand other’s.How ever one spoken language can be written in different
scripts & one script can be used for many languages.

Let me elaborate above statement with two examples.Punjabi is a language which is spoken both in Indian as well as
Pakistani Punjab but it is written in “ Shahmukhi” script in Pakistan where as “Gurmukhi” or some times “Devnagri” is
used in India for Punjabi language.We Indian Punjabi can therefore understand Punjabi language spoken by Pakistani but
can not do if written in “Shahmukhi” since we are not conversant with this script .Similarly Pakistani can not read
“Gurmukhi”& hence unable to follow our written Punjabi but understand our spoken Punjabi.

Again in “Devnagri”script,Hindi,Sanskrit,Marathi & Nepali languages are written.Now most of the north Indians know
“Devnagri”script,but they can not understand other than Hindi language.Knowledge of other three languages is required to
grasp them although they know their script. Hence language & script are not synonyms.

Of course some how or the other we can write all languages in one script & all known scripts can be used to write one
language with some inconvenience.Hope I am able to clarify the difference.

## Most likely

**Language is what you speak, Script is what you write, Period**

Now see the following

- I am a boy
- आई एम अ बॉय
- Main Ladkaa Hoon
- मैं लड़का हूँ

The above 4 sentences are, respectively

- English Language in Roman Script
- English Language in Devanagari Script
- Hindi Language in Roman Script
- Hindi Language in Devanagari Script

A native English speaker with no knowledge of Indian Languages/Script, will be able to read statement 1 and 3, but not 2
and 4. The speaker will also be able to understand statement 1 and 2, but not 3 and 4, if you read them out to her/him

ref:

- [Quora](https://www.quora.com/What-is-the-difference-between-language-and-script-not-in-terms-of-programming-language-in-history)

# Text processing with OpenType Layout fonts

A text-processing client follows a standard process to convert the string of characters entered by a user into
positioned glyphs. To produce text with OpenType Layout fonts:

1. Using the 'cmap' table in the font, the client converts the character codes into a string of glyph indices.
2. Using information in the GSUB table, the client modifies the resulting string, substituting positional or vertical
   glyphs, ligatures, or other alternatives as appropriate.
3. Using positioning information in the GPOS table and baseline offset information in the BASE table, the client then
   positions the glyphs.
4. Using design coordinates the client determines device-independent line breaks. Design coordinates are high-resolution
   and device-independent.
5. Using information in the JSTF table, the client justifies the lines, if the user has specified such alignment.
6. The operating system rasterizes the line of glyphs and renders the glyphs in device coordinates that correspond to
   the resolution of the output device.

Throughout this process the text-processing client keeps track of the association between the character codes for the
original text and the glyph indices of the final, rendered text. In addition, the client may save language and script
information within the text stream to clearly associate character codes with typographical behavior.

# Left-to-right and right-to-left text

When an OpenType text layout engine applies the Unicode bidi algorithm and gets to the point where mirroring needs to be
performed on runs with an even, i.e. left-to-right (LTR), resolved level, it does the following:

1. Glyph-level mirroring:

   Apply feature 'ltrm' to the entire LTR run to substitute mirrored forms.

2. LTR glyph alternates:

   Apply feature 'ltra' to the entire LTR run to finesse glyph selection.

For runs with an odd, i.e. right-to-left (RTL), resolved level, the engine does the following:

1. Character-level mirroring:

   For each character i in the RTL run:

   If it is mapped to character j by the OMPL and cmap(j) is non-zero:

   Use glyph cmap(j) at character i<br><br>

   Here OMPL refers to the OpenType Mirroring Pairs List, and cmap(j) refers to the glyph at code point j in the
   Unicode 'cmap' subtable.<br><br>

   For example, suppose U+0028, LEFT PARENTHESIS, occurred in the run at resolved level 1. The glyph at that code point
   in the run will be replaced by cmap(U+0029), since {U+0028, U+0029} is a pair in the OMPL.<br><br>

2. Glyph-level mirroring:

   The engine applies the 'rtlm' feature to the entire RTL run. The feature, if present, substitutes mirrored forms for
   characters other than those covered by the first elements of OMPL pairs (otherwise, it could cancel the effects of
   character-level mirroring).<br><br>

   The data contents of the OMPL are identical to the Bidi Mirroring Glyph Property file of Unicode 5.1, and will never
   be revised. Thus, it will be up to the 'rtlm' feature to provide, if needed, mirrored forms for both (a) Unicode 5.1
   code points with the “mirrored” property but no appropriate Unicode 5.1 character mirrors, as well as (b) all future
   “mirrored” property additions to Unicode, whether or not character mirrors exist for them.<br><br>

   With such a division of labor between the layout engine and the font, most fonts will not need to include an 'rtlm'
   feature, since the mirrored forms in their Unicode 'cmap' subtable would be adequate.<br><br>

3. RTL glyph alternates:

   The engine applies the 'rtla' feature to the entire RTL run. The feature, if present, substitutes variants
   appropriate for right-to-left text (other than mirrored forms).

In practice, the engine may apply features simultaneously; thus, it is up to the font vendor to ensure that the
features’ lookups are ordered to achieve the desired effect of the algorithms described above. The engine may optimize
its implementation in various ways, e.g. by taking advantage of the fact that character- and glyph-level mirroring won’t
both apply on the same element in the run.

# bidi

**U+202E**: Unicode right to left override (RTLO).

ref:

- [open type layout overview](https://docs.microsoft.com/en-us/typography/opentype/spec/ttochap1)

# Unicode

CJK Compatibility Ideographs is a Unicode block containing Han Ideographs that contained duplicate characters in the
South Korean KS X 1001:1998 (U+F900-U+FA0B), Taiwanese Big5 (U+FA0C-U+FA0D), Japanese IBM 32 (CP932 variant;
U+FA0E-U+FA2D), JIS X 0213 (U+FA30-U+FA6A), ARIB STD-B24 (U+FA6B-U+FA6D) and the North Korean KPS 10721-2000 (
U+FA70-U+FAD9) source standards for CJK characters. In order to retain round-trip compatibility with that standard, the
CJK Compatibility Ideographs block was created to hold those extra characters. In subsequent versions of the standard,
more compatibility ideographs, and even a few regular ideographs that do not have duplicates, have been added to the
block.

ref:

- [Unicode Utilities: Character Properties](https://util.unicode.org/UnicodeJsps/character.jsp?a=20011&B1=Show)
- [Unicode Utilities: Character Property Index](https://util.unicode.org/UnicodeJsps/properties.jsp?a=RGI_Emoji#RGI_Emoji)
- [CJK Unified Ideographs (Unicode block)
  ](https://en.wikipedia.org/wiki/CJK_Unified_Ideographs_(Unicode_block))

The zero-width non-joiner (ZWNJ) is a non-printing character used in the computerization of writing systems that make
use of ligatures. When placed between two characters that would otherwise be connected into a ligature, a ZWNJ causes
them to be printed in their final and initial forms, respectively. This is also an effect of a space character, but a
ZWNJ is used when it is desirable to keep the words closer together or to connect a word with its morpheme.

ref:

- [Regular and Unusual Space Characters](https://unicode-explorer.com/articles/space-characters)

# Emoji

Each emoticon has two variants:

- U+FE0E (VARIATION SELECTOR-15) selects text presentation (e.g. ☃︎),
- U+FE0F (VARIATION SELECTOR-16) selects emoji-style (e.g. ☃️).

If there is no variation selector appended, the default is the emoji-style.

ref:

- [emoji variation](https://nanomichael.github.io/MicroTeX/?tex=\text{\char%222603\char%22fe0f\char%222603\char%22fe0e})

The Miscellaneous Symbols and Pictographs block has 54 emoji that represent people or body parts. A set of "Emoji
modifiers" are defined for emojis that represent people or body parts. These are modifier characters intended to define
the skin colour to be used for the emoji. The draft document suggesting the introduction of this system for the
representation of "human diversity" was submitted in 2015 by Mark Davis of Google and Peter Edberg of Apple Inc.[8] Five
symbol modifier characters were added with Unicode 8.0 to provide a range of skin tones for human emoji. These modifiers
are called EMOJI MODIFIER FITZPATRICK TYPE-1-2, -3, -4, -5, and -6 (U+1F3FB–U+1F3FF): 🏻 🏼 🏽 🏾 🏿. They are based on the
Fitzpatrick scale for classifying human skin color.

ref:

- [emoji modifier](https://nanomichael.github.io/MicroTeX/?tex=\text{\char%221f474\char%221f474\char%221f3fb\char%221f474\char%221f3fc\char%221f474\char%221f3fd\char%221f474\char%221f3ff})
- [table of emoji modifiers](https://en.wikipedia.org/wiki/Miscellaneous_Symbols_and_Pictographs#Emoji_modifiers)

Zero Width Joiner (ZWJ) is a Unicode character that joins two or more other characters together in sequence to create a
new emoji.

Zero Width Joiner, pronounced "zwidge", is not an emoji and has no appearance by itself. This is an invisible character
when used alone.

View [Emoji ZWJ Sequences](https://emojipedia.org/emoji-zwj-sequence/) that use this character.

ref:

- [Emoji ZWJ Sequences: Three Letters, Many Possibilities](https://blog.emojipedia.org/emoji-zwj-sequences-three-letters-many-possibilities/)
- [Emoji ZWJ](https://nanomichael.github.io/MicroTeX/?tex=\text{\char%221f468\char%22200d\char%221f469\char%22200d\char%221f467%20\char%221f468\char%22200d\char%221f466})

# Combining Diacritical Marks

For example: < ́> with a: á

ref:

- [Combining Diacritical Marks](https://en.wikipedia.org/wiki/Combining_Diacritical_Marks)

# decomposition

ref:

- [Unicode Normalization](https://unicode.org/reports/tr15/)
- [Normalization](https://unicode.org/faq/normalization.html)

# Bopomofo

ref:

- [Bopomofo (Unicode block)](https://en.wikipedia.org/wiki/Bopomofo_(Unicode_block))
