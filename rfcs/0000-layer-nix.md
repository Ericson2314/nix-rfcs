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
I was in those RFC meetings, and many others with Eelco and the other participants, and I *don't* think had the RFC process been more mature we could have set towards a final design we basically all agreed upon.
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

## A brief history of Nix, and two schools of design

So, what actually is this big central design disagreement?
I'll get there in just a the a moment, but let me lay out the brief history.

I think a lot of Nix's "old guard" was drawn to the project because it seemed to espouse a certain set of values.
These values came from Nix's origins in the functional programming language community, and were carried over from that community's acts of defining itself in opposition to what we might call the "computing mainstream".
Functional programming has become less obscure, and Nix also, and both have happened because neither community actually wants to put up walls.
Importantly, that means over time Nix has had more users *and core contributors* who wouldn't consider themselves "from" functional programming, functional programming die-hards, etc.
(Full disclosure, I definitely consider myself both --- and so its very I don't want to come off as minimizing the contributions of those that don't!)

### The functional programming language school

I think I can best explain this school of design with some famous precepts.

> Programming languages should be designed not by piling feature on top of feature, but by removing the weaknesses and restrictions that make additional features appear necessary.

> Composition over configuration

> Avoid success at all costs.

I could write a lot more on these, but the basic message is a "tortoise and hair" like one.

In the short term, it is very tempting to pile another feature or configuration option on top.
Brand new features mean no risk of breakage, configuration can likewise by defaulted so one need care.
And yes, in that short term this may be the quickest path to success.

But in the long term, features and configurations pile up into ton of complexity, and *still* a mere still a mere simulacrum of flexibility.
Shifting tastes may also make certain more opinionated solutions loose their luster.
There is a general sense that if entropy is not continually beaten back, it will win in the code base.

To the critics of this approach it must be admitted that a cogent calculation of short term vs long term needs isn't always even the point.
There is no lying that there contrarian fun was found in being the perennial tortoise, making short term self-sabotaging actions in the name of long term mores.
The pursuit of long term good design can become the goal in and of itself, and the relishing whatever asceticism is needed to stick to that path the way to demonstrate virtue.

Nix excelled at hitting many of these points.
The strict separation between the drv language and the Nix expression language stands in stark contrast to the Makes, and Bazels of the world.
The Nix expression language also has a minimum of extra features, "beating Scheme at it's own game" one could say (outside the syntax itself, at least).
Of course, going for purity long before OS sandboxing became cool was a *serious* up front cost, as we invariably were using software in ways its original authors never anticipated.

One last precept, taking from the Eelco's PhD dissertation (Thanks Tom Berek for finding):

> Nix is *policy-free*;
> it provides *mechanisms* to implement various deployment policies, but does not enforce a specific one.
> Some policies described in this thesis are *channels* (in push and pull variants), *one-click installations*, and *pure source deployments*.

(https://edolstra.github.io/pubs/phd-thesis.pdf page 15, emphasis original)

### Changing winds

I am not sure to what extent Eelco considered himself an adherent to this school over Nix's almost 20 year history, but this is besides the point.

What happened is that the Nix community experienced certain growing pains --- completely understandable for any rapidly growing community but at the same time always a good chance for self reflection not something which should be shrugged off.
A blank `default.nix` and the minimalism of the Nix expression language make it highly unclear *what* they ought to do.
"Lambda calculus --- go wild" can make for quite the writer's block!
In practice, the central position of Nixpkgs imposed some order in the community.
But still, different downstream projects often developed completely different idioms.

Talking to Eelco over the years, I get a sense that he has increasingly seen Nix's minimalism and modularity as the root cause of these issues.
Where many different idioms were equally good, the project could step in and anoint one the winner, and the community could rally around it.
Where the existing layering prevented a solution, the layering could be violated to do that solution anyways.

This think did lead to Flakes, but also two a few things before Flakes that I think are worthwhile to bring up.

#### `nix-shell`

Firstly, there is good old `nix-shell`, which dates back (under a different name) to https://github.com/nixos/nix/commit/7f38087f35e6f74a73bfdb28da8acd8930565d51 in 2012.
From this first commit, it assumed a Nixpkgs-made derivation, doing `source $stdenv/setup`.
From the PL school's perspective, this is a gross layer violation --- derivations need not have an environment variable called `stdenv`! --- and something to be avoided.
But of course, it can't be argued that the assumption that the use Nixpkgs/stdenv/`mkDerivation` isn't highly likely.
Likewise, there was a clear need to be able to both better debug Nix derivations and get ad-hoc shells (rather than statefully use e.g. `nix-env`).
`nix-shell` has been quite useful and popular in the years since.
Still, it could be argued that we should have made a `nixpkgs-shell` that wrapped the underlying `nix-shell`, adding the `source $stdenv/setup`.
Would it have worked?
Sure.
Would it have been as accessible and discoverable to users?
Arguable either way.

#### `builtins.fetch*`

More recently there are the `builtins.fetch*` family of prim-ops.
It is well known that fixed output-derivations are hard to use in many ways:

 - Sometimes during quick development you just want to impurely fetch latest version, just like `./...` impurely grabs the latest version on disk. This is not possible.
 - When the fetching system also uses hashes (e.g. Git), there are two sets of hashes that must be kept in sync.
 - Forgetting to update the Nix hash "silently" leads to cache hits without any indication the user might have made a mistake.
 - Private sources / authentication is super harder, especially since a safe way to keep secrets in the Nix store hasn't yet been devised.
 - When fetching from version control, it can be useful to repo-specific caches that contain more than just the contents of the last version fetched.

From the near-beginning there was a "corepkg" fetchurl useful for bootstrapping that shipped with Nix.
But this just existed for bootstrapping, and just made a plain old fixed output derivation.

In 2015, Eelco added `builtins.fetchurl` and `builtins.fetchTarball` in https://github.com/nixos/nix/commit/000b5a000f8ceef9d79c7e8a9835fde9a98c367f .
In their first incarnation, they were quite simple.
Their main purpose was just the first problem above, which they solved by fetching without sandboxing at Eval time.

This was a small change, but it still was a layer violation, in that a specific sort of fetching was now given special support.
A more general mechanism, `builtins.exec` was added in https://github.com/nixos/nix/commit/0bb8db257d98a32abde759f4d07d28b5178bd3bf whereby *arbitrary code execution* could be done at eval time.
This never caught on, being the mother of all impurities and frankly a bit terrifying.
A trade of security for layering is not one I would expect most people to make, but I suppose `builtins.exec` did at least serve to emphasize the layering point, and remind us we could keep on searching for better solutions.

Since then, `builtins.fetchurl` and `builtins.fetchTarball` have grown into the "libfetchers" library. and `builtins.fetch*` suite of builtins.
(The library is a also one of the backbones of Flakes.)
Certainly they are used by many people and work around many of the issues above.
But the also are a *dramatic* departure from the layering and policy-freeness of before --- on `master` as I write this `sloc` tells me `libfetchers` is 1908 lines of non-comment, non-blank code, and that code little *but* policy.
The long debates around, e.g., how git submodules should be fetched show that not all the policy is "obvious" or "clearly what one would want" either.

These `builtins` were never as controversial as Flakes, but in hindsight I think this is really when the shift in approach got going.

### The product school --- feature richness

At this point, I think this is a good time to characterize the school of design Nix now subscribes to.

> Batteries included

The metaphor is that an electrical device without batteries is useless --- and that buying such a gizmo only to find it didn't come with the batteries after you left the store can be quite disappointing.
Applied to Nix there is idea that trying out Nix only to find you need some other tool, be it a `nixpkgs-shell` as described above, `niv`, fetching catch system for `builtins.exec`, etc. will frustrate new years, who want an "integrated" rather than "cobbled together" user interface.

In particular, I have personally heard Eelco say many times he doesn't flakes to be some sort of extension, or separate tool.
He wants flakes to be *in* Nix, in the single down, interacted with the single `nix` binary, so they same completely "first class".

Speaking of new years, While neither school of design of design would claim its ambivalent to new users, this school is adamant it cares the most about them, and the alternative demonstrates the biases of experts in pursuing principle that mainly benefit them, without considering the costs that they mainly don't face.
This sort of argument I have seen many, many times, not just with Nix.
I don't at this point many newer users of Nix have never known a world without Flakes, and are quite fond of them.

> Convention over configuration

I honestly don't know if anyone but the Rails people use this, but it seems relevant.
The idea is that there are certain conventional ways to do things, that might be better supported, but nothing is preventing on from going off the beaten path if they want.
Many times Eelco and others have pointed out that new Nix does not *force* the use of Flakes, that the old `nix-*` commands are not about to be deleted, etc.
Still, there is a no doubt some social engineering going on (I do not use that phrase nefariously!) in that Flakes get special support.

- Features like pure eval + eval caching could work without Flakes, but don't.
  (Caching depends on pure eval, but pure eval doesn't inherently depend on Flakes, similar to the "restricted eval" that came before it.)

- Idioms like `packages.${system}` get special CLI support.
  No general mechanism for other idioms to do "plugable" CLI support of this nature.

Certainly Eelco has said trying to design with the general, Flake agnostic version can seem like an extra chore that he doesn't feel very motivated to do.
The point of conventions are to solve problems for users, but also to help shepherd the community into using standard idioms.
From the vantage point of the second, generalizing features to be flake-agnostic can be **actively harmful* not just effort without benefit, because it undermines the strength of the convention.

## What to Do? Multiple Nixes

Hopefully it is clear now, that I consider these two approaches *irreconcilable*.
Any attempt to split Flakes into a separate tool would undermine the "single integrated tool" goal of the second approach.
Keeping Flakes as is, with a promise that you don't have to use it, undermines the first approach because layer violations are still occurring, and Flakes are still "biased" over other approaches, when the lower layers ought not meddle with what upper layer "wins".

So what can we do, other choosing a winner and either stabilizing or abandoning Flakes entirely, and risking schism?
Have multiple Nixes!
Before one scoffs, "isn't that just a fork -- in other words a schism?", note that this is very specific proposal.
Duplicating code across forks leads to a tragic waste of resources, but we don't need to duplicate code to have multiple Nixes.
The key insight is while "Nix within Flakes" must be whole self-contained tool to match Eelco's vision and the second school of design, it can still be *implemented* with good layering.
And indeed, Eelco has so far been fine merging invasive PRs of mine like https://github.com/NixOS/nix/pull/4114 and https://github.com/NixOS/nix/pull/6188 store that separate concerns.

The CLI logic is fairly modular already, and it is not hard to acknowledge it so that subsets of the total functionality can each have their own combined Nix.
Each of these Nixes is *cumulative*, so that the functionality in the more minimal ones is still exposed in the later ones, and no one need install more than one of the Nixes.

Merely having the multiple exes exist in tree is enough to allow CI to "catch" any new layer violations, so we could just keep the more minimal Nixes as glorified tests.
This, however, would be just a token gesture to the first design school / anti-flakes camp.
While there is no reason we can't have multiple versions of Nix indefinitely, the idea is also to have a fairer contest for what future of Nix the community preserves.
Flakes cannot put its best foot forward while still being marked experimental of course, but non-Flakes also can't while they are largely confined to a "legacy mode", or require extra hoops like `--expr`.
Remember, whether or not we do Flakes, we still mostly all want

 - A new, uniform CLI
 - Channels abolished (N.B. Anti-Flakers don't think channels necessarily need to be *replaced*, let alone replaced with something else in Nix itself.)
 - Unambiguous improvements to existing functionality like pure evaluation.

To have a fair referendum, it is important the multiple Nixes are presented as genuine possible futures*, not just compatibility support for the past.
Only then can both factions feel their visions are respected, and schism averted.

# Detailed design
[design]: #detailed-design

## The first split

First split? Huh?
Well, I expect the very notion that we can maintain multiple Nixes without more ongoing effort to be controversial.
Flakes is currently entangled with the other code in `libexprs` and `libfetchers`, and separating it would take some time.
Instead, we can start make the first cut above `libstore`.

`libstore` is the most natural division point in the code today.
Plenty of commands like `nix daemon`, `nix log`, and the `nix store` sub-commands don't care about evaluation at all.
In https://github.com/NixOS/nix/compare/master...obsidiansystems:layer-nix (TODO open draft PR!), I have split out a store-only binary with some of these commands.
We will finish it off with as many commands as are reasonable to include, and merge it.

## The second split

Of course, the schism that took us here is flakes, and so once the splitting approach is proven, we should separate Flakes from `libexper` and `libfetchers` into a `libflakes`, and make another Nix that just links the former two.
While splitting those libraries will be some work, the CLI split should be very rote an easy after with the already-proven approach.

## Obligations per split

### CI

Every more minimal version of Nix should be built and tested in CI runs, for release and PRs.
Less CI than full maximal Nix makes this an unfair contest, and defeats the main goal of trying to prevent a schism.

### Tests

The current test suite uses eval for most things, and in fact enables Flakes globally despite it being experimental.
That is unacceptable while flakes is an experimental feature, but not good practice anyways.
We should instead break up the tests so that each version of Nix is well tested, and each cumulative version of Nix can pass the test suites of the Nixes it subsumes.

### Manual

Each Nix should get its own manual.
Of course the manuals can share sections, so we aren't duplicating work.

### Website

All versions of Nix should be presented for download on the website.
This should be just like how Plasma, Gnome, and headless installer images for NixOS are all offered.

## Stabilization

With the split, I and the other anti-Flakers will be a lot less anxious about the looming stabilization of Flakes.
So we could go straight away to stabilizing that, without acrimony.
However, I think with the split we have an opportunity to do better.

The new CLI is also in need of stabilization, but so long as it and Flakes are quite bound together, it is hard to do so.
With this layering, all of the sudden all of these other concerns CLI concerns are back ready for discussion, without it devolving into a big flame war.

Questions like

- Logging
- store paths at end of build?
- Should commands like `show-derivation` should use `--json` by default
- Flat vs hierarchical commands

all deserve some discussion, because we really should be looking to get rid of the old CLI soon that confuses so many people.

With the splitting we stabilize the general feel of the CLI, along with the exact spec for the more minimal Nixes, before we stabilize Flakes.
We should do that first to bring the community back together for some healthier decisions making!
The layering will make clear that the results of the first around of stabilization should not

# Examples and Interactions
[examples-and-interactions]: #examples-and-interactions

## No schism!

The most important effect is that the community stays together, with no one feeling their vision for the future of Nix has been relegated to a dusty corner.
All else is secondary.

## Maximal Nix unlike today

I cannot emphasize this enough, but the interface of maximal store+exprs+flakes Nix remains *exactly* like today.
In particular, lower level commands can be used with higher level "installables" (arguments), so e.g.
```
nix store show-derivation flake#bar
```
will still work.

## Good tasks anyways!

Lots of the plan above I think is good work we should be doing anyways, regardless of whether we split Nix.
If you believe this, then the "cost" of this RFC is a lot less.

### Tests

Splitting the test suite per natural layer of the implementation is good work because it combines the specificity of unit tests with the real-world-ness of integration tests.
"entire kitchen sink" tests make it harder to narrow down root causes of failures.

Also note, to be able to test the store-only Nix, we will probably want a "JSON to drv" command that is the opposite of `show-derivation` to make derivations more easily.
This is also a good idea with or without PR, because for https://github.com/NixOS/rfcs/pull/92 we want to allow external programs to make derivations efficiently and ergonomically.

### Manual

The current manual I believe jumps across layer of abstractions way to much.
A so-layered manual would complement the "bottom up" approach of Nix pills:

 - The store-only Nix manual would lay out all the core concepts.
   We could even rewrite the earlier Nix pills to use the "JSON to drv" command we will likely need for the tests.
   While not very ergonomic, this Nix would be a crucial tool for anyone trying to deeply understand the foundation.

 - The store+exprs Nix manual can spend some time introducing functional programming, laziness, in general.
   `nix repl` and various debugging techniques should get lots of attention.

 - The store+exprs+flakes Nix manual would be *more free* than to gloss over how things work underneath the hood,
   It could focusing on idiomatic usage of Nix, and trying to get common problems done without needing to understand everything that's going on.

Based on the perennial feedback Nix threads on e.g. Hacker news receive, I think this would *hugely* popular.
Some folks just try to do things, and then get hopefully confused.
Other folks try to learn what's going, and find the details and core concepts maddeningly undocumented.
The 3 manuals correspond to different learning styles, and a different familiarity with various things (sysadmin stuff vs functional programming, perhaps.)
People will self-select, and end up a lot happier.

## Security

The daemon is a privileged process.
Even if with upcoming changes it shouldn't need root, it does tasks like administrating OS sandboxes correctly.

## Collaboration

I am a big believer in there being social ramifications of layering.
Specifically, with the layers of Nix taking more of an identity and *life* over their own, there is more chance to onboard people are who are interested in specific layers of the project.
Layers allow more people to work in parallel, while at the same time setting clear boundaries on what each layer is *for*, and thus what sorts of project in scope and out of scope.

Nix itself has long been far and away the least community-driven project in the NixOS ecosystem.
As the beating heart of it all, I think that is to some degree inevitable.
But, with layering I think will finally be able to broaden community involvement without Eelco feeling lake it's going off the rails.

To repeat the big idea behind all of this, layering means factions don't need to fight to stay afloat, less confrontational factions makes for healthy pluralism, and healthy pluralism makes for wider collaboration.

# Drawbacks
[drawbacks]: #drawbacks

Honestly, I think the biggest drawback will be that the Flake faction feels it is stuck doing some extra chores to appease the anti-Flake faction.

I think the problem is because the anti-Flake faction seems weaker than it should be because it is unclear what we want --- and many of us are "stuck" on Nix 2.3.
If we can rally around this compromise, we can also put in the bulk of work of implementation, making the peace offering of sorts, to pay for our continued existence as a full-fledged part of this project.

# Alternatives
[alternatives]: #alternatives

I really don't think there is alternative to this plan that doesn't leave a lot of people sad, demoralized, and angry.
That is why I took the time to write this very long RFC.

There a few variations, however:

- Because `libfetchers` is also a layer violation of sorts (though `builtins.exec` is not a satisfactory generalize equivalent at this time), we could also make a `libstore + libexers`-only Nix too, sitting between the store+eval and store+eval+flakes Nixes.

# Unresolved questions
[unresolved]: #unresolved-questions

## Names

Should we call all the binaries Nix because they compatible (either something is not recognized or it does the same thing).
Or should we come up with new names?
"Nix expression language" still deserves a new name regardless :).

## Nixpkgs

To what extent Nixpkgs would use flakes post stabilization is unresolved.
I am less worried about that, because I don't think Nixpkgs can be split up very much if at all.
If other Flakes depend on it, but it doesn't depend Flakes, there is no issue.

# Future work
[future]: #future-work

Here comes the fun part!

While preventing a further fracturing of the community is and remains the most important goal,
we can turn the argument around and think about what schism have *already* occurred.
Tvix and GUix, as mentioned in the front-end are good places to begin.

## Tvix

Tvix is a basically a focus on refining all the pre-flake parts of Nix -- it is no coincidence it is a fork of 2.3, more or less.
Tvix should do basically what store+expr non-flake Nix should, and wants to be a drop-in replacement for that (and what's needed for Nixpkgs, which may or may not eventually exceed that).
By splitting out more minimal Nixes, we formalize exactly what sort of interface that is, not as a historical snapshot "2.3", but as a *living* standard.

Frankly I am curious whether Tvix would have happened had we had this exposed, advertised layering from the get-go.
But, trying to undo history I think is not the point.
Tvix can have a relationship with store+expr Nix a lot like that between NeoVim and Vim.
Especially the ideas of trying to work with more off the shelf components (e.g. containerization standards, RPC protocols) I think have the potential to be welcomed back over here in the original project.

## Guix

Guix is a bit different, in that they don't want to share any code with us on principle as all the C++ should be replaced Guile.
In some respects, they are the most "against layering" because the whole point is the synergy between Guile as the implementation language, package planning language, and plan executing language.
In other words, where we use C++, Nix, and Bash, they use Guile, Guile, and Guile.

Still, I think there is potential for collaboration.
My long term vision for the store layer is that I want to see it deployed in every build firm and compute cluster, replacing the likes of Slurm, and sold as a service by every cloud provider.
The "derivation language" should be the standard language for batch jobs, full stop.

Guix is, if anything, more interested in HPC than we are.
See https://hpc.guix.info/ for example.
Insofar as we share the same goals, and same basic message about reproducibility, maintaining a common, living standard is to both our benefits so Nix and Guix users can together lobby cluster admins for one jobs-processing deployment.

There would still be separate Nix and Guix implementation to this common interface, perhaps with their own unstandardized extensions.
MPI and OpenMP are both widely-used interfaces with multiple implementation in wide use, and so I consider this sort of standards-writing coalition building a proven way to get things done.

I should say that I first reached out to some Guix folks roughly a year back about this sort of standardization in the context of the CA work.
They weren't super interested at the time, rightly calling me out on making an overture on shared design/standardization after the CA experiment was mostly implemented and so things were set in clay, if not stone, on our end.
Still, implementing the layering is a more concrete gesture towards collaboration than just sending an email.
I maintain optimism it could go better a second time.

## Other frontends

There are lot of folks interested in trying out other "frontends" to Nix.
[Dhall](https://dhall-lang.org/)'s targeting of Nix is the most notable example, but there are others.
Those projects would get a big lift from a store-only Nix, as it is basically a declaration that derivation language intends to be stable and amendable to outside consumption.

## Final thoughts

It would be easy to imagine splitting this RFC into two: one for store-only Nix, and one for eval nix without flakes.
The store layer, merely by being lower, offers a wider space of possibilities yet to be explored on top, and that is fun and exciting.
But I have chosen to couple them together, to couple preventing bad things and unlocking good things, because it is fundamentally the same process for both splits.

Similarly, the possibilities of collaboration with off-shots of Nix that already happened are easier to imagine,
because of the other projects are concrete things that actually exist.
Still, it is much easier to maintain contact with projects that fork than try to reestablish it later.
If we do have a Flakes schism, maybe one side will whither and disperse, but maybe we will end up with two separate self-contained communities.
Then we will be in the *exact* same position we are in today with respect to projects like Tvix and Guix.

With the layerism and pluralism plan, we should be able to have the best of both worlds, where projects are free to try their own things, but we always remain on friendly and collaborative terms.
Rather than a single Nix community, there should be concentric communities around each layer.
As the layers accumulate in one direction, the communities should accumulate in the other.

The world of open source software often seems to fragment and churn as rapidly as it grows, with all sorts of fads coming and going on one hand, and yet ever accumulation of the cruft of past ideas deep in the bowels of dependency graphs.
Yet in recent years, there is all sorts of interest in reproducibility, software "supply chains", functional programming, and whatnot.
We can be one idiosyncratic thing, and maybe achieve permanent dominance.
Or we can be another fad -- people often say things like "Nix is the good idea that something else eventually will make stick".
Layering is a defense against being to rigidly any one thing, while also a chance to build coalitions with like-minded projects, including those that would only come into being because the layering exists.
We are at the juncture where it should be both the safest and boldest step forward.
