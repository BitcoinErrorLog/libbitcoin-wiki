## Background
Libbitcoin support for mnemonic wallet seed encoding began with [Electrum](https://electrum.org/) v1. Later came [BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki), driven by the fine folks behind [Trezor](https://wiki.trezor.io/), which we believed Electrum was adopting. When the fine folks behind Electrum [decided against BIP39](https://electrum.readthedocs.io/en/latest/seedphrase.html#motivation), we found ourselves with three implementations. We had dropped Electrum v1 in the expectation that BIP39 would become sufficient. Later we added Electrum but found it necessary to also restore Electrum v1. It is not possible to properly implement Electrum mnemonic support without also implementing Electrum v1 and BIP39 mnemonics.

An overhaul of our mnemonic implementations was well overdue. What was anticipated to require one week required over a month of full time work. Before I forget the various lessons learned, I decided to write them down here. The information is all out there, somewhere. But ultimately it required digging through a lot of Python and C code. Wallet seeds are not something for a developer to take lightly, and code is always authoritative. Eventually I found myself sifting through Python internals, a deeper rabbit hole than I expected.

I will state for the record that I truly appreciate both Electrum and Trezor. Otherwise I would not have spent the time to provide comprehensive support for all three of these encodings. These observations are provided for my own record and to possibly aid others who may at some point find themselves in that same rabbit hole. When one goes this deep into implementation, interesting discoveries abound.

## Terminology
### Language
A universally-unique [natural language](https://en.wikipedia.org/wiki/Natural_language).
> Libbitcoin refers to a languages by the [IANA subtag](https://www.iana.org/assignments/language-subtag-registry/language-subtag-registry) standard.
### Token
In linguistics a `token` is an "individual occurrence of a linguistic unit in speech or writing".
> Tokens contain no [whitespace](https://en.wikipedia.org/wiki/Whitespace_character) [code points](https://en.wikipedia.org/wiki/Code_point).

> Tokens may or may not be normal form.

> Electrum allows seed generation from tokens (i.e. non-dictionary words) in normal form.

### Dictionary
A `dictionary` is a standard ordered set of distinct reference tokens of a single language.
> There may be more than one dictionary per language.

> Dictionaries of the same or distinct languages may intersect.

> A dictionary defines its word order, which may or may not be a [lexicographic sort](https://en.wikipedia.org/wiki/Lexicographic_order).
### Interpreter
An `interpreter` is a set of same length (word-count) dictionaries of distinct languages, each identified by language.
> An interpreter maps between entropy and mnemonic forms, given a specified or detected language.

> There is no necessary standard defining the set of interpreter dictionaries.
### Word
A `word` is a dictionary token.
### Mnemonic
A `mnemonic` is an ordered set of words from a common dictionary, conforming to standard size and [checksum](https://en.wikipedia.org/wiki/Checksum) constraints.
> Electrum v1 does not implement checksum constraints.

> A mnemonic may be fully contained by multiple dictionaries.

> A mnemonic may be [referred to](https://wiki.trezor.io/Developers_guide:Cryptography) as `recovery seed` by some implementations. 
### Whitespace
A `whitespace` character is a standard character with a [glyph](https://en.wikipedia.org/wiki/Glyph) of no visible pixels.
### Sentence
A `sentence` is a mnemonic [serialized](https://en.wikipedia.org/wiki/Serialization) as a [sinistrodextral](https://en.wiktionary.org/wiki/sinistrodextral) string of its words with whitespace [delimiters](https://en.wikipedia.org/wiki/Delimiter).
### Encoding
An `encoding` is a standard [bidirectional map](https://en.wikipedia.org/wiki/Bidirectional_map) between any mnemonic and its numeric representation.
> The Electrum v1 encoding is (inadvertently) not fully bidirectional.
### Normal Form
A `normal form` is a standard word, sentence or passphrase [character](https://en.wikipedia.org/wiki/Character_encoding) representation.
> A single glyph may have multiple distinct code points, and many distinct glyphs may be rendered similarly or identically.

> Word containment by a dictionary is determined by normal form equality.
### Entropy
Its `entropy` is the numeric representation of a mnemonic.
> Both a mnemonic and its entropy represent the same [entropic](https://en.wikipedia.org/wiki/Entropy) value.
### Passphrase
A `passphrase` is arbitrary text that may be combined with a mnemonic in the formation of a seed.
> Electrum v1 does not implement a passphrase.
### Seed
A `seed` is a secret number, derived using a standard [one-way hash](https://en.wikipedia.org/wiki/Cryptographic_hash_function) from a mnemonic.
### Master Private Key
A `master private key` is an [secp256k1](https://www.secg.org/sec2-v2.pdf) private key, obtained from a seed in a standard manner, allowing spending.
> BIP39 wallets typically derive this (and a chain code) from the seed in accordance with [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki).

> Electrum serializes the seed as a secret and chain code in accordance with [BIP32 serialization](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki#serialization-format).

> Electrum v1 maintains this as a 32 byte value.
### Master Public Key
A `master public key` is a secp256k1 public key, derived in the standard one-way manner from the master private key, allowing receiving.
> Electrum and typical BIP39 wallets derive this in accordance with [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki).

> Electrum v1 maintains this as a 64 byte value ([uncompressed, without prefix](https://en.bitcoin.it/wiki/Elliptic_Curve_Digital_Signature_Algorithm)).

## Standards
The following standards are implied by the above terminology.
* Language (identification)
* Dictionary (words and order)
* Mnemonic (length and checksum)
* Whitespace (delimiters)
* Normal Form (word, sentence, and passphrase)
* Encoding (entropy mapping)
* Seed (derivation)
* Master Private Key (derivation)
* Master Public Key (derivation)

## Hazards
### Unicode
The reliance of Electrum and BIP39 on [Unicode](https://en.wikipedia.org/wiki/Unicode) word and passphrase [normalization](https://en.wikipedia.org/wiki/Unicode_equivalence#Normalization) is an inherent risk. Unicode implementations are large and complex. Trivial conversions in [ASCII](https://en.wikipedia.org/wiki/ASCII), such as [lower-casing](https://en.wikipedia.org/wiki/Letter_case), become treacherous in Unicode. 

> "When two applications share Unicode data, but normalize them differently, errors and data loss can result. In one specific instance, OS X normalized Unicode filenames sent from the Samba file and printer sharing software. Samba did not recognize the altered filenames as equivalent to the original, leading to data loss. Resolving such an issue is non-trivial, as normalization is not losslessly invertible."

For this reason we have implemented Libbitcoin mnemonics without a hard dependency on Unicode normalization. The Electrum v1, Electrum, and BIP39 classes do not require Unicode normalization unless a non-ASCII passphrase is provided. If the library is compiled without WITH_ICU defined all features remain available with the exception that seed passphrases are ASCII limited.

For the same reason Libbitcoin does not support Electrum token-based seeding. All words must correspond to a dictionary. When WITH_ICU is defined, words are Unicode normalized before comparison, to improve the chance of matching. Ideally an implementation provides a dictionary-based word selector, making this unnecessary. If WITH_ICU is undefined then word normalizations are ASCII limited, though pre-normalized non-ASCII words will match the dictionary.

### String Functions
A mnemonic sentence must be parsed into a list of words for dictionary matching and seed generation. Similarly a mnemonic is often emitted in sentence form for portability.

### Language Differences

## Dictionaries
### Normalization
### Intersection
### Sort/Search

## Encoding
### Language (words of a single language)
### Languages (sentences of any language)
### Dictionary (search a language)
### Dictionaries (search a set of dictionaries)
### Mnemonics (sentences and dictionaries)
### Base2048 (Electrum and BIP39)
### Base1626 (Electrum v1)

## Electrum v1
### Dictionaries
### Word Dependence
### Word Normalization
### Unexpected Behavior
#### Entropy Overflow
#### String Data
### Encoder
### Decoder
### Stretcher
### Keys

## Electrum
### Dictionaries
### Word Independence
### Word Normalization
#### Unicode Split
#### Unicode NFKD
#### Unicode Lower
#### Unicode Combining
#### Not Diacritics!
#### CJK Compression
#### ASCII Join
#### Ideographic Space
### Passphrase Normalization
### Checksum (prefix/version)
### Entropy Independence
### Entropy Loss
### Encoder
### Decoder
### Grinder
### Seeder
### Keys

## BIP39
### Dictionaries
### Word Dependence
### Word Normalization
#### Unicode Split
#### Unicode NFKD
#### Unicode Lower
#### ASCII Join
#### Ideographic Space
### Passphrase Normalization
### Checksum
### Entropy Dependence
### Encoder
### Decoder
### Seeder
### Keys
