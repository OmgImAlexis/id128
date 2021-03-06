# Id128

Generate 128-bit unique identifiers for various specifications.  In particular:
- [ULID](#ulid)
- [Monotonic ULID](#ulidmonotonic)
- [UUID 1 (Variant 1 Version 1)](#uuid1)
- [UUID 4 (Variant 1 Version 4)](#uuid4)
- [UUID 6 (Variant 1 Version 6)](#uuid6)
- [Nil UUID (Variant 0 Version 0)](#uuidnil)
- [Uuid (Unknown Variant and Version)](#uuid)

# Common Usage

```es6
const {
	Ulid,
	UlidMonotonic,
	Uuid,
	Uuid1,
	Uuid4,
	Uuid6,
	UuidNil,
} = require('id128');

// Id factories
[
	Ulid,
	UlidMonotonic,
	Uuid1,
	Uuid4,
	Uuid6,
	UuidNil,
].forEach((IdType) => {
	// Identify the factory
	console.log(IdType.name);

	// Generate a new id
	const id = IdType.generate();

	// Get the smallest valid id
	const min = IdType.MIN();

	// Get the largest valid id
	const max = IdType.MAX();

	// Compare ids
	console.log(id.equal(id));
	console.log(! id.equal(min));
	console.log(! id.equal(max));
	console.log(id.compare(min) === 1);
	console.log(id.compare(id) === 0);
	console.log(id.compare(max) === -1);

	// Encode the id in its canonical form
	const canonical = id.toCanonical();
	console.log(canonical);

	// Encode the id for efficient db storage
	const raw = id.toRaw();
	console.log(raw);

	// Decode a valid canonically formatted id
	console.log(id.equal(IdType.fromCanonical(canonical)));

	// Decode a canonically formatted id, skipping validation
	console.log(id.equal(IdType.fromCanonicalTrusted(canonical)));

	// Decode a valid raw formatted id
	console.log(id.equal(IdType.fromRaw(raw)));

	// Decode a raw formatted id, skipping validation
	console.log(id.equal(IdType.fromRawTrusted(raw)));
});

// Uuid Factory
[0, 1, 4, 6].forEach((version) => {
	// Generate a new id
	const id = Uuid.generate({ version });

	// Get the smallest valid id
	const min = Uuid.MIN({ version });

	// Get the largest valid id
	const max = Uuid.MAX({ version });

	// Encode the id in its canonical form
	const canonical = id.toCanonical();
	console.log(canonical);

	// Encode the id for efficient db storage
	const raw = id.toRaw();
	console.log(raw);

	// Decode a valid canonically formatted id
	console.log(id.equal(Uuid.fromCanonical(canonical)));

	// Decode a canonically formatted id, skipping validation
	console.log(id.equal(Uuid.fromCanonicalTrusted(canonical)));

	// Decode a valid raw formatted id
	console.log(id.equal(Uuid.fromRaw(raw)));

	// Decode a raw formatted id, skipping validation
	console.log(id.equal(Uuid.fromRawTrusted(raw)));
});
```
## Common Factory Properties

### name
Return the name of the generated id type.

## Common Factory Methods

### .construct(bytes) => id
Return a new id instance without validating the bytes.

### .generate() => id
Return a new id instance.

### .MIN() => id
Return the id instance with the smallest valid value.

### .MAX() => id
Return the id instance with the largest valid value.

### .fromCanonical(canonical_string) => id
Decode an id from its canonical representation.
Throw `InvalidEncoding` if the string is undecodable.

### .fromCanonicalTrusted(canonical_string) => id
Decode an id from its canonical representation.
Skip validation and assume the input is decodable.

### .fromRaw(raw_string) => id
Decode an id from its raw representation.
Throw `InvalidEncoding` if the string is undecodable.

### .fromRawTrusted(raw_string) => id
Decode an id from its raw representation.
Skip validation and assume the input is decodable.

### .toCanonical(id) => canonical_string
Encode the given id in the canonical form.
Throw `InvalidBytes` if the id is not 128-bit conformant.

### .toRaw(id) => raw_string
Encode the given id in the raw form.
Throw `InvalidBytes` if the id is not 128-bit conformant.

### .compare(left_id, right_id) => (-1|0|1)
Determine if the left id is `less than | equal to | greater than`
the right id using lexicographical byte order.

### .equal(left_id, right_id) => (true|false)
Determine if 2 ids have the same byte value.

## Common Instance Properties

### bytes
Return the actual byte array representing the id.

## Common Instance Methods

### .clone() => deep_copy
Return a new instance of the id with the same bit signature.

### .compare(other) => (-1|0|1)
Determine how this id is ordered against another.

### .equal(other) => (true|false)
Determine if this id has the same bytes as another.

### .toCanonical() => canonical_string
Encode this id in its canonical form.

### .toRaw() => raw_string
Encode this id in its raw form.

# Ulid
```es6
const { Ulid } = require('id128');
```

Ulid, as [specified](https://github.com/ulid/spec), has some nice properties:
- collision resistant: 80-bits of randomness
- k-ordered: prefixed with millisecond precision timestamp
- database friendly: fits within a uuid and generally appends to the index
- human friendly: canonically encodes as a case-insensitive Crockford 32 number

It is useful when you need a distributed domain unique id.

## Additional Instance Properties

### time
Return a Date object for the epoch milliseconds encoded in the id.

## Additional Factory Methods

### .generate({ time }) => id
Return a new id instance.  Set any argument to `null` or `undefined` to trigger
its default behavior.

`time` defaults to the current time.  It can be given either as a `Date` object
or epoch milliseconds (milliseconds since January 1st, 1970).
Throw `InvalidEpoch` for times before the epoch or after approximately August 2nd, 10889.
This is provided mostly for unit tests.

## Byte Format
Format `ttttttrrrrrrrrrr` where:
- `t` is 8 bits of time
- `r` is 8 bits of random

# UlidMonotonic
```es6
const { UlidMonotonic } = require('id128');
```

UlidMonotonic is inspired by the [specification](https://github.com/ulid/spec#monotonicity):
- collision resistant: 15-bits of random seeded clock sequence plus 64-bits of randomness
- total ordered: prefixed with millisecond precision timestamp plus 15-bit clock sequence
- database friendly: fits within a uuid and generally appends to the index
- human friendly: canonically encodes as a case-insensitive Crockford 32 number

It is useful when you need to guarantee a process unique id.

## Additional Instance Properties

### time
Return a Date object for the epoch milliseconds encoded in the id.

## Additional Factory Methods

### .generate({ time }) => id
Return a new id instance.  Set any argument to `null` or `undefined` to trigger
its default behavior.

`time` defaults to the current time.  It can be given either as a `Date` object
or epoch milliseconds (milliseconds since January 1st, 1970).  Extra caution is
required since setting a future time and subsequently calling `generate`
guarantees usage of the clock sequence.
Throw `InvalidEpoch` for times before the epoch or after approximately August 2nd, 10889.
Throw `ClockSequenceOverflow` when the clock sequence is exhausted.
This is provided mostly for unit tests.

### .reset()
Return the clock sequence to its starting position.  This is provided mostly for
unit tests.

## Byte Format
Format `ttttttccrrrrrrrr` where:
- `t` is 8 bits of time
- `c` is 8 bits of random-seeded clock sequence
- `r` is 8 bits of random

More specifically, the clock sequence is a counter.  When the first id for a new
timestamp is generated, the clock sequence is seeded with random bits and the
left-most clock sequence bit is set to 0, reserving 2^15 clock ticks.  Whenever
a time from the past (now is so ephemeral) seeds the generator, the previous id's
time and clock sequence is used instead, incremented by 1.  This guarantees strict
local monotonicity and preserves lexical ordering and general randomness.

Given UlidMonotonic currently generates approximately 850 ids per millisecond,
the clock sequence should never overflow.  This also means the left most bit of
the clock sequence will rarely be set to 1.  However, in the unlikely event of
an overflow, id generation should be aborted.

# Uuid1
```es6
const { Uuid1 } = require('id128');
```

Uuid1 implements the [RFC 4122 time specification](https://tools.ietf.org/html/rfc4122#section-4.2):
- time-based: encodes the current millisecond timestamp
- location-based: encodes the mac address of the machine

While this mostly adheres to the spec, there are a few nuances in the handling
of time.  Instead of encoding time as 100-nanoseconds since the Gregorian epoch,
48 bits encode milliseconds since the Gregorian epoch time and 12 bits count past
time collisions, resetting whenever given a new future time.  There are a few benefits:
- high precision time is unreliable in the browser so this ensures better precision
- the max supported date is now around the year 10502 instead of around 5236
- generating 4096 ids/ms (~4,000,000 ids/s) is wildly unlikely in real world uses
- in the rare  hi-res overflow, the count simply spills over to the clock sequence

## Additional Instance Properties

### clock_sequence
Return the clock sequence encoded in the id.

### hires_time
Return the number of prior ids generated while time stood still.

### node
Return the MAC address encoded in the id.

### time
Return a Date object for the epoch milliseconds encoded in the id.

### variant
Return the variant as encoded in the id.  Should be 1.

### version
Return the version as encoded in the id.  Should be 4.

## Additional Factory Methods

### .generate({ node, time }) => id
Return a new id instance.  Set any argument to `null` or `undefined` to trigger
its default behavior.

`time` defaults to the current time.  It can be given either as a `Date` object
or Gregorian milliseconds (milliseconds since October 15th, 1582).  Extra caution
is required since setting a future time and subsequently calling `generate`
guarantees usage of the hi-res counter and clock sequence.
Throw `InvalidEpoch` for times before the Gregorian epoch or after approximately May 17, 10502.
This is provided mostly for unit tests.

`node` defaults to the MAC address, or a random multicast address when the MAC
address is unavilable.  It can be given as an array of 6 bytes.

### .reset()
Return the hi-res counter to its starting position and generate a new random
clock sequence seed.  This is provided mostly for unit tests.

## Byte Format
Format `llll lnnn mmmm vhhh tccc aaaa aaaa aaaa` where:
- `l` is 4 bits of low millisecond time
- `n` is 4 bits of hi-res time
- `m` is 4 bits of mid millisecond time
- `v` is 4 bits of the version
- `h` is 4 bits of high millisecond time
- `t` is 2 bits of the variant followed by 2 bits of the clock sequence
- `c` is 4 bits of the clock sequence
- `a` is 4 bits of the machine address

# Uuid4
```es6
const { Uuid4 } = require('id128');
```

Uuid4 implements the [RFC 4122 random uuid specification](https://tools.ietf.org/html/rfc4122#section-4.4):

- 122 random bits
- 2 bits reserved for the variant (1)
- 4 bits reserved for the version (4)

It is useful when you need a well-supported globally unique id.

## Additional Instance Properties

### variant
Return the variant as encoded in the id.  Should be 1.

### version
Return the version as encoded in the id.  Should be 4.

## Byte Format
Format `rrrr rrrr rrrr vrrr trrr rrrr rrrr rrrr` where:
- `r` is 4 bits of random
- `v` is 4 bits of the version
- `t` is 2 bits of the variant followed by 2 bits of random

# Uuid6
```es6
const { Uuid6 } = require('id128');
```

Uuid6 implements this [controversial blog post](https://bradleypeabody.github.io/uuidv6/):
- time-based: encodes the current millisecond timestamp
- location-based: encodes the mac address of the machine

This is essentially the same implementation as Uuid1, however the time bits are
arranged in lexicographical order.  If you're looking for a spacial UUID that
is optimized for clustered indices, consider Uuid6 as a viable option.

## Additional Instance Properties

### clock_sequence
Return the clock sequence encoded in the id.

### hires_time
Return the number of prior ids generated while time stood still.

### node
Return the MAC address encoded in the id.

### time
Return a Date object for the epoch milliseconds encoded in the id.

### variant
Return the variant as encoded in the id.  Should be 1.

### version
Return the version as encoded in the id.  Should be 6.

## Additional Factory Methods

### .generate({ node, time }) => id
Return a new id instance.  Set any argument to `null` or `undefined` to trigger
its default behavior.

`time` defaults to the current time.  It can be given either as a `Date` object
or Gregorian milliseconds (milliseconds since October 15th, 1582).  Extra caution
is required since setting a future time and subsequently calling `generate`
guarantees usage of the hi-res counter and clock sequence.
Throw `InvalidEpoch` for times before the Gregorian epoch or after approximately May 17, 10502.
This is provided mostly for unit tests.

`node` defaults to the MAC address, or a random multicast address when the MAC
address is unavilable.  It can be given as an array of 6 bytes.

### .reset()
Return the hi-res counter to its starting position and generate a new random
clock sequence seed.  This is provided mostly for unit tests.

## Byte Format
Format `mmmm mmmm mmmm vnnn tccc aaaa aaaa aaaa` where:
- `m` is 4 bits of millisecond time
- `n` is 4 bits of hi-res time
- `v` is 4 bits of the version
- `h` is 4 bits of high millisecond time
- `t` is 2 bits of the variant followed by 2 bits of the clock sequence
- `c` is 4 bits of the clock sequence
- `a` is 4 bits of the machine address

# UuidNil
```es6
const { UuidNil } = require('id128');
```

UuidNil implements the [RFC 4122 nil uuid specification](https://tools.ietf.org/html/rfc4122#section-4.1.7):

- 128 bits of glorious 0

It is useful as placeholder for other 128-bit ids.

## Additional Instance Properties

### variant
Return the variant as encoded in the id.  Should be 0.

### version
Return the version as encoded in the id.  Should be 0.

## Byte Format
Format `0000 0000 0000 v000 t000 0000 0000 0000` where:
- `0` is 4 bits of 0
- `v` is 4 bits of the version (also 0)
- `t` is 2 bits of the variant (also 0) followed by 2 bits of 0

# Uuid
```es6
const { Uuid } = require('id128');
```

Uuid is a factory for generating and decoding UUIDs when the version is unknown
until runtime.  If the version is supported, it will produce UUIDs of the
appropriate type.  In exchange for the runtime flexibility, there is a necessary
performance degradation.  It is recommended to use this for decoding data from
uncontrolled sources rather than generating new ids.

Uuid supports all the same methods as the other ID factories.  All modifications
to typical behavior are noted below.

## Factory Properties

### versioned_ids
Return the factories of all the supported ids.

## Factory Methods

### .construct(bytes) => versioned_id
Return a new versioned id instance without validating the bytes.
Return a Uuid if an appropriate version does not exist.

### .generate({ version, ... }) => versioned_id
Return a new versioned id instance.  All additional arguments are passed through
to the associated version.
Throw `UnsupportedVersion` if no associated version exists.

### .MIN({ version }) => versioned_id
Return the versioned id instance with the smallest valid value.
Throw `UnsupportedVersion` if no associated version exists.

### .MAX({ version }) => versioned_id
Return the versioned id instance with the largest valid value.
Throw `UnsupportedVersion` if no associated version exists.

### .fromCanonical(canonical_string) => versioned_id
Decode a versioned id from its canonical representation.
Return a Uuid if an appropriate version does not exist.
Throw `InvalidEncoding` if the string is undecodable.

### .fromCanonicalTrusted(canonical_string) => versioned_id
Decode a versioned id from its canonical representation.
Return a Uuid if an appropriate version does not exist.
Skip validation and assume the input is decodable.

### .fromRaw(raw_string) => versioned_id
Decode a versioned id from its raw representation.
Return a Uuid if an appropriate version does not exist.
Throw `InvalidEncoding` if the string is undecodable.

### .fromRawTrusted(raw_string) => versioned_id
Decode a versioned id from its raw representation.
Return a Uuid if an appropriate version does not exist.
Skip validation and assume the input is decodable.

# Exceptions
```es6
const { Exception } = require('id128');
```
All exceptions are namespaced under `Exception` for clarity.

## Id128Error
```es6
const { Exception: { Id128Error } } = require('id128');
```
Base exception class for generic error catching.

## ClockSequenceOverflow
```es6
const { Exception: { ClockSequenceOverflow } } = require('id128');
```
Incrementing the clock sequence is impossible.  Should not happen unless manually seeding `#generate`.

## InvalidBytes
```es6
const { Exception: { InvalidBytes } } = require('id128');
```
Encoding something other than 16 bytes.  Likely to happen when encoding untrusted user input.

## InvalidEncoding
```es6
const { Exception: { InvalidEncoding } } = require('id128');
```
Decoding an invalid format or non-string object.  Likely to happen when decoding untrusted user input.

## InvalidEpoch
```es6
const { Exception: { InvalidEpoch } } = require('id128');
```
Generating an id with an invalid timestamp.  Should not happen unless manually seeding `#generate`.

## UnsupportedVersion
```es6
const { Exception: { UnsupportedVersion } } = require('id128');
```
Failed to find a factory for the desired version.  Likely to happen when decoding untrusted user input.

# Browser Support

This module supports browser compilation though Webpack/Browserify with a few caveats:
- Random number generation is optimized for memory usage over speed since only a
handful of ids are likely to be generated during a user's session so the overhead
of generating a page of random values has poor amortized cost.
- The browser must have native support for `crypto`.  `Math.random` is far too
insecure to support as a fallback, especially since the fallback only makes sense
for older browsers with proven security holes.  `msCrypto` is not a supported
fallback due to many of the other required features.
- The browser must support:
	* classes
	* closures
	* `const` and `let`
	* `Uint8Array`
	* `Symbol`

This library is intended for modern browsers that keep pace with Javascript's
growing ecosystem.  I philosophically object to supporting efforts of companies
to pour more money into broken browsers that only cause headaches for developers
to support.  I expect these caveats to be unnecessary within the next 5 years.

All that said, please notify me of any issues with modern browsers and I'll do
my best to support you.

# Motivation

Originally, I was looking for an id that is independent of the database, but plays
nice with database indices and data types.  Most databases have built-in support
for storing UUIDs efficiently, but UUID v4 does not cluster well and the other UUIDs
require bit manipulation to get good performance, which will likely cause future
maintenance headaches.

After a bit of research, ULID was determined to nicely solve the problem.  However,
the javascript implementation had 2 major issues:
1. lacks database support
2. it's slow, which in a single-threaded Node server is deadly

I considered sending in a patch, however I saw an opportunity for a more expressive
interface, which is typically a bit harder to modify once a project is in wide use.
There was also a clear pattern for encoding 128-bit ids into various formats,
which seems generally useful.

Ultimately, this library strives to be:
- secure: uses cryptographic randomness to ensure general uniqueness
- performant: currently one of the fastest id generators available
- maintainable: heavily tested isolated code with a consistent interface
- extensible: modular design to easily add new ids and new encodings

# Tests

To run the tests:
```bash
yarn install
yarn test-all
```

# Benchmarks

Competitive benchmarks have been moved to [benchmark-guid](https://github.com/aarondcohen/benchmark-guid)

To run the benchmarks:
```bash
yarn install
yarn benchmark-all
```

```
                 Ulid
  1,769,033 op/s » generate
  6,569,198 op/s » MIN
 11,014,665 op/s » MAX
  1,477,454 op/s » fromCanonical
  1,748,102 op/s » fromCanonicalTrusted
  1,242,933 op/s » fromRaw
  1,536,321 op/s » fromRawTrusted
  2,802,490 op/s » toCanonical
  5,258,532 op/s » toRaw

                 UlidMonotonic
  1,695,117 op/s » generate
  5,967,879 op/s » MIN
  6,137,375 op/s » MAX
  1,322,621 op/s » fromCanonical
  1,555,218 op/s » fromCanonicalTrusted
  1,150,228 op/s » fromRaw
  1,377,673 op/s » fromRawTrusted
  2,603,634 op/s » toCanonical
  4,629,156 op/s » toRaw

                 Uuid1
  3,878,454 op/s » generate
  5,779,313 op/s » MIN
  8,024,808 op/s » MAX
  1,085,930 op/s » fromCanonical
  1,354,619 op/s » fromCanonicalTrusted
  1,165,904 op/s » fromRaw
  1,433,542 op/s » fromRawTrusted
  3,651,304 op/s » toCanonical
  4,537,886 op/s » toRaw

                 Uuid4
  2,175,579 op/s » generate
  4,768,260 op/s » MIN
  5,859,188 op/s » MAX
  1,059,137 op/s » fromCanonical
  1,275,629 op/s » fromCanonicalTrusted
  1,148,766 op/s » fromRaw
  1,403,192 op/s » fromRawTrusted
  4,196,694 op/s » toCanonical
  4,522,759 op/s » toRaw

                 Uuid6
  3,748,749 op/s » generate
  3,878,965 op/s » MIN
  4,811,298 op/s » MAX
  1,089,643 op/s » fromCanonical
  1,299,267 op/s » fromCanonicalTrusted
  1,090,523 op/s » fromRaw
  1,373,687 op/s » fromRawTrusted
  3,894,566 op/s » toCanonical
  4,114,762 op/s » toRaw

                 UuidNil
  8,618,101 op/s » generate
  4,174,545 op/s » MIN
  8,297,900 op/s » MAX
  1,072,836 op/s » fromCanonical
  1,272,640 op/s » fromCanonicalTrusted
  1,247,268 op/s » fromRaw
  1,509,222 op/s » fromRawTrusted
  3,909,988 op/s » toCanonical
  4,105,263 op/s » toRaw

                 Uuid processing Uuid1
  4,008,788 op/s » generate
  3,959,086 op/s » MIN
  4,734,408 op/s » MAX
    953,132 op/s » fromCanonical
  1,111,260 op/s » fromCanonicalTrusted
    990,218 op/s » fromRaw
  1,218,708 op/s » fromRawTrusted

                 Uuid processing Uuid4
  2,088,674 op/s » generate
  3,952,405 op/s » MIN
  4,824,903 op/s » MAX
    972,908 op/s » fromCanonical
  1,107,268 op/s » fromCanonicalTrusted
    988,771 op/s » fromRaw
  1,209,602 op/s » fromRawTrusted

                 Uuid processing Uuid6
  3,988,815 op/s » generate
  4,048,811 op/s » MIN
  4,771,905 op/s » MAX
    983,902 op/s » fromCanonical
  1,123,541 op/s » fromCanonicalTrusted
    991,070 op/s » fromRaw
  1,242,203 op/s » fromRawTrusted

                 Uuid processing UuidNil
  8,144,932 op/s » generate
  3,993,800 op/s » MIN
  8,180,888 op/s » MAX
    960,049 op/s » fromCanonical
  1,091,887 op/s » fromCanonicalTrusted
  1,089,924 op/s » fromRaw
  1,311,902 op/s » fromRawTrusted
```

# Acknowledgments

Much of this library would not exist without the great work and documentation of
other engineers.  In particular:

- [ksuid](https://github.com/segmentio/ksuid): an in-depth exploration of the guid nuances
- [ulid](https://github.com/ulid/javascript): an elegant solution to a persistent problem
- [uuid-random](https://github.com/jchook/uuid-random): allocating pages of randomness is by far the biggest performance factor

# Contributing

Feel free to make a branch and send a pull request through [github](https://github.com/aarondcohen/id128)

# Issues

Please report any issues or bugs through [github](https://github.com/aarondcohen/id128/issues)
