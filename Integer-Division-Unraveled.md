Sometimes the most trivial coding tasks can be unexpectedly challenging. In working with Python equivalence I found it necessary to implement floored signed integer modulo in C++.

A quick stop at Stack Overflow shows how confounding this can be. There are several different approaches, most of which either do not support signed operands, cause overflows, throw exceptions, and/or just don't work. Even the "accepted" answers are flawed in such ways, with large numbers of upvotes. So I decided to just figure it out myself.

## Objectives
* Provide ceilinged and floored `\` and `%` functions, for all integer types.
* Maintain the failure behavior of native operators (i.e. `x / 0` and `x % 0`)
* Maintain the result type deduction of native operators.
* Maintain overflow behavior of native operators.
* Avoid unnecessary computation.

## Hacks
Integer division is actually a challenging subject. Programming languages implement it in [different ways](https://en.wikipedia.org/wiki/Modulo_operation). Python actually [changed the behavior](https://www.python.org/dev/peps/pep-0238/) of `\` and `%` in a minor version release! This of course led to additional scrutiny in the original Python equivalence task.

> We propose to fix this by introducing different operators for different operations: x/y to return a reasonable approximation of the mathematical result of the division ("true division"), x//y to return the floor ("floor division"). We call the current, mixed meaning of x/y "classic division".

Common hacks for "ceilinged" division in C/C++ include:

```cpp
// This causes (non-inherent) overflows and is limited to positive integers.
auto q = (x + (y - 1)) / y;

// This is limited to positive integers.
auto q = (x / y) + !!(x % y);

// This warns on unsigned types (tautological compare).
auto q = (x / y) + (((x < 0) != (y < 0)) && (x % y);
```

## C++ Standard
ISO standards are copyrighted. A working draft is available [here](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2013/n3690.pdf). The relevant sections are listed below.

#### 5.6 Multiplicative operators
2) The operands of * and / shall have arithmetic or unscoped enumeration type; the operands of % shall have integral or unscoped enumeration type. The usual arithmetic conversions are performed on the operands and determine the type of the result.
4) The binary / operator yields the quotient, and the binary % operator yields the remainder from the division of the first expression by the second. If the second operand of / or % is zero the behavior is undefined. For integral operands the / operator yields the algebraic quotient with any fractional part discarded; if the quotient a/b is representable in the type of the result, (a/b)*b + a%b is equal to a; otherwise, the behavior of both a/b and a%b is undefined.

## Key Concepts
* `/` supports integer operands.
* `%` supports (only) integer operands.
* integer `/` yields the quotient with the fractional part discarded.
* `(x % y)` yields as a remainder the fractional part discarded by `(x / y)`.
* `(x / 0)`and `(x % 0)` by zero are undefined operations.
* for both operands integer, the operators determine and yield an integer result type.
* result is undefined for both if the quotient is not representable in the result type.
* the following identity relation otherwise always holds: `(x / y) * y + (x % y) == x`.

## Rounding
Integer division cannot retain a remainder. There are several rounding approaches. The common ones are called:

* "floored" (rounded down - becomes more negative)
* "ceilinged" (rounded up - becomes more positive)
* "truncated" (dropped remainder - becomes closer to zero)

All rounding methods return the same result if there is no remainder. The distinction is in rounding direction. C++ specifies "truncated" (also called "toward zero") division. So in the case of a positive quotient, the quotient is floored (less positive), and in the case of a negative quotient, the quotient is ceilinged (less negative).

The near-universal approach to changing rounding method is to rely on native operations and to adjust the quotient and remainder accordingly. In changing rounding behavior from native C++ (truncated) to either floored or ceilinged, one must know whether there is a truncation (and its value when implementing `%`) and the sign of the quotient. If there is no truncation, there is nothing to adjust. And because truncated is a non-continuous method, with a bifurcation based on quotient sign, the adjustment must be conditioned on quotient sign. These are the necessary conditions for adjustment.

* if `(x % y) != 0` then a remainder is truncated by `(x / y)`.
* if `(x < 0) != (y < 0)` then the sign of `(x / y)` is `-`.
* if `(x < 0) == (y < 0)` then the sign of `(x / y)` is `+`.

To satisfy the identity relation, both division and modulo will be adjusted if there is an adjustment required. There is an adjustment required in any case of a non-zero remainder when the truncation is not already ceilinged (when implementing ceilinged) or not already floored (when implementing floored). Positive quotient truncation is floored (less positive) and negative quotient truncation is ceilinged (less negative).

* Floored adjusts truncation when there is a remainder of a negative quotient truncated division.
* Ceilinged adjusts truncation when there is a remainder of a positive quotient truncated division.

## Adjusting the Quotient
The quotient adjustment is always magnitude 1. A division truncation (not the remainder) is by definition fractional (magnitude less than 1). Therefore the quotient resulting from any rounding method varies by 0, +1, or -1 from any other method. When adjusting quotients from truncated (TQ) to floored (FQ), the adjustment is always `-1` (more negative), and from truncated to ceilinged (CQ) is always `+1` (more positive). Therefore `CQ > FQ` and `CQ - FQ = 1`. With truncated remainder (TR) the following relations hold:

```
Where: TR == 0
TQ = TQ
FQ = TQ
CQ = TQ

Where: TR != 0 && TQ > 0
TQ = TQ
FQ = TQ
CQ = TQ + 1

Where: TR != 0 && TQ < 0
TQ = TQ
FQ = TQ - 1
CQ = TQ
```

## Adjusting the Remainder
The sum of the truncated remainder and adjusted remainder is +/- the divisor. The floored quotient adjustment from truncated is `-1` (more negative). Therefore the remainder adjustment is `+ divisor`. The floored adjustment rounded down, so the remainder is the positive complement. The ceilinged quotient adjustment from truncated is `+1` (more positive). Therefore the remainder adjustment is `- divisor`. The ceilinged adjustment rounded up, so the remainder is the negative complement. So in the floored adjustment of truncated modulo, the divisor is added and in ceilinged adjustment, the divisor is subtracted.

The remainder adjustment is derived from the identity relation after applying the division adjustment (+/-1). The following expressions demonstrate the correctness, in all cases, of adjusting the remainder by the product of `-1`, the division adjustment, and the divisor. The signs (and values) of `x` and `y` can be modified to cover the cases not shown.

```cpp
constexpr auto x = 4;
constexpr auto y = 3;
static_assert(((+x / +y) + 1) * +y + ((+x % +y) - +y) == +x, "+1++");
static_assert(((+x / +y) - 1) * +y + ((+x % +y) + +y) == +x, "-1++");
static_assert(((-x / -y) + 1) * -y + ((-x % -y) - -y) == -x, "+1--");
static_assert(((-x / -y) - 1) * -y + ((-x % -y) + -y) == -x, "-1--");
static_assert(((-x / +y) + 1) * +y + ((-x % +y) - +y) == -x, "+1-+");
static_assert(((-x / +y) - 1) * +y + ((-x % +y) + +y) == -x, "-1-+");
static_assert(((+x / -y) + 1) * -y + ((+x % -y) - -y) == +x, "+1+-");
static_assert(((+x / -y) - 1) * -y + ((+x % -y) + -y) == +x, "-1+-");
```
```
Where: TR == 0
TR = 0
FR = 0
CR = 0

Where: TR != 0 && TQ > 0
TR = TR
FR = TR
CR = TR - Divisor

Where: TR != 0 && TQ < 0
TR = TR
FR = TR + Divisor
CR = TR
```

## Implementation
These templates determine the sign of a signed or unsigned integer type.

```cpp
template <typename Integer, if_signed_integer<Integer> = true>
constexpr bool is_negative(Integer value) noexcept
{
    return value < 0;
}

template <typename Integer, if_unsigned_integer<Integer> = true>
constexpr bool is_negative(Integer) noexcept
{
    return false;
}
```
These templates are used to determine rounding direction.
```cpp
template <typename Factor1, typename Factor2,
    if_integer<Factor1> = true, if_integer<Factor2> = true>
constexpr bool is_negative(Factor1 factor1, Factor2 factor2) noexcept
{
    return is_negative(factor1) != is_negative(factor2);
}

template <typename Dividend, typename Divisor,
    if_integer<Dividend> = true, if_integer<Divisor> = true>
constexpr bool no_remainder(Dividend dividend, Divisor divisor) noexcept
{
    return (dividend % divisor) == 0;
}
```
These templates combine those preceding into a single answer for a given rounding method.
```cpp
template <typename Dividend, typename Divisor,
    if_integer<Dividend> = true, if_integer<Divisor> = true>
constexpr bool is_ceilinged(Dividend dividend, Divisor divisor) noexcept
{
    return is_negative(dividend, divisor) || no_remainder(dividend, divisor);
}

template <typename Dividend, typename Divisor,
    if_integer<Dividend> = true, if_integer<Divisor> = true>
constexpr bool is_floored(Dividend dividend, Divisor divisor) noexcept
{
    return !is_negative(dividend, divisor) || no_remainder(dividend, divisor);
}
```
These templates implement the three common rounding methods.
```cpp
#include <type_traits>

template <typename Dividend, typename Divisor,
    if_integer<Dividend> = true, if_integer<Divisor> = true>
constexpr auto ceilinged_divide(Dividend dividend, Divisor divisor) noexcept
    -> decltype(dividend / divisor)
{
    return truncated_divide(dividend, divisor) + 
        (is_ceilinged(dividend, divisor) ? 0 : 1);
}

template <typename Dividend, typename Divisor,
    if_integer<Dividend> = true, if_integer<Divisor> = true>
constexpr auto ceilinged_modulo(Dividend dividend, Divisor divisor) noexcept
    -> typename std::make_signed<decltype(dividend % divisor)>::type
{
    return truncated_modulo(dividend, divisor) -
        (is_ceilinged(dividend, divisor) ? 0 : divisor);
}

template <typename Dividend, typename Divisor,
    if_integer<Dividend> = true, if_integer<Divisor> = true>
constexpr auto floored_divide(Dividend dividend, Divisor divisor) noexcept
    -> decltype(dividend / divisor)
{
    return truncated_divide(dividend, divisor) -
        (is_floored(dividend, divisor) ? 0 : 1);
}

template <typename Dividend, typename Divisor,
    if_integer<Dividend> = true, if_integer<Divisor> = true>
constexpr auto floored_modulo(Dividend dividend, Divisor divisor) noexcept
    -> decltype(dividend % divisor)
{
    return truncated_modulo(dividend, divisor) +
        (is_floored(dividend, divisor) ? 0 : divisor);
}

template <typename Dividend, typename Divisor,
    if_integer<Dividend> = true, if_integer<Divisor> = true>
constexpr auto truncated_divide(Dividend dividend, Divisor divisor) noexcept
    -> decltype(dividend / divisor)
{
    return dividend / divisor;
}

template <typename Dividend, typename Divisor,
    if_integer<Dividend> = true, if_integer<Divisor> = true>
constexpr auto truncated_modulo(Dividend dividend, Divisor divisor) noexcept
    -> decltype(dividend % divisor)
{
    return dividend % divisor;
}
```

## Optimization
The use of `std::signbit` is avoided as [it casts](https://en.cppreference.com/w/cpp/numeric/math/signbit) to `double`, though otherwise would be sufficient to replace the `is_negative<Integer>` templates.

The `%` operator may be invoked twice in `floored_modulo` and `ceilinged_modulo`. The first tests for remainder and the second produces it. It does not seem worth denormalizing the implementation by adding a variable to cache the value in the (sometimes) case where there is a non-zero remainder. So in these cases I am relying on CPU cache and/or compiler optimization to avoid remainder recomputation.

The `constexpr` keyword ensures that the functions can be evaluated at compile time and advises the compiler that inlining of the functions is preferred. Generally I prefer to let the compiler make these decisions, preserving code readability.

Despite the relative verbosity of the templates the result should be as optimal as manually inlining the minimal *necessary* operations. A few runs through an NDEBUG build in a debugger confirm this.

## Template Type Constraints
Given that the C++ operators determine the result type (based on the operand types) the return type must be so determined. This is achieved by using the `decltype` keyword.

Without the template overloads there would be warnings on unsigned operands, as they all invoke `factor < 0`, which is always `false` (tautological). These also bypass unnecessary conditions at compile time. For functions or operand combinations that are not referenced, the corresponding templates are not even compiled.

Template specialization could be further employed to reduce a couple calls when both parameters are unsigned. However there is little to no actual performance optimization and the denormalization didn't seem like a worthwhile compromise.

The templates can be factored into header (.hpp) and implementation (.ipp) files, just be sure to remove the default template parameter values in the implementation.

See [Type Constraints Unraveled](Type-Constraints-Unraveled) for an explanation of the template type constraints used above.

## Mixing Unsigned and Signed Operands
It is an objective is to reproduce native operand behavior, changing only the rounding. The native operators allow mixed sign types, although compilers warn that the signed operand will be converted to unsigned. All division and modulo operations are executed in the original data type against the native operators. The consequence is that when mixing signed and unsigned *type* operands, the operation is unsigned.

The quotient sign is always consistent with native operations. Only a signed operand can cause a negative quotient, and in this case the deduced quotient type is signed. Notice that an unsigned deduced quotient type always corresponds to a positive logical result.

| operation | rounding  | dividend | divisor  | quotient  | sign  |
|-----------|-----------|----------|----------|-----------|-------|
| divide    | truncated | signed   | signed   | signed    |  +/-  |
| divide    | truncated | unsigned | signed   | signed    |  +/-  |
| divide    | truncated | signed   | signed   | signed    |  +/-  |
| divide    | truncated | unsigned | unsigned | unsigned  |   +   |
| divide    | floored   | signed   | signed   | signed    |  +/-  |
| divide    | floored   | unsigned | signed   | signed    |  +/-  |
| divide    | floored   | signed   | signed   | signed    |  +/-  |
| divide    | floored   | unsigned | unsigned | unsigned  |  +    |
| divide    | ceilinged | signed   | signed   | signed    |  +/-  |
| divide    | ceilinged | unsigned | signed   | signed    |  +/-  |
| divide    | ceilinged | signed   | signed   | signed    |  +/-  |
| divide    | ceilinged | unsigned | unsigned | unsigned  |   +   |

However, a ceilinged modulo of unsigned operands produces a logically a negative remainder, but in this case the deduced remainder type is unsigned.

| operation | rounding  | dividend | divisor  | remainder | sign  |
|-----------|-----------|----------|----------|-----------|-------|
| modulo    | truncated | signed   | signed   | signed    |  +/-  |
| modulo    | truncated | unsigned | signed   | signed    |  +/-  |
| modulo    | truncated | signed   | signed   | signed    |  +/-  |
| modulo    | truncated | unsigned | unsigned | unsigned  |   +   |
| modulo    | floored   | signed   | signed   | signed    |  +/-  |
| modulo    | floored   | unsigned | signed   | signed    |  +/-  |
| modulo    | floored   | signed   | signed   | signed    |  +/-  |
| modulo    | floored   | unsigned | unsigned | unsigned  |  +    |
| modulo    | ceilinged | signed   | signed   | signed    |  +/-  |
| modulo    | ceilinged | unsigned | signed   | signed    |  +/-  |
| modulo    | ceilinged | signed   | signed   | signed    |  +/-  |
| modulo    | ceilinged | unsigned | unsigned | unsigned  |   -   |

The consequence would be a [two's complement](https://en.wikipedia.org/wiki/Two%27s_complement) representation of the logically-negative remainder. In all cases of calling `ceilinged_modulo` with **unsigned** operands the positive unsigned remainder would have to be interpreted as logically **negative**. For this reason the return type on this operation is converted to the signed type of the same size, using `std::make_signed`.

## Conclusion
* Behavior satisfies the identity function for all sign combinations.
* Behavior is consistent with native operators.
* Return type is consistent with native operators.
* The functions cannot *cause* overflows.
* There can be no "tautological compare" warnings from unsigned parameters.
* The functions fail as native operators with a zero-valued divisor.
* The stack calls are removed by inlining.
* The `is_negative` calls compile away for `unsigned` operands.
* The `no_remainder` and `is_negative` calls compile away for `constexpr` operands.
* The `||` conditions compile away when the above render them tautological.
* All that remains is *necessary*.

## Test Vectors
These expressions demonstrate and prove the correctness of the algorithm above. They may be useful in generating a test matrix.
```cpp
// If the quotient x/y is representable in the type of the result:
// Identity: (x / y) * y + (x % y) = x

// Divide increment magnitude.
constexpr auto i = 1;

// Operand magnitudes.
constexpr auto x = 4;
constexpr auto y = 3;

// Zero to zero.
static_assert(0 / -y == 0, "0");
static_assert(0 / +y == 0, "0");
static_assert(0 % -y == 0, "0");
static_assert(0 % +y == 0, "0");

// ----------------------------------------------------------------------------

// Truncated divide:
static_assert(+x / +y == +1, "+");
static_assert(-x / -y == +1, "+");
static_assert(-x / +y == -1, "-");
static_assert(+x / -y == -1, "-");

// Truncated modulo:
static_assert(+x % +y == +1, "+");
static_assert(-x % -y == -1, "+");
static_assert(-x % +y == -1, "-");
static_assert(+x % -y == +1, "-");

// Truncated identity:
static_assert((+x / +y) * (+y) + (+x % +y) == +x, "+");
static_assert((-x / -y) * (-y) + (-x % -y) == -x, "+");
static_assert((-x / +y) * (+y) + (-x % +y) == -x, "-");
static_assert((+x / -y) * (-y) + (+x % -y) == +x, "-");

// ----------------------------------------------------------------------------

// Ceilinged divide: (increment truncated quotient if positive and remainder)
static_assert((+x / +y) + (((+x < 0) == (+y < 0)) ? ((+x % +y) != 0 ? +i : 0) : 0) == +1 + i, "+");
static_assert((-x / -y) + (((-x < 0) == (-y < 0)) ? ((-x % -y) != 0 ? +i : 0) : 0) == +1 + i, "+");
static_assert((-x / +y) + (((-x < 0) == (+y < 0)) ? ((-x % +y) != 0 ? +i : 0) : 0) == -1 + 0, "-");
static_assert((+x / -y) + (((+x < 0) == (-y < 0)) ? ((+x % -y) != 0 ? +i : 0) : 0) == -1 + 0, "-");

// Ceilinged modulo: (decrease truncated modulo by divisor if positive and remainder)
static_assert((+x % +y) - (((+x < 0) == (+y < 0)) ? +y : 0) == +1 - +y, "+");
static_assert((-x % -y) - (((-x < 0) == (-y < 0)) ? -y : 0) == -1 - -y, "+");
static_assert((-x % +y) - (((-x < 0) == (+y < 0)) ? +y : 0) == -1 - +0, "-");
static_assert((+x % -y) - (((+x < 0) == (-y < 0)) ? -y : 0) == +1 - -0, "-");

// Ceilinged identity
static_assert((+1 + i) * (+y) + (+1 - +y) == +x, "+");
static_assert((+1 + i) * (-y) + (-1 - -y) == -x, "+");
static_assert((-1 + 0) * (+y) + (-1 - +0) == -x, "-");
static_assert((-1 + 0) * (-y) + (+1 - -0) == +x, "-");

// ----------------------------------------------------------------------------

// Floored divide: (decrement truncated quotient if negative and remainder)
static_assert((+x / +y) - (((+x < 0) != (+y < 0)) ? ((+x % +y) != 0 ? +i : 0) : 0) == +1 - 0, "+");
static_assert((-x / -y) - (((-x < 0) != (-y < 0)) ? ((-x % -y) != 0 ? +i : 0) : 0) == +1 - 0, "+");
static_assert((-x / +y) - (((-x < 0) != (+y < 0)) ? ((-x % +y) != 0 ? +i : 0) : 0) == -1 - i, "-");
static_assert((+x / -y) - (((+x < 0) != (-y < 0)) ? ((+x % -y) != 0 ? +i : 0) : 0) == -1 - i, "-");

// Floored modulo: (increase truncated modulo by divisor if negative and remainder)
static_assert((+x % +y) + (((+x < 0) != (+y < 0)) ? +y : 0) == +1 + +0, "+");
static_assert((-x % -y) + (((-x < 0) != (-y < 0)) ? -y : 0) == -1 + +0, "+");
static_assert((-x % +y) + (((-x < 0) != (+y < 0)) ? +y : 0) == -1 + +y, "-");
static_assert((+x % -y) + (((+x < 0) != (-y < 0)) ? -y : 0) == +1 + -y, "-");

// Floored identity:
static_assert((+1 - 0) * (+y) + (+1 + +0) == +x, "+");
static_assert((+1 - 0) * (-y) + (-1 + +0) == -x, "+");
static_assert((-1 - i) * (+y) + (-1 + +y) == -x, "-");
static_assert((-1 - i) * (-y) + (+1 + -y) == +x, "-");
```