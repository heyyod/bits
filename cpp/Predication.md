Another way to mitigate the effects of branch dependencies is to simply avoid
branching altogether. Consider again the SafeIntegerDivide() function,

although we’ll modify it slightly to work in terms of floating-point values in-
stead of integers:

```c++
float SafeFloatDivide(float a, float b, float d)
{
	return (b != 0.0f) ? a / b : d;
}
```

This simple function calculates one of two answers, depending on the re-
sults of the conditional test b != 0. Instead of using a conditional branch

statement to return one of these two answers, we can instead arrange for our

conditional test to generate a bit mask consisting of all zeros (0x0U) if the con-
dition is false, and all ones (0xFFFFFFFFU) if it is true. We can then execute

both branches, generating two alterate answers. Finally, we use the mask to
produce the final answer that we’ll return from the function.



The following pseudocode illustrates the idea behind predication. (Note
that this code won’t run as-is. In particular, you can’t mask a float with
an unsigned int and obtain a float result—you’d need to use a union to
reinterpret the bit patterns of the floats as if they were unsigned integers
when applying the mask.)

```c++
int SafeFloatDivide_pred(float a, float b, float d)
{
    // convert Boolean (b != 0.0f) into either 1U or 0U
    const unsigned condition = (unsigned)(b != 0.0f);
    // convert 1U -> 0xFFFFFFFFU
    // convert 0U -> 0x00000000U
    const unsigned mask = 0U - condition;
    // calculate quotient (will be QNaN if b == 0.0f)
    const float q = a / b;
    // select quotient when mask is all ones, or default
    // value d when mask is all zeros (NOTE: this won't
    // work as written -- you'd need to use a union to
    // interpret the floats as unsigned for masking)
    const float result = (q & mask) | (d & ~mask);
    return result;
}
```

Let’s take a closer look at how this works:
• The test b != 0.0f produces a bool result. We convert this into an
unsigned integer by simply casting it. This results in either the value
1U (corresponding to true) or 0U (corresponding to false).
• We convert this unsigned result into a bit mask by subtracting it from
0U. Zero minus zero is still zero, and zero minus one is −1, which is
0xFFFFFFFFU in 32-bit unsigned integer arithmetic.

• Next, we go ahead and calculate the quotient. We run this code regard-
less of the result of the non-zero test, thereby side-stepping any branch

dependency issues.
• We now have our two answers ready to go: The quotient q and the
default value d. We want to apply the mask in order to “select” one

or the other value. But to do this, we need to reinterpret the floating-
point bit patterns of q and d as if they were unsigned integers. The most

portable way to accomplish this in C/C++ is to use a union containing

two members, one of which interprets a 32-bit value as a float, the
other of which interprets it as an unsigned.
• The mask is applied as follows: We bitwise AND the quotient q with the
mask, producing a bit pattern that matches q if the mask is all ones, but
is all zeros if the mask is all zeros. We bitwise AND the default value
d with the complement of the mask, yielding all zeros if the mask is all
ones, or the bit pattern of d if the mask is all zeros. Finally, we bitwise
OR these two values together, effectively selecting either the value of q or
the value of d.

The use of a mask to select one of two possible values like this is called pred-
ication because we run both code paths (the one that returns a / b and

the one that returns d) but each code path is predicated on the results of the
test (a != 0), via the mask. Because we are selecting one of two possible
values, this is also often called a select operation.
Going to all this trouble to avoid a branch may seem like overkill. And it

can be—its usefulness depends on the relative cost of a branch versus the pred-
icated alternative on your target hardware. Predication really shines when

a CPU’s ISAs provide special machine language instructions for performing
a select operation. For example, the PowerPC ISA offers an integer select
instruction isel, a floating-point select instruction fsel, and even a SIMD

vector select instruction vecsel, and their use can definitely result in perfor-
mance improvements on PowerPC based platforms (like the PS3).

It’s important to realize that predication only works when both branches
can be executed safely. Performing a divide by zero operation in floating-point
generates a quiet not-a-number (QNaN), but an integer divide by zero throws
an exception that will crash your game (unless it is caught). That’s why we
converted this example to floating-point before applying predication to it.

> From Game Engines Architecture, pg 239