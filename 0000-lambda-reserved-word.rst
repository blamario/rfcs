- Feature Name: Lambda the Ultimate Reserved Word
- Start Date: 2017-10-07
- RFC PR: Leave this empty, filled on proposal accept
- Haskell Report Issue: Leave this empty, filled on proposal accept



#######
Summary
#######

In functional programming, lambda is much more than just another letter! Let's free it from the drudgery of serving as
nothing more that a potential part of an identifer.


##########
Motivation
##########

The theoretical foundation for Haskell and most other functional languages is called lambda calculus. Outside the
constraints of Haskell, lambda calculus is written with the Greek lowercase letter lambda, λ. Every once in a while,
newcomers to Haskell ask why they have to use the ugly ASCII backslash instead.

The answer is simple. Mathematicians and computer scientits may think of lambda and other Greek letters as mathematical
symbols, but the reason they're called Greek letters is because Greeks use them to make words in their language. If we
made the lowercase letter lambda a reserved symbol, identifiers in Greek language could not contain it any more, and
that seems very wrong.

This proposal is a compromise that aims to satisfy Greeks and mathematicians alike.


###############
Detailed design
###############

The existing lexical syntax of Haskell variable identifiers is specified as

|   \ *varid* → (*small* {*small* | *large* | *digit* | ' }) \\ *reservedid*

where *small*, *large*, and *digit* are respectively a lowercase Unicode letter or underscore, an upper or titlecase
Unicode letter, and a Unicode decimal digit.

The Greek language objection to making the lowercase lambda special closes off the simple approach of excluding λ from
*small*, but it's also missing a nuance or two. In particular, the typical use of lambda-the-symbol in computer science
is something like ``λx.x``, and λx is definitely not a word of the Greek language.

The present proposal takes advantage of this feature by changing the *varid* production to

|   \ *varid* → ((*small* \\ λ) {*small* | *large* | *digit* | ' }) \\ *reservedid*
|             | λ (*greek* | *digit* | ') {*small* | *large* | *digit* | ' }

where *greek* refers to any Greek letter in any case. In other words, a leading λ followed by a letter can start an
identifier only if that letter is Greek.

Under this proposal, ``λ``, ``λx``, or ``λfoo`` would not be valid identifiers any more. On the other hand, Greek words
like ``λουκουμάς`` would remain valid identifiers, together with some weirder tokens like ``λ2``, ``λ'`` or ``aλ``.


#########
Drawbacks
#########

This proposal breaks the compatibility with Haskell 2010, but few programs are likely to be affected.

It also makes the lexical syntax of the language incrementally more complex and harder to implement, especially compared
to a naïve solution that simply excludes λ from identifers anywhere and throws the Greek language under the bus. The
proposed syntax can still be expressed using regular expressions, so most lexers should have no trouble with it.


############
Alternatives
############

As mentioned above, one alternative would be to give up on the Greek language speakers, but that would seem rather
narrow-minded.

There are other possible ways to draw the distinction between Greek words and mathematical uses of λ. One particular
sticking point is the question of whether a lone lambda character should be a valid identifier. This proposal deems it
not one and reserves it for a more useful purpose, but one could argue for a more conservative approach.


####################
Unresolved questions
####################

This proposal stops at restricting the identifier syntax. If accepted, there should be a follow-up proposal or two on
the best use of the newly available λ. Should we be content with allowing λ instead of \\, or should we also admit the
period and allow expressions like ``λx.x*x``? What about other Unicode characters like →, ⇒, or ∀? Whatever should
happen, this proposal clears the way by breaking the backward compatibility and freeing λ from the confines of the
identifier syntax.
