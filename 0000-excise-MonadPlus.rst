- Feature Name: Excise the MonadPlus class
- Start Date: 2018-12-16
- RFC PR: Leave this empty, filled on proposal accept
- Haskell Report Issue: Leave this empty, filled on proposal accept



#######
Summary
#######

The type class ``MonadPlus`` appears in the language report even
though it has no bearing on the language specification. It is nothing
but a liability, and is best removed.


##########
Motivation
##########

The venerable ``Monad`` class has good reasons to be in the report,
the main one being the ``do`` syntax which would be difficult to
define without ``Monad``. Yet its presence there, together with other
backward-compatibility issues, had prevented the ``Applicative`` class
from assuming its rightful place in the class hierarchy for years.

We have a very similar problem with the ``Alternative`` and
``MonadPlus`` classes. We can solve it in the same way and introduce
``Alternative`` into the report as a new superclass for
``MonadPlus``. In this case, however, we have an easier solution
available: simply don't mention either class.


###############
Detailed design
###############

``MonadPlus`` is not exported from ``Prelude``, and neither are ``guard``
and ``mplus``, the only two functions whose types reference the class.
The report mentions the class only in the non-normative definition of the
``Control.Monad`` module, and in the lists of classes instantiated by the
list type and ``Maybe`` type. The function ``guard`` is used in the
example implementation of fixity resolution for Haskell expressions.
These references can all be easily removed without affecting the rest of
the report.


#########
Drawbacks
#########

One could argue for tradition, or for having a bigger language report
as a good thing by itself. Otherwise, the only drawback of losing the
current perfunctory appearance of ``MonadPlus`` is not being able to
expand on it.


############
Alternatives
############

A diametric alternative to dropping an unused class from the language
report would be to start using it. We could export the class and its
methods from ``Prelude`` and thus justify its presence in the report.

This solution could be a good one for more fundamental and more
popular classes like ``Semigroup`` or even ``Alternative``, but
``MonadPlus`` has never gained their high profile despite its early
appearance.


####################
Unresolved questions
####################

The fate of ``MonadPlus`` is loosely linked to that of
``Alternative``. As mentioned in the previous section, we could choose
to add ``Alternative`` to the ``Prelude`` even as we excise
``MonadPlus``. That question is out of this proposal's scope.
