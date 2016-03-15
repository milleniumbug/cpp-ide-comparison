C++ IDE Comparison
------------------

Extensible comparison between various C++ IDEs

Goal
----

Discovery of new(ly fixed) features in C++ IDEs and possibly new IDEs in hope of discovering a C++ IDE that's awesome enough to be included in [loungesome-cpp](https://github.com/LoungeCPP/loungesome-cpp).

Test procedures
-----

Test procedures are contained in `_tests` directory - they are versioned with the following versioning scheme:

- MAJOR_MINOR (where MAJOR and MINOR are non-negative integers)
- pre-release tests (0_?) can be removed or modified at any time in a way that invalidates already existing tests.
- released tests (X_? with X > 0) cannot be removed. They can be only modified without bumping version if modification fixes an error in the testing procedure in a way that doesn't invalidate already done tests.
- bumping the version involves creating a new file (NAME_NEWVERSION.md) where the NEWVERSION is created by bumping the version by major revision (major-bumping) or minor revision (minor-bumping).
- minor-bumping is done when a test is extended in a way that you can extend already made test with new results. It involves incrementing a minor version number.
- major-bumping is done when a test is completely revised in a way that it's pretty much a new test, but it's similar enough ideologically it could have the same name. It involves incrementing a major version number and resetting the minor version number back to 0.
- any test can be marked as deprecated by adding a deprecation notice at the end of the file in the following format:

```

Deprecated
----------

This test is deprecated for reasons X, Y, and Z. 
```

IDEs
----

Testing subjects: For every tested IDE the repository contains tests with the same name as the corresponding test procedure. It also contains the file named `version` with the friendly name of the IDE (for example `Visual Studio Community 2015 Update 1`)