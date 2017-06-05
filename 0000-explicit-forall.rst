- Feature Name: Allow the explicit ``forall`` keyword
- Start Date: 4st June 2017
- RFC PR: Leave this empty, filled on proposal accept
- Haskell Report Issue: Leave this empty, filled on proposal accept



#######
Summary
#######

Adopt the ``ExplicitForAll`` extension by default, in the form already implemented by GHC.

##########
Motivation
##########

Several widely used language extensions depend on ``ExplicitForAll``:

- ExistentialQuantification
- GADTs
- ImpredicativeTypes
- LiberalTypeSynonyms
- Rank2Types
- RankNTypes
- ScopedTypeVariables

It seems highly likely that at least some of these extensions will become standard in the future. This proposal is the
first prerequisite before any other extension above can become standard.

###############
Detailed design
###############

The ``forall`` keyword is not a reserwed word, so it does not affect the lexical structure of Haskell. That is true of
GHC today, as this example compiles::

    {-# LANGUAGE ScopedTypeVariables #-}
    f :: forall x. Num x => x -> x
    f forall = forall + 1

The language syntax would be modified to allow the optional ``forall`` keyword and type context, followed by a
comma-separated list of type variables and a period. The simplest point to adjust the grammar appears to be at the
``type`` production::

     type    →  btype "->" type
             |  "forall" tyvars "." type
             |  context "=>" type
             |  btype
     tyvars  →  tyvar₁ , …, tyvar_n

This change would then allow several other grammar productions to be simplified::

    gendecl     →  vars "::" type
                |  …
    exp         →  infixexp "::" type
                |  …
    simpletype  →  tycon tyvars
    inst        →  …
                |  ( gtycon tyvars )
                |  ( tyvars )
                |  …

This proposal stops at the syntax. In the unfortunate case this proposal is accepted and no other follow-up proposal is,
the language semantics would remain exactly the same. The syntactic extensions (i.e., the second and third alternative
of the ``type`` production) would then be ignored at the very beginning of a type, and rejected elsewhere else.

#########
Drawbacks
#########

The proposal is largely backward compatible with Haskell 2010. There still is a slight compatibility break: any existing
code that uses the keyword ``forall`` as a type variable name will break. It's doubtful there is any existing Haskell
code like this, unless it was intentionally written to break.

############
Alternatives
############

The Haskell' seemingly stalled, I chose to put together the most conservative and necessary proposal I could imagine. In
any other situation, I'd have gone directly for ``ScopedTypeVariables``. Still, this is one way to start.

There are other ways one could organize the grammar, some more conservative, but the present proposal appears to fit
the GHC implementation. That makes its implementation free and minimizes any compatibility issues.

The one exception to the previous claim is that GHC's ``ExplicitForAll`` extension goes slightly beyond this proposal
and allows the ``forall`` keyword at the beginning of every ``instance`` declaration::

    instance forall a. Eq a => Eq [a] where

This particular syntax has no effect with any language extension, so the proposal deviates from GHC and does not allow
it. Universal quantification is implicit in both ``class`` and ``instance`` declarations. GHC seems inconsistent in
allowing the explicit ``forall`` only in the latter.

####################
Unresolved questions
####################

None I'm aware of.
