# Changelog
{:.no_toc}

* TOC will be rendered here
{:toc}

# 0.1.0 - Initial release

## 0.1.2 - Allow sending/validation of JSON lists

## 0.1.3 - Fix global configuration loading via pytest command line

## 0.1.4 - Fix global configuration if it wasn't actually passed

## 0.1.5 - Fix temporary file wrapping on windows

# 0.2.0 - Add python 2 support

## 0.2.1 - Add option to install 'pytest' extra

## 0.2.2 - Support for 'verify' arg to requests

## 0.2.3 - quote nested json in query parameters

## 0.2.4 - Fix log format interpolation for py2

## 0.2.5 - Fix empty yaml files hard-failing

# 0.3.0 - Use a persistent requests Session to allow cookies to be propagated forward through tests

# 0.4.0 - MQTT support

# 0.5.0 - Add special 'tavern' key for formatting magic variables, and don't strictly enforce some HTTP verbs not having a body

## 0.5.1 - Add regex validation function and verify tests at run time, not discovery time

## 0.5.2 - Add MQTT TLS support and fixes to formatting nested arrays/dicts

## 0.5.3 - Update README

## 0.5.4 - Add 'meta' key to request block

currently the only key in 'meta' is clear_session_cookies which wipes the session cookies before the request is made

# 0.6.0 - Allow multiple global config options on the command line and in pytest config file

## 0.6.1 - Fix implementation of 'auth' keyword

# 0.7.0 - Add new 'anything' constructor for matching any value returned which should now also work with nested values. Also add special constructors for int/float types

## 0.7.1 - fix delay_after/before to accept float arguments

## 0.7.2 - Fix warning on incorrect status codes

## 0.7.3 - Improve error handling in parser errors

## 0.7.4 - Fix python 2

## 0.7.5 - Fix pytest-pspec error

## 0.7.6 - Move dict utilities around

## 0.7.7 - Improve validation on the type of block returned

# 0.8.0 - Fix matching magic variables and add new type sentinels for matching

## 0.8.1 - Fix formatting env vars into included variables

## 0.8.2 - Cleanup of type conversion code and better list item validation

# 0.9.0 - Add file upload capability

## 0.9.1 - Fix logging library warning

## 0.9.10 - Add new tag to match floating point numbers approximately in responses

## 0.9.2 - Minor improvement to error messages

## 0.9.3 - Improve error reporting from dictionary mismatches and allow regex checks in headers

## 0.9.4 - Fixes to type conversion tags, and add a new 'anybool' type sentinel to match either True or False

## 0.9.5 - Fix type conversion tokens and add more robust integration tests for them

## 0.9.6 - Add bool conversion type token as well

## 0.9.7 - Fix error in formatting MQTT variables

## 0.9.8 - Fix tavern overriding content type header when sending a file with extra headers

## 0.9.9 - Allow nesting of variables in included files that can be access using dot notation

# 0.10.0 - Add basic plugin system

## 0.10.1 - Slightly improve docstrings for use with pytest-pspec

## 0.10.2 - Fix python 2 type token issue

# 0.11.0 - Marking, strict key checking, and multiple status codes

- Add ability to use custom QoS for subscribing in MQTT
- Add pytest marks to tests
- Add strict key checking controllable by cli/per test
- Add verification for multiple status codes
- Improve 'doc' of test for pytest-pspec
- Add internal xfail for testing Tavern
