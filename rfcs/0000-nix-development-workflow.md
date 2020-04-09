---
feature: nix-development-workflow
start-date: (fill me in with today's date, YYYY-MM-DD)
author: John Ericson (@Ericson2314)
co-authors:
shepherd-team: (names, to be nominated and accepted by RFC steering committee)
shepherd-leader: (name to be appointed by RFC steering committee)
related-issues: (will contain links to implementation PRs)
---

# Summary
[summary]: #summary

Codify development practices for Nix to encourage new contributors and ease release management.

# Motivation
[motivation]: #motivation

Nix releases have been unpredictable, such that new features (and, less often, bug fixes) sit on master for a long time before people can rely on them.
We want to hold master to a standard of high quality so backports can be cherry-picked and releases cut with confidence.

Many PRs languish as it is unclear who besides @edosltra can release and merge them.
By clarifying the standards of PRs and holding all development to it, we hope to level the playing field.

The time is ripe for this.
When originally proposed, this policy change would have substantially changed developer workflows, but peoples habits have thankfully been moving in this direction.
Also, now that we have CI, a lot goals that were out of reach no longer are.
This PR seeks to solidify (and celebrate!) those changes, and encourage more of them going forward.

# Detailed design
[design]: #detailed-design

1. Do non-trivial work in feature branches.
   Branches should always be merged with a PR, even if they are not reviewed.
   This is done for consistency of the repo history, and ease of browsing on the web.

2. Only merge PRs which pass all tests, unless there is a CI issue or other extenuating circumstance.

3. Only merge PRs which are approved by a non-author, unless the author is @edolstra, in which case an external approval is merely encouraged.

4. If a feature branch may introduce regressions, including performance regressions, ensure they are tested by relevant parties before merge.
   If the regression test can be automated, write it and commit first to demonstrate behavior is unchanged.

5. Master should always be releasable.
   If Master is not releasable, the next merged PR is not obligated to fix every problem (though that is encouraged!)
   However, if Master is releaseable, no change can be knowingly made that makes it less releasable.

   Release master frequently.
   Community members can make a request for a new release and it's almost always granted.

6. All new features must first land as experimental.
   Only existing features can be stabilized, preferably in conjunction with an RFC.

7. If the need arises, start maintenance branches that *only* do bug fixes.

# Examples and Interactions
[examples-and-interactions]: #examples-and-interactions

- A sub-par cannot be merged if it makes things no better, even if someone says "but a release is a long ways off so we have time to clean up"

- A sub-par PR can merged if and only if what currently exists is strictly worse.

# Drawbacks
[drawbacks]: #drawbacks

 - Since the burden is amortized, the additional work per PR ought to be minuscule, but developers might nevertheless find it frustrating.
   The change may also make some large-scale features harder to develop, though practice disputes this.
   Other projects following this policy haven't found that to be much of an issue.

# Alternatives
[alternatives]: #alternatives

This is based on #9, which, dating before the current "golden age" of the RFC process, languished.
It was replaced with #28, which also predating current golden age, languished.
#28 sought to add many, many more concrete details, but it is John's opinion that it is counter-productive.
This risk with these RFCs is that unlike Nixpkgs, Nix, has always been a primarily @edolstra rather than community endeavor, and we don't want to upset too much too quick lest the RFC go no-where.
It is best to build confidence with smaller steps to begin demonstrate that the community can share the burden without screwing everything up, than go for complete, exactingly-detailed "rule of law" right out of the gate and get nowhere.

# Unresolved questions
[unresolved]: #unresolved-questions

The steps needed to get 1.12 out.

# Future work
[future]: #future-work

- Set release cadence

- Set versioning policy

- Codify what sort of PRs *don't* need Eelco's approval.

- Stabilizing interfaces for Nix being used as a library
