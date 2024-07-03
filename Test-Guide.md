### Considerations
*Tests have not been fully converted to this style in all repositories.*

It is important that tests can be evaluated visually. Otherwise it becomes impossible to determine if the test itself is correct. To achieve this result we apply certain style rules as follows:

#### Declarativeness.

This means that test execution is unconditional, there is no branching. Loops, if statements and the ternary operator are prohibited. Test helpers may include conditional logic, but then these should be tested for correctness.

#### Transparent Names
When a test or set of tests fail this helps a developer to quickly isolate the cause. The naming convention is:
```c
BOOST_AUTO_TEST_CASE([class|file]__[method|function]__[condition]__[expectation])
```
For example:
```c
BOOST_AUTO_TEST_CASE(reservation__stopped__import_last_block__true)
```

Tests against a method, constructor or function override should use a simple ordinal numbering scheme to group test of the same signature. For example:
```c
BOOST_AUTO_TEST_CASE(configuration__construct1__none_context__expected)
BOOST_AUTO_TEST_CASE(configuration__construct1__mainnet_context__expected)
BOOST_AUTO_TEST_CASE(configuration__construct1__testnet_context__expected)
BOOST_AUTO_TEST_CASE(configuration__construct2__none_context__expected)
```

Group tests in a single `.cpp` file to mirror the source file naming convention. If it becomes necessary to break up the test file into independent files for a single corresponding class or source file, use the following test file naming convention (or a subdirectory):
```
class__method.cpp
file__function.cpp
```

Group tests semantically using `BOOST_AUTO_TEST_SUITE(...)`. This allows tests to be executed independently. These can be grouped hierarchically if necessary. Generally there is one suite per file or class under test.

#### Test One Thing
Test just one thing. The test name indicates the conditions and expectation of the test. When a test or set of tests fail this allows a developer to quickly isolate the cause. Sometimes it is convenient to test a set of results under one condition, such as a true result code and an expected out value. But these combinations should be semantically associated in that they are necessary to test together. For tests with very costly setup, such as mock databases and network operations, this rule can be bent.

#### Line Length
A line of test code should not be wrapped. For this reason line length is not limited as in source files. However intermediate variables can and should be used to limit line length. Structure definitions may be structured across multiple lines.

#### Test Helpers
The BOOST test helpers expose information about the test when there is a failure. It is preferred to use the least general test helper available:
```c
BOOST_REQUIRE(!foo.is_valid())
```
instead of
```c
BOOST_REQUIRE_EQUAL(foo.is_valid(), false)
```
If values are serializable then use:
```c
BOOST_REQUIRE_EQUAL(foo.size(), 5u)
```
as opposed to
```c
BOOST_REQUIRE(foo.size() == 5u)
```
and in all cases place the condition under test on the left side and the test expectation on the right.
