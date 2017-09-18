- Feature Name: Generalized, named, and exportable ``default`` declarations
- Start Date: 2017-09-14
- RFC PR: Leave this empty, filled on proposal accept
- Haskell Report Issue: Leave this empty, filled on proposal accept



#######
Summary
#######

The ``default`` declaration as specified in Haskell 2010 report is very limited. This proposal aims to make it useful
while keeping backward compatibility. Another goal of the proposal is to leave the ``default`` declaration in the deep
language background, in a manner of speaking; so that expert users can apply it to help beginners who are kept
blissfully unaware of it.

##########
Motivation
##########

Section `4.3.4<https://www.haskell.org/onlinereport/haskell2010/haskellch4.html#x10-790004.3.4>` of the Haskell 2010
language report specifies the behaviour of the ``default`` declaration, and makes it clear that the declaration is
fairly useless, as it applies
- only within the current module,
- only to the class ``Num``.

The first limitation means that every user is forced to repeat the same declaration in every module if they should try
to use a non-standard numeric data type in an ambiguous way.

The reason for the latter limitation is that the numeric literals are the only literals with ambiguous types in the
language report. Since then, however, the ``OverloadedStrings`` and ``OverloadedLists`` language extensions have made
more syntactic constructs ambiguous. The former in particular is commonly used for more convenient coding of ``Text``
literals.

###############
Detailed design
###############

The present proposal is basically an expanded version of the earlier `name the
class<https://prime.haskell.org/wiki/Defaulting#Proposal1-nametheclass>` Haskell Prime proposal.

The Haskell 2010 language report specifies the following syntax for the ``default`` declaration:
.. math::
   topdecl 	→ 	\texttt{default} (type_1 , … , type_n) 	    (n ≥ 0) 

where each :math:`type_i` must be an instance of class ``Num``.

Naming the class
================

In the current language standard, the ``default`` declaration implicitly applies to class ``Num`` only. The proposal is
to make this class explicit, so the syntax becomes
.. math::
   topdecl 	→ 	\texttt{default} class? (type_1 , … , type_n) 	    (n ≥ 0) 

where each :math:`type_i` must be an instance of the specified *class*. If no *class* is specified, the earlier default
of ``Num`` is assumed.

The types may belong to any kind, but the *class* must have a single parameter.

Exporting the defaults
======================

Another thing the current report specifies is that the declaration applies only within the current module. This proposal
does not modify that behaviour: a ``default`` declaration by itself does not apply outside its module. That is the
purpose of another extension to the module export list. To the existing syntax
.. math::
   export 	→ 	qvar
      | 	qtycon [(..) | ( cname_1 , … , cname_n )] 	    (n ≥ 0)
      | 	qtycls [(..) | ( var_1 , … , var_n )] 	    (n ≥ 0)
      | 	\texttt{module} modid

would be added another alternative
.. math::
      | 	\texttt{default} class

The effect of this export item would be to export the default declaration for the named *class* that is in effect in the
module, which can mean either that it's declared in the same module or that it's imported from another module.

When exporting a ``default Num`` declaration, the class ``Num`` has to be explicitly named like any other class.

An ``import`` of a module always imports all the ``default`` declarations listed in its export list. There is no way to
exclude any of them. This is the default option for this proposal, but there are Alternatives_.

Rules for disambiguation of multiple declarations
=================================================

Only a single ``default`` declaration can be in effect in any single module for any single class. If there is more than
one ``default`` declaration in scope, the conflict is resolved using the following rules:

1. Two declarations for two different classes are not considered to be in conflict; they can, however, clash at a
   particular use site as we'll see in the following section.
2. Two declarations for the same class explicitly declared in the same module are considered a static error.
3. A ``default`` declaration in a module takes precedence over any imported ``default`` declarations for the same class.
4. For any two imported ``default`` declarations for the same class
   .. math:
      \texttt{default} C (Type_1^a , … , Type_m^a)
      \texttt{default} C (Type_1^b , … , Type_n^b)
   if *m* ≥ *n* and the second type sequence :math:`Type_1^b , … , Type_n^b` is a sub-sequence of the first sequence
   :math:`Type_1^a , … , Type_m^a`, the first declaration subsumes the second one which can be ignored.
5. If a class has neither a local ``default`` declaration nor an imported ``default`` declaration that subsumes all
   other imported ``default`` declarations for the class, the conflict between the imports is unresolvable. The effect
   is to ignore all ``default`` declarations for the class, so that no declaration is in effect in the module.

Rules for disambiguation at the use site
========================================

The disambiguation rules are a conservative extension of the existing rules in Haskell 2010, which state that ambiguous
type variable *v* is defaultable if:

    - *v* appears only in constraints of the form *C* *v*, where *C* is a class, and
    - at least one of these classes is a numeric class, (that is, *Num* or a subclass of *Num*), and
    - all of these classes are defined in the Prelude or a standard library.

    Each defaultable variable is replaced by the first type in the default list that is an instance of all the ambiguous
    variable’s classes. It is a static error if no such type is found.

The new rules require instead that 

- *v* appears only in constraints of the form *C* *v*, where *C* is a class, and
- there is a ``default`` declaration in effect for at least one of these classes.

The type selection process remains the same for any given class *C*. If there are multiple *C* *v* constraints with
competing ``default`` declarations, they have to resolve to the same type. In other words, the type selected for
defaulting has to be the first type that satisfies all the class constraints, in every ``default`` declaration in
effect. It is a static error for different ``default`` declarations to resolve to different types, or for any of them to
not resolve to any type.

Examples
========

The main motivation for expanding the ``default`` rules is the widespread use of the ``OverloadedStrings`` language
extension, usually for the purpose of using the ``Text`` data type instead of ``String``.

With this proposal in effect, and some form of ``FlexibleInstances``, the Haskell Prelude could export the declaration

.. Haskell::
   default IsString (String)

Then a user module could activate the ``OverloadedStrings`` extension without triggering any ambiguous type errors,
still using the ``String`` type from the Prelude.

The authors of the alternative string implementations like ``Text`` would export the following declaration instead:

.. Haskell::
   default IsString (Text, String)

Any user module that activates the ``OverloadedStrings`` extension and imports ``Data.Text`` would thus obtain the
default declaration suitable for working with ``Text`` without any extra effort. Since the Prelude declaration's list of
types is a sub-sequence of the latter declarations, it would be subsumed by it.

A user module could, by chance or by design, import two independently-developed modules that export competing defaults
for the same class, for example the previous ``Text`` module and the ``Foundation.String`` module with its own exported
declaration

.. Haskell::
   default IsString (Foundation.String, String)

In this case the importing module would discard both contradictory declarations. If the developers wish a particular
default, they just have to declare it in the importing module. Furthermore, if they export this ``default`` declaration,
every importer of the module will have the conflicts resolved for them:

.. Haskell::
   module ProjectImports (Text.Text, Foundation.String, default IsString)

   import qualified Data.Text         as Text
   import qualified Foundation.String as Foundation

   default IsString (Text.Text, Foundation.String, String)

#########
Drawbacks
#########

Use-site conflicts
==================

The earlier `Haskell Prime proposal<https://prime.haskell.org/wiki/Defaulting>` notes several ways in which defaults for
different classes can contradict each other:

.. Haskell::
   default A (Int,String,())
   default B (String,(),Int)
   (A t, B t) => t

   default C (Int, Double, String, ())
   default D (Double,String,Int,())
   (C t, D t) => t

The solution to this depends on where the conflicting defaults come from.
- If they are declared in the same module: just don't do that; or
- if the defaults are imported, declare one or more overriding defaults to resolve the conflict.

############
Alternatives
############

Declaration imports
===================

Most features of the present proposal are completely determined by the constraints of backward compatibility and ease of
use, but in case of declaration imports the choice was more arbitrary.

As stated above, the default option is to automatically import all ``default`` declarations the module exports, with no
choice given. If a default is unwanted, it can easily be modified or turned off by another ``default`` declaration.

This choice has been made because it seems to be easiest on the beginners: they don't need to know anything about
defaults, especially if they work with a prepared set of imports that take care to resolve all ``default`` conflicts for
them.

An alternative approach would be to treat default exports the same way normal named exports are treated: if an
``import`` declaration explicitly lists the names it wants to import, it has to also explicitly list ``default`` and the
class name for each desired default declaration.

An optional extension compatible with either of these alternatives would be to allow the ``hiding`` clause to list the
``default`` declarations that should not be brought into the scope. This is not a part of the present proposal simply
because it's unnecessary.

Multi-parameter type classes
============================

This proposal does not cover MPTCs, but this section will speculate how it could be extended to cover them in future.

First, let us generalize the single-parameter type class defaults by expanding the class name and the type name to full
constraints. The above example

.. Haskell::
   default IsString (Text, String)

would then be written as

.. Haskell::
   default IsString t => (t ~ Text, t ~ String)

The former notation would be syntactic sugar for the latter. Since comma is already used as a constraint combinator,
we'd actually prefer to replace it by something else. The logical choice would be semicolon, which tends to be contained
in braces:

.. Haskell::
   default IsString t => {t ~ Text; t ~ String}

So now we have a general enough notation to accommodate MPTCs. We could, for example, say

.. Haskell::
   default HasKey m k => {m ~ IntMap, k ~ Int; m ~ Map k; m ~ [k]; m ~ Map k, k ~ String}

The defaulting algorithm would replace the constraint on the left hand side consecutively by each semicolon-separated
constraint group on the right-hand side until it finds one that completely resolves the ambiguity.

Again, this extension is not a part of the proposal because it would depend on type equality at least, and because its
utility is unproven. Still, it's good to know that the proposal does not close off this potentially important
development direction.

####################
Unresolved questions
####################

This proposal does not cover GHCi and its special defaulting behaviour.
