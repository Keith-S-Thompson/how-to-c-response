# A critique of ["How to C in 2016"](https://matt.sh/howto-c) by Matt

## Keith Thompson, Sat 2016-01-09, updated Fri 2016-01-15

Matt (whose web site does not mention his
last name as far as I can tell) has written an
article "How to C in 2016".  It's been linked to from
[Reddit](https://www.reddit.com/r/programming/comments/400v0b/how_to_c_as_of_2016/)
and from [Hacker News](https://news.ycombinator.com/item?id=10864176);
the latter is where I saw it.

Update: Matt has been kind enough to add a link to this critique to
his article.

Update: A couple of people have found Matt's last name from other
sites, but since he didn't choose to include it in his article I
won't mention it here.

Just to avoid any possible confusion, I am not
[Ken Thompson](https://en.wikipedia.org/wiki/Ken_Thompson),
nor am I related to him.

This article has been linked from
[https://reddit.com/r/programming/](https://reddit.com/r/programming/),
and the link is currently at the top of the page.  Lots of comments
there, some of them even constructive.

As is inevitable for anything expressing opinions about the C language,
there are some things I disagree with.  This critique is intended
to be *constructive* criticism.  It's entirely possible that in some
cases Matt is right and I'm wrong.

I haven't quoted Matt's entire article.  In particular, I've omitted
some material that I agree with.  I suggest reading this critique in
parallel with Matt's article.

> *The first rule of C is don't write C if you can avoid it.*

I don't agree, but it's too broad a topic to discuss meaningfully.

> *C99 is the default C implementation for clang, no extra options needed.*

That depends on the version of clang.  clang 3.5 defaults to C99.
clang 3.6 defaults to C11.  I'm not sure how strict it is by default.

If you want to use a particular standard for either gcc or clang,
be explicit: use `-std=cNN -pedantic`.

> *gcc-5 defaults to `-std=gnu11`, but you should still specify a
> non-GNU c99 or c11 for practical usage.*

Unless you want to use gcc-specific extensions, which is a perfectly
legitimate thing to do.

> *If you find yourself typing `char` or `int` or `short` or `long`
> or `unsigned` into new code, you're doing it wrong.*

Sorry, this is nonsense.  `int` in particular is going to be the most
"natural" integer type for the current platform.  If you want signed
integers that are reasonably fast and are at least 16 bits, there's
nothing wrong with using `int`.  (Or you can use `int_least16_t`,
which may well be the same type, but IMHO that's more verbose than
it needs to be.)

> *For modern programs, you should `#include <stdint.h>` then use standard types.*

The fact that `int` doesn't have "std" in its name doesn't make it
non-standard.  Types such as `int`, `long`, et al are built into the
language.  The `typedef`s defined in `<stdint.h>` are later add-ons.
That doesn't make them less "standard" than the predefined types, but
they're certainly no more standard.

> *`float` — standard 32-bit floating point*

> *`double` - standard 64-bit floating point*

`float` and `double` are very commonly IEEE 32-bit and 64-bit
floating-point types, particularly on modern systems, but there's
no guarantee of that in the language.  I've worked on systems where
`float` is 64 bits.

> *Notice we don't have `char` anymore. `char` is actually misnamed
> and misused in C.*

C's conflation of characters and bytes is unfortunate, but we're
stuck with it.  The type `char` is guaranteed to be exactly one byte,
where a "byte" is at least 8 bits.

> *Developers routinely abuse `char` to mean "byte" even when they are
> doing unsigned byte manipulations. It's much cleaner to use `uint8_t`
> to mean single a unsigned-byte/octet-value and `uint8_t *` to mean
> sequence-of-unsigned-byte/octet-values.*

If you want *bytes*, use `unsigned char`.  If you want *octets*, use
`uint8_t`.  If `CHAR_BIT > 8` then `uint8_t` won't exist, and your
code won't compile (which is probably what you want).  If you want
something that's *at least* 8 bits, use `uint_least8_t`.  If you want
to assume that bytes are octets, add something like this to your code:

    #include <limits.h>
    #if CHAR_BIT != 8
        #error "This program assumes 8-bit bytes"
    #endif

Note that POSIX requires `CHAR_BIT == 8`.

>  *the C type of string literals (`"hello"`) is `char *`.*

No, the type of string literals is `char[]`.  In particular, the type
of `"hello"` is `char[6]`.  **Arrays are not pointers**.  See section 6
of the [comp.lang.c FAQ](http://www.c-faq.com/) for more on this topic.

> *At no point should you be typing the word `unsigned` into your code. We
> can now write code without the ugly C convention of multi-word types
> that impair readability as well as usage.*

Many C types have multi-word names.  There's nothing wrong with that.
Saving keystrokes is not a good reason to use terse abbreviations.

> Who wants to type `unsigned long long int` when you can type `uint64_t`?

For one thing, you can use `unsigned long long`; the `int` is implied.
For another, they mean different things.  `unsigned long long` is *at
least* 64 bits, and may or may not have padding bits.  `uint64_t` is
*exactly* 64 bits, has no padding bits, and is not guaranteed to exist.

And `unsigned long long` is a predefined C type.  Any C programmer
will recognize it.

You can also use `uint_least64_t` -- which may or may not be the same
type as `unsigned long long`.

> *The `<stdint.h>` types are more explicit, more exact in meaning,
> convey intentions better, and are more compact for typographic usage
> and readability.*

Yes, the `intN_t` and `uintN_t` types are more explicit.  *That's not
necessarily a good thing.*  Don't specify things that you don't care
about.  Use `uint64_t` only if you really need *exactly* 64 bits,
no more, no less.

Sometimes you do need a type with a particular exact width, for
example when you need to conform to some externally imposed format.
(Sometimes you also need a particular endianness, alignment, and so
forth; C's `<stdint.h>` doesn't let you specify those.)  But more
often all you need is a particular *range* of values.  For that,
you can use either the `[u]int_leastN_t` or `[u]int_leastN_t` types,
or one of the predefined types.

> *The correct type for pointer math is `uintptr_t` defined in `<stddef.h>`.*

This is dangerously wrong.

First off a minor point: `uintptr_t` is defined in `<stdint.h>`, not
`<stddef.h>`.

That's assuming it's defined at all.  An implementation where `void*`
can't be converted to any integer type without loss of information
won't define `uintptr_t`.  (Such implementations are admittedly rare,
perhaps nonexistent.)

> *Instead of:*

            long diff = (long)ptrOld - (long)ptrNew;

Yes, that's bad.

> *Use:*

            ptrdiff_t diff = (uintptr_t)ptrOld - (uintptr_t)ptrNew;

That's no better.

If you want the difference in terms of the type they point to, use:

    ptrdiff_t diff = ptrOld - ptrNew;

If you want the difference in bytes:

    ptrdiff_t diff = (char*)ptrOld - (char*)ptrNew;

If `ptrOld` and `ptrNew` don't point into, or just past the end of,
the same object, the behavior of the pointer subtraction is undefined.
Converting to `uintptr_t` will give you *some* result -- but that
result won't be useful.  Unless you're writing low-level system code,
don't do *any* pointer comparison or arithmetic unless both pointers
point into or just past the end of the same object.  (Exception:
pointer `==` and `!=` work correctly for pointers to distinct objects.)

> *In these situations, you should use `intptr_t` — the integer type
> defined to be the word size of your current platform.*

No, it isn't.  The concept of "word size" is not well defined.
`intptr_t` is a signed integer type big enough that you can convert a
`void*` to `intptr_t` and back without loss of information.  It might
be *bigger* than `void*`.

> *On 32-bit platforms, `intptr_t` is `int32_t`.*

Probably, but it's not guaranteed.

> *On 64-bit platforms, `intptr_t` is `int64_t`.*

Again, probably, but it's not guaranteed.

> *`size_t` is defined as "an integer capable of holding the largest array
> index"*

No, it isn't.

> *which also means it's capable of holding the largest memory
> index" which also means it's capable of holding the largest memory
> offset in your program.*

It's capable of holding the size of the largest object your
implementation supports.  (There's an argument that that's not
*necessarily* guaranteed, but for practical purposes you can rely
on it.)  It can hold the largest memory offset if all offsets are
within a single object.

> *In either case: `size_t` is practically defined to be the same as
> `uintptr_t` on all modern platforms, so on a 32-bit platform `size_t`
> is `uint32_t` and on a 64-bit platform `size_t` is `uint64_t`.*

Likely, but not guaranteed.

More to the point, `size_t` can represent the size of any *single*
object, while `uintptr_t` can represent *any* pointer value, which
means it can distinguish between the addresses of any byte of any
object.  Most modern systems have monolithic address spaces, so the
theoretical maximum size of an object is the same as the size of the
entire memory space -- but the C standard very carefully does not
require this.  You could have a 64-bit system where no object can be
bigger than 32 bits, for example.

By emphasizing "modern" systems, you ignore both old systems (like
the old x86 systems that exposed segmented addressing with "near" and
"far" pointers) and possible future systems that might be perfectly
compatible with the C standard while violating "modern" assumptions.

> *You should never cast types during printing. You should use proper
> type specifiers.*

That's one way to do it, but it's not the only good approach.
(And even you agree that you need to cast to `void*` for `"%p"`.)

> *raw pointer value - `%p` (prints hex value; cast your pointer to
> `(void *)` first)*

Good advice -- but the output format is implementation-defined.
It's *usually* hex, but don't assume that.

            printf("Local number: %" PRIdPTR "\n\n", someIntPtr);

The name `someIntPtr` implies a type of `int*`, but in fact it's of
type `intptr_t`.

There's an alternative which means you don't have to remember the
alphabet soup of macro names:

    some_signed_type n;
    some_unsigned_type u;
    printf("n = %jd, u = %ju\n", (intmax_t)n, (uintmax_t)u);

`intmax_t` and `uintmax_t` are typically 64 bits.  The conversions
are going to be far cheaper than the physical I/O.

> *Notice you put the `'%'` inside your format string, but the type
> specifier is outside your format string.*

It's all part of the format string.  The macros are defined as string
literals, which are concatenated with adjacent string literals.

> *Modern compilers support `#pragma once`*

That doesn't mean you should use it.  Even the GNU cpp manual doesn't
recommend it.  The section on "Once-Only Headers" doesn't even mention
`#pragma once`; it discusses the `#ifndef` idiom.  The following
section, "Alternatives to Wrapper `#ifndef`", briefly mentions
`#pragma once` but points out that it's not portable.

> *This pragma is widely supported across all compilers across all
> platforms and is recommended over manually naming header guards.*

Recommended by whom?  The `#ifndef` trick isn't pretty, but it's
reliable and portable.

> <i>**IMPORTANT NOTE:** If your struct has padding, the `{0}` method
> does not zero out extra padding bytes. For example, `struct thing`
> has 4 bytes of padding after counter (on a 64-bit platform) because
> structs are padded to word-sized increments. If you need to zero out
> an entire struct including unused padding, use `memset(&localThing,
> 0, sizeof(localThing))` because `sizeof(localThing) == 16 bytes` even
> though the addressable contents is only `8 + 4 = 12` bytes.</i>

This gets a bit tricky.  Usually there's no reason to care about
padding bytes.  If you do, then yes, `memset` is the way to zero them.
But zeroing a structure with `memset`, though it will set any integer
members to zero, is not guaranteed to set floating-point members to
`0.0` or pointers to `NULL` (though it will on most systems).

> *C99 allows variable length array initializers*

No, C99 doesn't allow *initializers* for VLAs (variable length arrays).
But Matt isn't actually talking about VLA initializers, just about
VLAs.

VLAs are controversial.  Unlike `malloc`, VLAs provide no mechanism
for detecting allocation failures.  So if you need to allocate `N`
bytes of data, then this:

    {
        unsigned char *buf = malloc(N);
        if (buf == NULL) { /* allocation failed */ }
        /* ... */
        free(buf);
    }

is at least in principle safer than this:

    {
        unsigned char buf[N];
        /* ... */
    }


Certainly VLAs are dangerous when used incorrectly.  The same is true
of just about every feature in every language.

But old-fashioned fixed-length arrays have exactly the same problem.
As long as you check the size before creating the array, a VLA of
some variable size N is no more dangerous than a fixed-length array of
the same size.  And it's common to define a fixed-length array that's
bigger than it needs to be to ensure that you use *part* of it to store
your actual data.  With a VLA, you can allocate just what you need.
I agree with Matt's advice here.

Except for one thing: C11 made VLAs optional.  I doubt that many C11
compilers will actually decide not to implement them, except perhaps
for small embedded systems, but it's something to keep in mind if
you want your code to be as portable as possible.

> *If a function accepts **arbitrary** input data and a length to
> process, don't restrict the type of the parameter.*
> 
> *So, do NOT do this:*

            void processAddBytesOverflow(uint8_t *bytes, uint32_t len)

> *Do THIS instead:*

            void processAddBytesOverflow(void *input, uint32_t len) 

(I've omitted the bodies of the functions.)

I agree, `void*` is the right type to use for a parameter pointing
to an arbitrary chunk of memory.  See the `mem*` functions in the
standard library.  (But `len` should be `size_t`, not `uint32_t`.)

> *By declaring your input type as `void *` and re-casting inside your
> function, you save the users of your function from having to think
> about abstractions inside your own library.*

A small quibble: There's no cast in Matt's function.  There's an
implicit conversion from `void*` to `uint8_t*`.

> *Some readers have pointed out alignment problems with this example.*

Some readers are mistaken.  Accessing a chunk of memory as a sequence
of bytes is always safe.

> *C99 gives us the power of `<stdbool.h>` which defines `true` to `1`
> and `false` to `0`.*

Yes, and it also defines `bool` as an alias for the predefined Boolean
type `_Bool`.

> *For success/failure return values, functions should return `true` or
> `false`, not an `int32_t` return type with manually specifying `1` and `0`
> (or worse, `1` and `-1` (or is it `0` success and `1` failure? or is it `0`
> success and `-1` failure?)).*

There is a widespread convention, particularly in Unix-like systems,
for functions to return `0` for success and some non-zero value
(often `-1`) for failure.  In many cases different non-zero results
denote different kinds of failure.  It's important to follow this
convention when adding new functions to such an interface.  (`0`
is used for success because typically there's only one way for a
function to succeed, and multiple ways for it to fail.)

A function that tests whether some condition is true or false
should return `true` or `false`.  But success vs. failure is often
a different thing.

A `bool` function should have a name that's a *predicate*.  In English,
it would be something that answers a yes/no question.  Examples are
`is_foo()` and `has_widget()`.  A function that tries to *do*
something, and that needs to let you know whether it succeeded or
not, should probably use a different convention.  In some languages
raising or throwing an exception is the right approach.  In C, you
should follow some existing convention -- and zero for success is
the most common one.

> *The only usable C formatter as of 2016 is
> [clang-format](http://clang.llvm.org/docs/ClangFormat.html).
> clang-format has the best defaults of any automatic C formatter and
> is still actively developed.*

I haven't used clang-format myself.  I'll have to look into it.

I have my own fairly strong opinions about C code layout:

- Opening brace goes at the end of the line;
- Spaces, not tabs;
- 4-columns per level;
- Always use curly braces (except in rare cases where putting a statement
  on one line improves readability).

These are just my own personal preferences, which can be overridden
by one important rule:

- **Follow the conventions of the project you're working on.**

I don't often use automatic formatting tools myself.  Perhaps I should.

> *Never use `malloc`*
>
> *You should always use `calloc`.*

I disagree.  Initializing allocated memory to all-bits-zero is
somewhat arbitrary, and it's typically not a meaningful value.
If you write your code correctly, you won't access any object unless
you have first assigned a meaningful value to it.  Using `calloc`
means that a bug in your code will give you a value of zero, rather
than some arbitrary garbage.  This isn't necessarily an improvement.

Zeroing memory often means that buggy code will have *consistent*
behavior; by definition it will not have *correct* behavior.  And
consistently incorrect behavior can be more difficult to track down.

Of course there's no such thing as bug-free code.  But if you're trying
to program defensively, you might consider initializing allocated
memory to some value that's known to be *invalid* rather than one
that might be valid.

On the other hand, if all-bits-zero happens to be a reasonable initial
value, `calloc` might be a good approach.
