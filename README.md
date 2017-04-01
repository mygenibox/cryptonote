This is the pre-implementation for NomadCoin (NOM) on Egalitarian proof of work example.

* Launch NomadCoin (NOM) currency: 
* NomadCoin (NOM) reference pre-implementation: [NomadCoin Pre-implementation](https://github.com/mygenibox/nomadcoin)
* NomadCoin Wiki: [NomadCoin Wiki]

## NomadCoin (NOM) Path-Map

### Pre-implementation and Preparation.

1. Ubuntu servers setup for seed node.
2. Setting up mining tools, mining pool and wallet on universal pool for pre-mining.
3. Coding for pre-mining.
### Below is the CryptoNote forked setup for NomadCoin (NOM) pre-implementation purposes only.


**Change file src/CryptoNoteConfig.h** - `CRYPTONOTE_NAME` constant 
to```
const char CRYPTONOTE_NAME[] = "nomadcoin";
```
**Change in src/CMakeList.txt file** - set_property(TARGET daemon PROPERTY OUTPUT_NAME "")

to
```
set_property(TARGET daemon PROPERTY OUTPUT_NAME "nomadcoind")
```

**Note:** Changed Repository name

### Emission logic 

**1. Total money supply** (src/CryptoNoteConfig.h)

Total amount of coins to be emitted. Most of CryptoNote based coins use `(uint64_t)(-1)` (equals to 18446744073709551616). Defining number explicitly using example `UINT64_C(858986905600000000`).

const uint64_t MONEY_SUPPLY = (uint64_t)(-1);
```

**Change Emission curve** (src/CryptoNoteConfig.h)

Be default CryptoNote provides emission formula with slight decrease of block reward with each block. This is different from Bitcoin where block reward halves every 4 years.

`EMISSION_SPEED_FACTOR` constant defines emission curve slope. This parameter is required to calulate block reward. 

```
const unsigned EMISSION_SPEED_FACTOR = 18;
```

**Change Difficulty target** (src/CryptoNoteConfig.h)

Difficulty target is an ideal time period between blocks. In case an average time between blocks becomes less than difficulty target, the difficulty increases. Difficulty target is measured in seconds.

Difficulty target directly influences several aspects of coin's behavior:

- transaction confirmation speed: the longer the time between the blocks is, the slower transaction confirmation is
- emission speed: the longer the time between the blocks is the slower the emission process is
- orphan rate: chains with very fast blocks have greater orphan rate

For AltCoins difficulty target is 1 or 2 minutes.

```
const uint64_t DIFFICULTY_TARGET = 120; - where 120 as seconds
```

**Change Block reward formula**

By default, implementation of block reward logic can be changed. `src/CryptoNoteCore/Currency.cpp`:
```
bool Currency::getBlockReward(size_t medianSize, size_t currentBlockSize, uint64_t alreadyGeneratedCoins, uint64_t fee, uint64_t& reward, int64_t& emissionChange) const
```

This function has two parts:

- basic block reward calculation: `uint64_t baseReward = (m_moneySupply - alreadyGeneratedCoins) >> m_emissionSpeedFactor;`
- big block penalty calculation: this is the way CryptoNote protects the block chain from transaction flooding attacks and preserves opportunities for organic network growth at the same time.

Only the first part of this function is directly related to the emission logic. See MonetaVerde and DuckNote as the examples where this function is also modified.


### Third step. Peer to peer networking

**1. Default ports for P2P and RPC networking** (src/CryptoNoteConfig.h)

P2P port is used by daemons to talk to each other through P2P protocol.
RPC port is used by wallet and other programs to talk to daemon.

Choosing ports that aren't used by other software or coins.

```
const int P2P_DEFAULT_PORT = 17236;
const int RPC_DEFAULT_PORT = 18236;
```


**2. Network identifier** (src/P2p/P2pNetworks.h)

This identifier is used in network packages in order not to mix two different cryptocoin networks so chainging all the bytes to random values on the network:
```
const static boost::uuids::uuid CRYPTONOTE_NETWORK = { { 0xA1, 0x1A, 0xA1, 0x1A, 0xA1, 0x0A, 0xA1, 0x0A, 0xA0, 0x1A, 0xA0, 0x1A, 0xA0, 0x1A, 0xA1, 0x1A } };
```


**3. Seed nodes** (src/CryptoNoteConfig.h)

Adding IP addresses of seed nodes examples:

```
const std::initializer_list<const char*> SEED_NODES = {
  "111.11.11.11:17236",
  "222.22.22.22:17236",
};
```


### Changing transaction fee and related parameters

**1. Minimum transaction fee** (src/CryptoNoteConfig.h)

Zero minimum fee can lead to transaction flooding. Transactions cheaper than the minimum transaction fee wouldn't be accepted by daemons. 100000 value for `MINIMUM_FEE` is usually enough.

```
const uint64_t MINIMUM_FEE = 100000;
```


**2. Penalty free block size** (src/CryptoNoteConfig.h)

CryptoNote protects chain from tx flooding by reducing block reward for blocks larger than the median block size. However, this rule applies for blocks larger than `CRYPTONOTE_BLOCK_GRANTED_FULL_REWARD_ZONE` bytes.

```
const size_t CRYPTONOTE_BLOCK_GRANTED_FULL_REWARD_ZONE = 20000;
```


### Address prefixing

Prefixing all the coins public address defined by `CRYPTONOTE_PUBLIC_ADDRESS_BASE58_PREFIX` constant. using [prefix generator tool].

```
const uint64_t CRYPTONOTE_PUBLIC_ADDRESS_BASE58_PREFIX = 0xe9; // addresses start with "f"
```


### Genesis block

**1. Build the binaries with blank genesis tx hex** (src/CryptoNoteConfig.h)

You should leave `const char GENESIS_COINBASE_TX_HEX[]` blank and compile the binaries without it.

```
const char GENESIS_COINBASE_TX_HEX[] = "";
```


**2. Starting the daemon to print out the genesis block**

Run your daemon with `--print-genesis-tx` argument that will print out the genesis block coinbase transaction hash.

```
nomadcoind --print-genesis-tx
```


**3. Copy the printed transaction hash** (src/CryptoNoteConfig.h)

TX hash that has been printed by the daemon for the `GENESIS_COINBASE_TX_HEX` in `src/CryptoNoteConfig.h`

```
const char GENESIS_COINBASE_TX_HEX[] = "013c01ff0001ffff...785a33d9ebdba68b0";
```


**4. Recompiling the binaries**

NomadCoin node should be ready after Recompiling!

## Building and mining NomadCoin

### On *nix

Using dependencies: GCC 4.7.3 or later, CMake 2.8.6 or later, and Boost 1.55.

can be downloaded from:

* http://gcc.gnu.org/
* http://www.cmake.org/
* http://www.boost.org/
* Alternatively, it may be possible to install them using a package manager.

To build, change to a directory where this file is located, and run `make`. The resulting executables can be found in `build/release/src`.

**Advanced options:**

* Parallel build: run `make -j<number of threads>` instead of `make`.
* Debug build: run `make build-debug`.
* Test suite: run `make test-release` to run tests in addition to building. Running `make test-debug` will do the same to the debug version.
* Building with Clang: it may be possible to use Clang instead of GCC, but this may not work everywhere. To build, run `export CC=clang CXX=clang++` before running `make`.

### On Windows
Using dependencies: MSVC 2013 or later, CMake 2.8.6 or later, and Boost 1.55 that are downloadable from:

* http://www.microsoft.com/
* http://www.cmake.org/
* http://www.boost.org/

To build, change to a directory where this file is located, and run commands: 
```
mkdir build
cd build
cmake -G "Visual Studio 12 Win64" ..
```
Build NomadCoin (NOM) expanding the blockchain probability.
Good luck!
