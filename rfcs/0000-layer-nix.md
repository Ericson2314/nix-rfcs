---
feature: layer-nix
start-date: 2022-03-06
author: John Ericson
co-authors: (find a buddy later to help out with the RFC)
shepherd-team: (names, to be nominated and accepted by RFC steering committee)
shepherd-leader: (name to be appointed by RFC steering committee)
related-issues: (will contain links to implementation PRs)
---

# Summary
[summary]: #summary

Split Nix into more layer, and better enforce the layers that currently exist.
Do this to acknowledge and support the *pluralism* of the Nix community, and foster our differing and sometimes diametrically opposed goals.
As a bonus, try to build bridges with communities that have already forked off, such as Tvix and Guix.

# Motivation
[motivation]: #motivation

I believe the Flakes saga has sent towards an impending schism.
Right now things are somewhat quite, but I fear the process of stabilizing Flakes will rip the wounds back up.
No one wants this to happen, but I haven't seen a concrete off-ramp to avoid that looming outcome proposed either.

This is an attempt to plan that off-ramp, to write the road-map for keeping the community together.

The community dissolution to be avoided is very scary, but I don't want plan to just be about that.
At the same time we guard against that, I also want to move back in reach some goals we haven't yet considered, or already gave up on.
I am writing this fearing our current trajectory, but if we accept this plan, I will have more hope than I did before Flakes.

## Identifying the problem

The first step in finding the solution is to identify the problem.

I think the dominant narrative around Flakes has been similar to what Graham Christensen wrote in his blog post [flakes-are-an-obviously-good-thing](https://grahamc.com/blog/flakes-are-an-obviously-good-thing).
A rough summary is Flakes themselves are not the problem, just the manner in which they were rolled out, especially with the initial RFC.
Reduced even further it's this: our disagreements are fundamentally social, not technical.

While certainly are parts that process that could have gone better, and perhaps those parts caused the most pain and acrimony, I don't think that is the heart of the matter.
I was in those RFC meetings, and many others with Eelco and the other participants, and I don't* think had the RFC process been more mature we could have set towards a final design we basically all agreed upon.
At the root of all of this, I think there *is* a fundamental technical disagreement and irreconcilable difference in design philosophies.
This the core point of contention I don't think has gotten enough attention, and I think bringing it front and center to discuss is the key to getting us out of our current quagmire.

Now, this might sound very depressing:
If there was no fundamental disagreement, just bad blood, we could overcome that and get back to working together in perfect alignment.
Conversely, if there is a fundamental disagreement, how can we ever move past this and forward?
What even *is* "forward" when groups of us want fundamentally different things?

If I may, I think this is good news.
If there was no fundamental contention, then all of the flakes drama would be both a tragedy and a bit of farce.
If there is a fundamental contention, then the resulting drama reflects less poorly on our governance; it's still tragic, but not tragic in its sheer pointlessness.

A major underlying theme of Graham's post, and, I shall argue, of the philosophy behind the design of Flakes, is that the Nix community has grown immensely in recent years, and this requires fundamental shifts.
I agree with both of those!
To me however, the main less is to embrace pluralism and disagreement, and abandon any notion of complete community unity or even consensus on all major points.
If we give up on a single `/nix/store/...-nix/bin/nix` making everyone happy, we take a great weight off our shoulders.

## A brief history of Nix

So, what actually is this big central design disagreement?
I'll get there in just a the a moment, but let me lay out the brief history.

I think a lot of Nix's "old guard" was drawn to the project because it seemed to espouse a certain set of values.
These values came from Nix's origins in the functional programming language community, and were carried over from that community's acts of defining itself in opposition to what we might call the "computing mainstream".
Functional programming has become less obscure, and Nix also, and both have happened because neither community actually wants to put up walls.
Importantly, that means over time Nix has had more users *and core contributors* who wouldn't consider themselves "from" functional programming, functional programming die-hards, etc.
Full disclosure, I definitely consider myself both --- and so its very I don't want to come off as minimizing the contributions of those that don't!

I am not sure whether Eelco considered himself at any point over Nix's almost 20 year history, but this is besides the point.
What happened is that the Nix community experienced certain growing pains --- completely understandable for any rapidly growing community but at the same time always a good chance for self reflection not something which should be shrugged off.
Long before Flakes, this led Eelco to consider issues of community cohesiveness and new user accessibility.

, but not something which just because its inevitability shouldn't be

### The functional programming language school --- asceticism

### The product school --- feature completeness

With flakes and other development, we are moving towards a more "batteries included" Nix command line.
We don't want any of those features in the daemon, however, because the daemon is a special trusted process that we should strive to keep as simple as possible.
\[This is comparable to a compilation pipeline, with a concise intermediate representation that nicer user-facing features "desugar" into.\]

There are many things we could do about this, but I mainly want to establish some rough consensus around the problem while taking a small step to signal that consensus.
Originally, each Nix command was its own executable, but then we combined them into one executable.
I think this is fine for the main user-facing commands, but not good for the daemon.

Finally, it's probably best not to give the daemon---a long lived process running with elevated privileges---access to tons of dead code.
All of the other commands entry points and library functions they use, such as the Nix evaluator, are in the same process even though the daemon should never need to use them.
C++ doesn't exactly prevent memory errors, and that dead code is just more fodder to be used in some low-level attack.
There are other solutions to this in the long term, but this is the easiest solution in the short term.

# Detailed design
[design]: #detailed-design

- `nix-daemon` will be a separate executable that only links the nix libraries it needs.
  \[At this time, those libraries are `libnixutil`, `libnixstore`, and `libnixrust`, but this is subject to change.\]

- `nix-daemon` should never need to understand the expression language and depend  `libnixexpr`.

# Examples and Interactions
[examples-and-interactions]: #examples-and-interactions

I certainly hope there are no interactions!
One of the bad things we should seek to prevent with this is the daemon unintentionally growing dependencies on more of the code base.

# Drawbacks
[drawbacks]: #drawbacks

Installation is slightly bigger as the two binaries (`nix` and `nix-daemon`) have some redundancy.
Build rules perhaps are slightly more complex as there are both separate and independent executables.

# Alternatives
[alternatives]: #alternatives

 - Do nothing.

 - Something more invasive, such as packaging the libraries and commands separately, or intending the libaries for widespread public consumption.
   But I much rather save that for later, as such steps would be far more controversial.

# Unresolved questions
[unresolved]: #unresolved-questions

No known unknowns.

# Future work
[future]: #future-work

I think most people for this will have future plans they wish to persue in the name of modularity.
But I don't expect everyone to agree on what exactly those plans should be.
The point of this small step is to punt on all that for now.
