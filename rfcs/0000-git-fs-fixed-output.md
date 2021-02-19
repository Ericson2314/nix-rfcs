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
This fixes an annoying U.X. problem right now, and opens the door to modularizing Nix for better integration with other technologies in the future.

# Motivation
[motivation]: #motivation

## Fixed-output derivation U.X.

As [RFC #84](https://github.com/NixOS/rfcs/pull/84) points out, we currently have a very annoying user experience updating sources.
It's very easy to update the version control system's identifier, but fail to update the nar hash.
Nix, will then see that the resulting fixed-output derivation has a hash of data we already have, and just use that data instead of downloading fresh.

But the fix proposed in RFC 84 is to effectively cancel out the fixed-output derivation semantics by stuffing an input address in the name of a derivation.
This is bad:

- Changes to *how* we obtain the source cause needless rebuilds

- Changes to *whence* we obtain the source cause needles rebuilds

Still we need a solution to this long-standing foot-gun.
This can be it.
Not only does having 1 hash remove the possibility hashes being out-of-sync,
having one hash is just *plain easier*.

## Better integration with other tools

As Eeclo says:

> it goes the wrong way, namely away from content-addressability" (In CA Nix, ideally we wouldn't have a name attribute...)

It's a bit hard to explain why content-addressing is so great, but one aspect is that it's a concept shared by a number of tools, including ever popular Git.
I've long thought while the Nix community should never give up the important and *necessary* ways we do things differently, there are a number of accidental ways we are bespoke to no benefit:

 - We hash file system data in a bespoke way

 - We store data in a bespoke way

 - We exchange data between stores in a bespoke way

 - We sandbox data

I think one of the best ways to promote Nix is to collaborate with other parties.
Making these areas less bespoke is a great way to "compromise" when integrating Nix, *without compromising any of our core principles*.

Finally, it should be to our benefit even beyond growing the community in that the division of the labor with good abstractions (rather than Conway's law tragic interfaces) deduplicates work to everyone more.
(This is why modality is essential for free software.)

# Detailed design
[design]: #detailed-design

1. Add a third way way of agglomerating data in addition to "flat" and "recursive": "git".
   This is:
     - git blobs hashing method for files.
     - git tree hashing method for directories.
     - special directory entry for blobs representing symlinks.
   Submodules and other exotic named entrees in trees are prohibited.

2. Only support the SHA1 hashing algorithm with "git-fs".
   As git adopts better hashing algorithms, so can we.

4. Fixed-output derivations may use "git-fs" + "sha-1", too.

5. For the time being:
   - still transmit store entries with nar files.
   - still always calculate Nar hash for local store entries.
   Nix already works well with a normative hash + nar hash.

# Drawbacks
[drawbacks]: #drawbacks

- Data that isn't content-addressed doesn't benefit.

  Of course, once we download data, we can hash it however we like,
  but if it there isn't a git hash to use in the URL, etc. we are back where we started with information that can drift out of sync.

  It is my contention that it's best to just live with this.
  The most frequently changing sources tend to be git, due to the tenancy for project development to die down, and git's complete dominance in popularity in recent years.
  Also, the new users mostly likely to be caught unaware by the bad U.X. are probably working with their own code.
  For the same statistical reasons, that is also extra likely to be in git.

  Data the only comes in tarballs or compressed, and then is hashed as-is, poses a number of difficulties such as the ones described at the bottom of https://www.tweag.io/blog/2020-06-18-software-heritage/.
  Not helping that legacy path as much could help incite people to move away from it, which I think is a good thing.

- SHA-1 is a bad hash algorithm.

  My view is that:

  - It's worth using it temporarily to foster collaboration.
    Git is already migrating away from SHA-1.
    We can deprecate it accordingly.

  - Working with the git hash migration is a good exercise in crypto-agility we would miss if we waited.

  - When we rehash git data *defined* by SHA-1, the improvements are somewhat nebulous anyways.

- Git tree objects don't do chunking / fancy dedup / etc.

  But, Git is currently the preeminent way of content-addressing source code.
  It is the basis behind https://docs.softwareheritage.org/devel/swh-model/persistent-identifiers.html too.
  It is my view that collaboration, and not https://xkcd.com/927/, is the best way.
  Supporting git can be the beginning of supporting more formats, if one becomes popular.

# Alternatives
[alternatives]: #alternatives

- Do nothing

- [RFC #84](https://github.com/NixOS/rfcs/pull/84)

# Unresolved questions
[unresolved]: #unresolved-questions

1. Permissions and attributes.
   Git and Nix both just support one executable bit for files, so that's great.
   I'm not sure what either does for directories.

2. Anything we need to note specially in `narinfo` files?

# Future work
[future]: #future-work

Actually integrate with those other technologies!
Full disclosure, this would help immensely with the integration with IPFS, which I have worked on.
But I've tried to come up "proper-noun-neutral" language to emphasize I don't just care about this because of the specific projects I work on.
