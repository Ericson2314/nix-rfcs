
---
feature: git-fs-fixed-output
start-date: 2019-10-22
author: John Ericson (@Ericson2314)
co-authors: (find a buddy later to help our with the RFC)
shepherd-team: (names, to be nominated and accepted by RFC steering committee)
shepherd-leader: (name to be appointed by RFC steering committee)
related-issues: (will contain links to implementation PRs)
---

# Summary
[summary]: #summary

Natively support git tree hashes.
This opens the door to decoupling the storage layer from the rest of Nix
That, in turn, opens the door to many other interesting possibilities.

# Motivation
[motivation]: #motivation

If Nix's implementation and specification were more modular, we could more easily leverage other tools for interesting experiments.
Thankfully, the nix language is fairly separate from the rest. E.g. GNU Guix had no trouble staring with our daemon.
But, the storing data and perfoming building is sadly still rather intertwined.
Performing builds of course is the heart of what Nix does; that part shouldn't be outsourced.
Storing data, quering remotes for data, and mounting that data in `/nix/store`, however, in principle has nothing to do with the above.
But the format of store paths, hashing method, and probably a long tail of smaller papercuts make a non-nix-specific implementation of this functionality almost impossible to news.

We cannot just break all existing store paths, but we can create new opt-in modes of operation that will avoid these Nix-specific choices.
The most basic store-related functionality Nix has is adding a new store entry, closely followed by fixed-output derivations.
These are thus good place to start creating an off-the-shelf mode.
Such added store paths and built fixed output derivation builds are content addressed, with the content just being the files themselves and no Nix-specific metadata.

Git blob and tree hashes are well suited to this task.
Git is extremely prevalent; there might be more git hashed data than anything but BitTorrent.
Unlike BitTorrent, git is much better suited to incrementally modified data, as the filesystem is hashes recursively, which allows direct reuse of unmodified subtrees.
This matches our use case where both source code and builds me change incrementally.
Git is thus a better fit than BitTorrent.
While this proposal doesn't leverage the incrementality directly, ideas in the future work section do.

# Detailed design
[design]: #detailed-design

1. Add a third way way of agglomerating data in addition to "flat" and "recursive": "git-fs".
   This is git blobs hashing method for files, and git tree hashing method for directories.
   Submodules and other exotic named entrees in trees are prohibited.

2. Only support the SHA1 hashing algorithm with "git-fs".

3. Store paths hashed according to "git-fs" do not have names, but are just the hash.

4. Fixed-output derivations may use "git-fs" + "sha-1".
   The derivation still has a name, but the resulting build won't per the above.

5. For the time being, still transmit store entries with nar files.
   Of course, the hash cannot be verified by hashing the nar byte for byte.

# Drawbacks
[drawbacks]: #drawbacks

The proposal makes Nix bigger with no immediate benefit.

# Alternatives
[alternatives]: #alternatives

# Unresolved questions
[unresolved]: #unresolved-questions

1. Permissions and attributes.
   Git and Nix both just support one executable bit for files, so that's great.
   I'm not sure what either does for directories.

2. Anything we need to note specially in `narinfo` files?

# Future work
[future]: #future-work

Here's the fun part!
There are ton of ideas to explore that all become more feasible with this change in place.
I'll try to detail some of them

## Data Exchange

HTTP is not an efficient way to multicast data.
IPFS or something similar has all the tools to do a much better job, side-stepping the far more difficult task traditional network filesystems face.
While we cannot predict what technology will "win", we can be pretty sure they will support git since so much git data already exists and is well suited to bootstrap the ecosystem.
This was the original use-case for [Nix issue #1006], where I originally proposed this.

## Deduplication

For storage, we support deduplicating with hardlinks, but this still wastes metadata when whole trees are duplicated.
Git hashing makes it O(1) to compare subtrees, which is essential to doing this efficiently on-line.
To actually do it though, I'm afraid we'd need some sort of custom filesystem (including FUSE), as the hardlink restriction is OS-imposed.
For this reason, it's not part of the proposal proper.

## Projecting store paths

Often, a derivation only depends on a subtree of store path.
Unfortunately, due to the way Nix works today, if only the rest of that store path changes, the build will still be invalidated.
To efficiently do the deduplication mentioned above, we need an index of all tree hashes recursively decomposed.
At that point, and given we have a custom filesystem anyways, we might as well expose all these subtrees as store paths in their own right.
We can have a special derivation to project a subtree from a store path which can work by copying "recursive", but be optimized for "git-fs".

## Integration with Intentional Store

The intentional store in a nutshell rewrites drvs so that all their dependencies are fixed-output derivations.
We are just introducing a new type of fixed output derivation, so that shouldn't be problem.
This also provides proper formalism for the special projection derivation mention above, as that derivation must "lose" it's connection to the original drv if it is to achieve its purpose of avoiding unecessary rebuilds.

This is a point of complication, however, self-references.
In short, many builds today contain their own path, and thus are unable to be content-addressed.
To work around that, there is a rewriting step to break the loop with self-references.
Once we expose "git-fs" trees as compositions of other "git-fs" builds, we no longer have a "self", and must deal with larger cycles.
There's a couple ways around this; I think it's fine as a first step to punt.
We can assert that certain derivations must not contain any cycles and limit "git-fs" hashing to such derivations.

[Nix issue #1006]: https://github.com/NixOS/nix/issues/1006