# HashFusion
HashFusion - Reference implementations and documentation for flexible hash combination.

Author: brian.monahan@hpe.com

Version 1.1, August 2017

## Introduction
HashFusion provides a way of flexibly combining hashes in an _associative_ and
_non-commutative_ manner.  This allows greater flexibility in building hash
trees and _accumulation structures_ than the known solution using Merkle hash trees.
HashFusion is particularly useful when assembling out-of-order data (streaming)
and also in handling integrity of file-systems (currently work-in-progress).

### Accumulation Structures.
HashFusion supports three types of accumulation structure:

- **Direct:** (In-order data) No accumulation structure needed.
  Assemble hashes by walking over the data in-order.
  This leads to an O(N) solution overall.

- **Linear List:**
  (Out-of-order data)  A linear list of partial hash fragments
  is used that grows and shrinks as hash fragments are added.  Finally, only one
  hash fragment should be left once all hashes have been added.
  This needs O(N) effort to insert fragments, leading to an O(N<sup>2</sup>)
  solution overall for out-of-order data.

  This method is applicable for _transient_ construction of overall hashes for
  small numbers of data blocks (where no update is needed). 

- **Self-Balancing Binary Tree (Red-Black):**
  (Out-of-order data)  A self-balancing binary tree provides a _scalable_
  solution to assembling overall hashes.
  This needs O(lg N) effort to insert fragments (same as Merkle trees), leading
  to an O(N.ln N) solution overall for out-of-order data.

  This method is applicable for both transiently constructing
  overall hashes and for maintaining a persistent set of blocks, since updating
  existing entries takes O(lg N) effort.
  
  More significantly, we can expoit HashFusion to provide insertion/deletion of
  unanticipated entries into an existing structure with O(lg N) effort - the
  corresponding operation is typically O(N.lg N) for Merkle trees.

### Security analysis: Collision-resistance?
We are investigating security properties, in particular preservation of
collision-resistance of hashes combined using HashFusion.

## Source code
The source code implements:

-  The current HashFusion library (ANSI C).  See src/lib
-  Basic demonstrator application: hmt.  See src/app

Directory structure:

- doc
- src
  - lib
  - app
- README.md

## Usage: hmt
This app provides command-line performance tests of hash-fusion code
compared with using Merkle Trees.  The name "hmt" means "Hashfusion - Merkle
tree Test".

### Approach
The basic experiment peformed by 'hmt' consists of:
  
  1. Generating a sequence of randomised data blocks
  2. Calculating hash of the given block (computed in sequence).
  3. Randomise the order of the blocks so that they are presented in randomised sequence.
  4. Rehash the out-of-order sequence of blocks by using a specified accumulation structure.
     to build the required in-order hash.
  5. Compare the two hashes ... (hashes should be identical)
  6. Report statistics (e.g. timing averages)

This is run for HashFusion and/or Merkle hash trees (default is both).

### Options
The following options are supported:

    -a <accum-type>   : Accumulation structure (either d: direct, l: linear, t: tree)
                        (default: t).

    -s <seed-value>   : Seed value for generating random data.  Setting this to
                        zero provides a seed determined by date/time
                        (default: 27652761).

    -h                : HashFusion only.

    -m                : Merkle Tree only.

    -r <runs>         : Number of runs (default: 1).

    -n <blocks>       : Number of blocks. (default: 50)

    -b <block-size>   : Size of each data block (default: 1024 bytes).

    -csv              : CSV format report output -- reports stats using csv output format.
                        This format will output the column header line once.

    -json             : JSON format report output.

The block-size and number of blocks options can use a multiplier code (k = 1024).

When using the 'direct' accumulation structure, it doesn't make sense to deal with out-of-order
data.  In this case, only the first two (and final) steps are performed.

### Examples:

     hmt -s 236483624 -b 8k -n 22k -r 20
     -- Use seed 236483624, blocksize = 8192, number of blocks = 22528, runs = 20
        and default binary tree accumulation structure

     hmt -s 4526263 -b 4k -n 2k -r 30 -a l
     -- Use seed 4526263, blocksize = 4096, number of blocks = 2048, runs = 30
        and use the linear list accumulation structure

     hmt -b 6035 -h -n 16k -r 25
     -- Use default seed, blocksize = 6035, perform HashFusion only, number 
        of blocks = 16384, and runs = 25

## Getting help with hmt
Type hmt with no arguments to get info/help ...

## Running hmt
To run hmt, ensure that src/bin/hmt is on your path.

## Building hmt
To build src/bin/hmt:

+  cd src
+  make clean
+  make build

## OpenSSL libraries
The code uses the OpenSSL libraries - these will need to have been built/installed
before you can compile hmt.   This typically involves downloading the latest source,
compiling it and then installing the libraries (usually in /usr/local/lib).

