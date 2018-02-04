- Feature Name: Unicode Identifiers
- Start Date: 2017-11-26
- RFC PR: Leave this empty, filled on proposal accept
- Haskell Report Issue: Leave this empty, filled on proposal accept



#######
Summary
#######

The lexical syntax of Haskell'98 has been designed primarily for the ASCII character set. The language report does
acknowledge Unicode as an afterthought, but there are large gaps in its support. This proposal aims to rectify this
while retaining backward compatibility with all existing ASCII-only Haskell code.

##########
Motivation
##########

There are several problems with the way Haskell grapples with Unicode:
* The language fundamentally insists on classifying all identifiers as either capital or lowercase identifiers at the
  lexical layer, but many Unicode scripts don't even recognize the concept. No word of Arabic, Chinese, and most other
  languages that use non-European scripts cannot be Haskell identifiers as a consequence.
* Another problem with the lexical identifier syntax is that it admits no accents and other modifier characters. This
  excludes or deters yet another class of languages, such as modern Greek or Navajo, from providing proper words for
  Haskell identifiers.
* In keeping with other programming languages, Haskell uses the “maximal munch” rule for both identifers and symbolic
  operators. But symbol characters are not meant to be syntactically combined with each other. The multi-character
  operators like ``<=`` are a legacy that exists only due to the small size of the legacy ASCII character set; the
  proper representation for the operation is the single character ``≤``.
* The same objection appplies to alphanumeric symbols, such as enclosed (Ⓐ) and mathematical (ℕ) alphanumerics. Even
  though these are letters and they read as identifiers, they are not meant to be combined into words.
* The language syntax is restricted to ASCII characters, even when Unicode offers superior alternatives. `GHC's
  UnicodeSyntax extension
  <https://downloads.haskell.org/~ghc/latest/docs/html/users_guide/glasgow_exts.html#ghc-flag--XUnicodeSyntax>`_
  rectifies this to some extent, but it doesn't fill all the gaps.

This proposal attempts to resolve all these issues at once.

###############
Detailed design
###############

The existing lexical syntax of Haskell variable identifiers and symbols is specified as

|   \ *varid*  → (*small* {*idCharacter*}) \\ *reservedid*
|   \ *conid*  → *large* {*idCharacter*}
|   \ *varsym* → ((*symbol* \\ :) {*symbol*}) \\ ⟨*reservedop* | *dashes*⟩
|   \ *consym* → (: {*symbol*}) \\ *reservedop*

|   \ *idCharacter* → *small* | *large* | *digit* | '
|   \ *small*    → *ascSmall* | *uniSmall* | _
|   \ *large*    → *ascLarge* | *uniLarge*

|   \ *ascSmall* → a | b | … | z
|   \ *uniSmall* → any Unicode lowercase letter
 
|   \ *ascLarge* → A | B | … | Z
|   \ *uniLarge* → any uppercase or titlecase Unicode letter

|   \ *digit*    → *ascDigit* | *uniDigit*
|   \ *ascDigit* → 0 | 1 | … | 9
|   \ *uniDigit* → any Unicode decimal digit

|   \ *symbol*    → *ascSymbol* | *uniSymbol* \\ ⟨*special* | _ | " | '⟩
|   \ *ascSymbol* → ! | # | $ | % | & | ⋆ | + | . | / | < | = | > | ? | @ | \\ | ^ | | | - | ~ | :
|   \ *uniSymbol* → any Unicode symbol or punctuation
|   \ *special*   → ( | ) | , | ; | [ | ] | ` | { | }


New lexical syntax of identifiers and symbols
#############################################

The present proposal changes the lexical syntax to the following:

|   \ *varid*   → (*small* {*idCharacter*} | *mathSmall* {*modifier*}) \\ *reservedid*
|   \ *conid*   → *large* {*idCharacter*} | *mathLarge* {*modifier*}
|   \ *varsym*  → ((*ascSymbol* \\ :) {*ascSymbol* | *combining*} | *uniSymbol* {*combining*})
|                 \\ ⟨*reservedop* | *dashes*⟩
|   \ *consym* → (: {*symbol* | *combining*}) \\ *reservedop*

|   \ *idCharacter* → *small* | *large* | *uncased* | *modifier*
|   \ *modifier* → *digit* | *combining* | '
|   \ *small*    → *ascSmall* | *uniSmall* \\ *mathSmall* | _ | *uncased* *combiningBelow*
|   \ *large*    → *ascLarge* | *uniLarge* \\ *mathLarge* | *uncased* *combiningAbove*
|   \ *uncased* → any Unicode letter with no case

|   \ *mathSmall* → any Unicode lowercase mathematical letter (i.e., with the derived property ``Math``)
|   \ *mathLarge* → any Unicode capital mathematical letter
|   \ *uniSymbol* → any Unicode symbol or punctuation

|   \ *combining* → any Unicode combining character
|   \ *combiningBelow* → any Unicode combining character below the base character
|   \ *combiningAbove* → any Unicode combining character above the base character


The changes can be explained and justified as follows:
* The new lexical syntax permits arbitrary combining characters in both identifiers and symbols.
* The purpose of the *combiningBelow* and *combiningAbove* characters is to fake the case distinction. An identifier
  that starts with a non-cased letter must have one of these combining characters immediately following the letter.
* While a single symbol token can still contain a sequence of ASCII symbols, it can only contain a single non-ASCII
  symbol character and only at the beginning. The symbol character can be followed only by combining characters.
* Equivalently, every mathematical alphanumeric symbol represents a whole identifier, together with any following
  combining characters and digits.
* As a consequence, the sequence of characters ``𝛌x.x`` would be tokenized into four distinct tokens. The identifier 𝛌
  (U+1D6CC) should be added to the list of reserved words, to prepare the ground for a future proposal that makes it a
  valid alternative for the backslash.

Examples
########

Arabic
العربية - "Arabic", a word of Arabic written in the Arabic script
ا̊لعربية - same word with a *combining ring above* the first character, marking it as capital
ا̥لعربية - same word with a *combining ring below* the first character, marking it as capital

Devanagari
भोजपुरी - "Bhojpuri", a word of the Bhojpuri language written in the Devanagari script
भो͘जपुरी - same word with a *combining dot above right* of the first character, marking it as capital
भो᪶जपुरी - same word with a *combining wiggly line below* of the first character, marking it as lowercase

ⁱfoo
ⁿbarˆ

𝚺
sin𝛼
x⃗
x′ʹ


𝐈x   = x
𝐊𝑥𝑦  = 𝑥
𝐒𝑥𝑦𝑧 = 𝑥𝑧(𝑦𝑧)

𝐖 = 𝐒𝐒(𝐒𝐊)

a⇒b = a∨¬b


#########
Drawbacks
#########

This proposal breaks the compatibility with Haskell 2010, but few programs will be affected. The most significant
compatibility break would probably be to programs that define operators as sequences of non-ASCII symbol
characters. These would now be considered multiple symbol tokens.

If implemented whole, the proposal would also make the lexical syntax of the language incrementally more complex and
harder to implement. The proposed syntax can still be expressed using regular expressions, so most lexers should have no
trouble with it. The main difficulty may be in correctly recognizing various Unicode character classes, but there are
existing libraries that can help with that.

While the proposal is rather ambitious in some ways, it changes only the lexical syntax of Haskell. As a consequence,
the unfortunate distinction between the capital and lowercase identifiers imposed by the higher-level syntax is still in
place. Scripts of non-European origin that don't have any case distinctions can now be used with the *combiningBelow*
and *combiningAbove* characters, but this is only a fig leaf.


############
Alternatives
############

As noted above, the proposal is limited to the lexical layer of the language. A more ambitious alternative would be to
eliminate the false uppercase/lowercase dichotomy from the syntax altogether. Both Agda and Idris have done that with no
obvious adverse consequences.

The Unicode Consortium itself suggests a <Default Identifier
Syntax>`https://www.unicode.org/reports/tr31/tr31-10.html#Default_Identifier_Syntax`_ that takes into consideration many
more problems than considered here, but is also much more complex that the proposed syntax.

####################
Unresolved questions
####################

It is unclear if the *combiningBelow* / *combiningAbove* hack is enough to enable the use of non-European scripts. The
proper answer can be given only by a poll of Haskell users. The set of possible answers would include:
* I only ever write English identifiers in Haskell.
* I only write identifiers using Latin or Cyrillic scripts with no diacritical modifiers.
* I wish I could write Haskell with identifiers in my native language,
  * and with this extension I would
  * the proposed extension is insufficient, but a step in the right direction
  * but the proposed extension is useless.

If this proposal were adopted, the next step would be to move up a layer to the language syntax. Most importantly, the
newly available mathematical lambda keyword should be allowed instead of its sad backslash immitation.
