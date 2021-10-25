Adding a package to idris2-pkgs
===============================

We’ve `built a package <new-project.rst>`__ with idris2-pkgs, and now
want to publish it to this repository.

Adding to flake inputs
----------------------

The first thing to do is add our source - here, a repo hosted at
https://github.com/example/mypkg - to ``idris2-pkgs::flake.nix``.
Idris2-pkgs assumes all package inputs are not flakes, so add
``flake = false;`` even if the project repository is a flake.

.. code:: nix

   {
      inputs.mypkg = { url = "github:example/mypkg"; flake = false; };
   }

If ``mypkg = idrisPackage ./. { };`` was sufficient to build the
package, that’s probably enough. We can build the package with
``nix build .#mypkg``, ensure it works as expected, commit the change to
git, and submit a pull request.

Configuring the build
---------------------

If the package was instead built with something like
``mypkg = idrisPackage ./. cfg;`` for some attrset ``cfg``, we’ll need
to give ``idris2-pkgs`` that configuration. The place to do that, and
all other configuration overrides, is in
`packageSet.nix <../packageSet.nix.rst>`__.

Here, add an entry into ``packageSet.nix::packageConfig`` so that
``packageConfig.mypkg = cfg``.

Likewise, if the package used ``useRuntimeLibs``, add the package name
to the list ``packageSet.nix::needRuntimLibs``.

Again, run ``nix build .#mypkg``, run any tests you feel are necessary,
and submit a PR.

Naming considerations
---------------------

Collisions
~~~~~~~~~~

Neither idris2 nor nix flakes handle name collisions overly well.
``idris2-pkgs`` is set up so that each flake input can be given a unique
name, and the ``cfg.extraPkgs`` input to ``idrisPackage`` can function
as a map from what’s written in an ipkg file to an idris derivation, but
it’s much cleaner if packages are given unique names.

Unicode
~~~~~~~

Idris2 allows unicode in package names, but support for unicode names is poor in Nix.
This section is less of a guide, and more notes for potential workarounds.

- Nix does not allow unicode names, except when an attribute name and explicitly a string.

.. code-block::

   let
      📦 = 42;               # error
      a = { "📦" = mypkg; }; # accepted
      b = { 📦 = mypkg; };   # error
      c = a."📦";            # accepted
      d = with a; "📦";      # d is the string "📦"
      e = with a; 📦;        # error

-  Nix flake inputs do not allow unicode.

extraPackages
-------------

Any packages which do not have their own, dedicated flake input can be
added to the package set in ``packageSet.nix::extraPackages``.
