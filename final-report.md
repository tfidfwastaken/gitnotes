# Porting Git Submodules to C: The Technical Report

## The Project, briefly explained

Git has historically had many components implemented in the form of shell scripts. This was less than ideal for several reasons:

- Portability: Non-POSIX systems like Windows don’t play nice with shell script commands like grep, cd and printf, to name a few, and these commands have to be reimplemented for the system. There are also POSIX to Windows path conversion issues.
- No direct access to plumbing: Shell commands do not have direct access to the low level Git API, and a separate shell is spawned to just to carry out their operations.
- Performance: Shell scripts tend to create a lot of child processes which slows down the functioning of these commands, especially with large repositories.

The goal of this project is to complete the conversion of the remaining parts of `git submodule` to C, namely, the `add` and `update` commands. If possible, I intend to even get rid of the shell script called `git-submodule.sh` entirely, which currently calls `submodule--helper` to perform most of the business logic, and instead make submodule a proper builtin in pure C.

## What has been completed

My project can be broadly divided into three components:
1. Conversion of `submodule add`.
1. Conversion of `submodule update`.
1. Submodule commands have their entry point in a shell script called `git-submodule.sh`, which in turn call some C helper commands that do the real work. My goal is to finally remove the shell script middleman, and simplify the architecture.

Here are the statuses of each component.

### Conversion of `submodule add`:
* Partial conversion [merged to master](https://github.com/git/git/commit/10f57e0eb9070bf00c45def2980a47eacbae8316) (add-clone).
* Rest of the conversion has been `seen`, but not merged. Links to patches: [`ar/submodule-add-config`](https://lore.kernel.org/git/20210806140431.92018-1-raykar.ath@gmail.com/) and [`ar/submodule-add-more`](https://lore.kernel.org/git/20210810114641.27188-1-raykar.ath@gmail.com/).
* Reviewing seems to have settled, consensus reached?

From "What's Cooking in git.git", 16 Aug 2021:
```
* ar/submodule-add-config (2021-08-10) 1 commit
 - submodule--helper: introduce add-config subcommand
 (this branch is used by ar/submodule-add-more.)

 Large part of "git submodule add" gets rewritten in C.

[...]

* ar/submodule-add-more (2021-08-10) 10 commits
 - submodule--helper: rename compute_submodule_clone_url()
 - submodule--helper: remove resolve-relative-url subcommand
 - submodule--helper: remove add-config subcommand
 - submodule--helper: remove add-clone subcommand
 - submodule--helper: convert the bulk of cmd_add() to C
 - dir: libify and export helper functions from clone.c
 - submodule--helper: remove repeated code in sync_submodule()
 - submodule--helper: refactor resolve_relative_url() helper
 - submodule--helper: add options for compute_submodule_clone_url()
 - Merge branch 'ar/submodule-add-config' into ar/submodule-add
 (this branch uses ar/submodule-add-config.)

 More parts of "git submoudle add" has been rewritten in C.
```

### Conversion of `submodule update`:
* The first part (run-update-procedures) has been `seen`, reviews still ongoing. Link to patch: [`ar/submodule-run-update-procedure`](https://lore.kernel.org/git/20210813075653.56817-1-raykar.ath@gmail.com/)
* The second part of this conversion has not been sent to the list yet, and needs reviewing. Source code for this change can be found [here](https://github.com/tfidfwastaken/git/commits/submodule-update-list-1).

From "What's Cooking in git.git", 16 Aug 2021:
```
* ar/submodule-run-update-procedure (2021-08-13) 1 commit
 - submodule--helper: run update procedures from C

 Reimplementation of parts of "git submodule" in C continues.
```

### Conversion of `git submodule` command to C builtin
* The basic implementation works, but not yet complete. Only `status` has been converted to a pure C builtin. The source code can be found [here](https://github.com/tfidfwastaken/git/commits/submodule-make-builtin-2).
* Not yet sent to the list. Depends on the above two parts being merged first. Once those are merged, this series is quite easy to complete.

### Miscellaneous

There were other tiny contributions that I made to Git as well.

- [Add userdiff support for schemes](https://lore.kernel.org/git/20210408091442.22740-1-raykar.ath@gmail.com/): This was my microproject that was required for my GSoC selection. I taught Git's diff machinery to be smarter about Scheme and Racket code, and provide better word diffs and function context headers.
- [Documentation update: linking the IRC to Libera Chat](https://lore.kernel.org/git/20210608190612.72807-1-raykar.ath@gmail.com/).
- [Other](https://github.com/git/git-scm.com/commit/b5ecccb292c3202bdf298be90a6061d9fc23b950) [doc](https://github.com/git/git.github.io/commit/bda6f861914e306e8100c046cdca95de2c977681) fixes.

## What's next?

I will still carry on with the reviews for the patches that are in flight. I will also continue sending the many patches that I am still holding on to. Let's hope we can finish off the submodule conversion effort that has been going on for over five years now!

## Technical Challenges

Here are some things I found challenging while working on this project.

### Structuring Patches

This was a large overarching theme across all the three parts of my work. Projects like Git do not operate by developers merely churning out a boatload of code. There needs to be structure to the changes that are sent, so that they can be easily reviewed, and held to high standards. This would be hard to do if I had done the conversion in one single run and sent a giant series to the mailing list.

Given the complexity of the existing submodule code, it was not trivial to break up the changes into convenient bite-sized pieces. There was always a tension between "is this series too big?" and "is breaking this change into multiple series making it to complicated to follow the changes?"

I would not be surprised if more than half the times I was asked to modify my changes from mentors and listfolk were because of reasons related to how I structured my patches. This taught me how effective communication makes software scale—your changes should tell a story that's easy to follow, so that the code can easily be picked up by others by a mere examination of its commit and list history.

### Finding equivalents to shell invocations

Since my project was converting a lot of shell code, it was not always easy to find an equivalent in the C API of Git, especially for all those `rev-list` calls that do a lot of git-fu to retrieve commit information. There was always an escape hatch—we could fork a process and run the shell invocation. But this was not ideal, and I tried to avoid it as much as possible.

### Recursion

The `submodule update` command has the `--recursive` flag, which updates nested modules recursively down all paths. So roughly speaking, the implementation did this by running the update for the current worktree root, and then running update again, but by switching the root path to inside a submodule in a recursive process of `update`.

I quite like the elegance of recursive functions, but I can't say the same for recursive _forked processes_. The shell version was still smooth because the script had its own setup, and the way of handling the environment that was not too problematic. But translating this to C introduced a bunch of problems, like [certain environment variables not being updated properly](https://atharvaraykar.me/gitnotes/week8#path-pains). I had to apply a small band-aid fix that made it work nicely for me. It was an overall win, because it led to me discovering some much needed refactoring for some repeated code, and it also [helped me save a subprocess spawn](https://atharvaraykar.me/gitnotes/week9#passing-the-superprefix-explicitly).

Ideally we should not be forking processes recursively at all, because of how expensive and finnicky they are. Unfortunately, the Git submodule API is not quite there yet to make a more elegant solution happen. A lot of the configuration functions still operate only on the global `the_repository` objects, which makes recursive submodule operations not work correctly if done in the same process, as it will mess with the state of the root repository state. But we are getting there. The pieces are present, it just needs to be assembled. Maybe this could be a good idea for a future GSoC/Outreachy project?

## What I learned over the course of this project

This project confirmed a belief that I held. Learning isn't sitting in class. Doing is learning. I joined this project as an impostor who knew nothing about anything. But now I know a lot more things, all forced by the requirements of my projects. Here's the brief:

- Programming to be understood
- How to write for a large C codebase
- Mailing list development—I was amazed by the fact that I worked on this for three months _without scheduling a single meeting_ with anyone. Email is powerful. You can go a really long way with concise email exchanges in mature Open Source projects like these. It felt liberating especially because of how a large chunk of my lifespan has gone into listening to people drone on and on for getting anything done.
- Investigation skills. I started to actually experience the difference between _programming_ and _engineering_ and how it goes hand-in-hand. I definitely spent way more time reading and analysing other's code and my problem than I did for coding the solution.
- Debug-fu, especially in a scenario where you have a lot of forking going on
- Shell scripting (at the time I submitted the proposal I knew shell only on a surface level)
- How to name things and design things better
