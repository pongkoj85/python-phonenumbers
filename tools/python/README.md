phonenumbers Python Library
===========================

This is a Python port of [libphonenumber](https://github.com/google/libphonenumber).

Original Java code is Copyright (C) 2009-2016 The Libphonenumber Authors

Manual phonenumbers Installation
--------------------------------

Install using `setup.py`:

```console
tar xfz phonenumbers-<version>.tar.gz
cd phonenumbers-<version>
python setup.py build
sudo python setup.py install  # or su first
```

Running Tests
-------------

With phonenumbers on the Python path, run:

```console
python -m testwrapper
```

Auto-Generating Python Code
---------------------------

Several subdirectories under `python/phonenumbers` are automatically generated
from the master metadata under `resources/`:

- `python/phonenumbers/data` is generated from `resources/PhoneNumberMetadata.xml`.
- `python/phonenumbers/shortdata` is generated from `resources/ShortNumberMetadata.xml`.
- `python/phonenumbers/geodata` is generated from files under `resources/geocoding/`.
- `python/phonenumbers/carrierdata` is generated from files under `resources/carrier/`.
- `python/phonenumbers/tzdata` is generated from files under `resources/timezones/`.

The script `tools/buildmetadatafromxml.py` performs the first pair of autogenerations.
The script `tools/buildprefixdata.py` performs the latter 3 sets of autogenerations.

This approach results in a large set of Python files, but makes it easy
to apply local fixes to the formatting metadata.

The `phonenumberslite` version of the package does not include the prefix-based
metadata (in order to reduce package size), so `phonenumbers.geodata`,
`phonenumbers.carrierdata` and `phonenumbers.tzdata` are all unavailable with this
version.

Library Developer Internals
---------------------------

The Python code is derived from the original Java code, and
mostly sticks to the structure of that code to make it easier
to include future changes to the upstream code.

However, there are a number of differences:

- Naming conventions are converted to Python standards; in
  particular, method names are `connected_with_underscores`
  rather than `beingInCamelCase`.
- The `PhoneNumber` and `PhoneMetadata` classes are written by hand
  based on the Java code and the protocol buffer definitions,
  rather than by using the the Python protocol buffer library.
  This makes the mapping to the Java code easier to follow, and
  allows for the custom modifications that have been made to
  the base protocol buffer.  Attribute values of `None` are used
  to indicate that a particular (optional) attribute is not
  present (instead of `hasAttribute()` methods).
- The Java `PhoneNumberUtil` class was a singleton, and so its
  contents are included at the top level in `phonenumberutil.py`.
  Static methods from the `PhoneNumberUtil` class thus become
  functions in `phonenumberutil.py`; private and package methods
  get a leading underscore in their name.
- Accessor functions (`setAttribute()` and `getAttribute()` are
  avoided, and direct access to attributes is used instead.
- Methods named `get_something_from(object)` are typically renamed
  to `something_from(object)`.
- The `format()` methods in `PhoneNumberUtil` were renamed to
  `format_number()` to avoid clashing with the Python built-in
  `format()`.
- The internals of `phonenumberutil.py` do not have logging.
- The Python version is less concerned with speed and size
  optimization than the Java version (as Python code is more likely
  to run on a server platform, and less likely to run on an
  embedded/smartphone platform).

Much of the functionality of this library depends on regular
expressions, so it's worth highlighting the translation between
Java and Python regexps:

- Java replacement group references are `"$1 $2"` etc, Python's are
  `"\1 \2"` etc.
- Java `Matcher(x).lookingAt()` translates to Python `re_obj.match(x)`.
- Java `Matcher(x).find()` translates to Python `m = re.search(x)`.
- Java `Matcher(x).matches()` translates to Python `m = re_obj.match(x)`
  together with a check that `m.end() == len(x)`.

The last of these is encapsulated in the `fullmatch()` function in
`re_util.py`.

Some other mappings between the Java and Python versions:

|Java                                      | Python                                     |
|------------------------------------------|--------------------------------------------|
|`countryCallingCodeToRegionCodeMap`       |`COUNTRY_CODE_TO_REGION_CODE`               |
|`getSupportedRegions()`                   |`SUPPORTED_REGIONS`                         |
|`getSupportedGlobalNetworkCallingCodes()` |`COUNTRY_CODES_FOR_NON_GEO_REGIONS`         |
|`getMetadataForNonGeographicalRegion()`   |`PhoneMetadata.metadata_for_nongeo_region()`|

Release Procedure
-----------------

- Ensure that `python/HISTORY.md` file is up-to-date, and includes
  descriptions of changes in this version (adapted from the
  upstream [release notes](https://github.com/google/libphonenumber/blob/master/release_notes.txt),
  skipping the metadata changes chunks).
- Set the `__version__` field in `python/phonenumbers/__init__.py`
- Check that the list of symbols in `python/phonenumbers/__init__.py` `__all__` is
  up to date.  The `tools/python/allcheck.py` script helps with this.
- Optionally, force metadata regeneration:
    `cd tools/python && make metaclean alldata`
- Check that the unit tests all run successfully:
    `cd tools/python && make test`
- Optionally, check that metadata regeneration works in Python 3:
    `cd tools/python && make PYTHON=python3 metaclean alldata`
- Check that the unit tests all run successfully in Python 3:
    `cd tools/python && make PYTHON=python3 test`
- Check that Python 2.5 is still supported:
    `cd tools/python && make PYTHON=python2.5 test`
- Create a vX.Y.Z tag:
    `git tag vX.Y.Z`
- Push the tag to Github with:
    `git push <github-remote> vX.Y.Z`
- Push the lite package to PyPI with:
    ```
    cd python && rm -rf build dist && ./setup.py lite sdist bdist_wheel
    twine check dist/*
    twine upload dist/*
    ```
- Push the package to PyPI with:
    ```
    cd python && rm -rf build dist && ./setup.py sdist bdist_wheel
    twine check dist/*
    twine upload dist/*
    ```
