# Week 3: It all will fall right into place

This week onwards my blog shall be divided into two sections. The first half will follow a fixed structure, where I report on the progress on my project. It will be technical and rooted in the time of me writing this. The second half will be more evergreen and personal. It will focus on what I learnt, and be a reflection of my experience.

Why? My mentor, Christian, has shown me that blog posts are how mentees usually give updates to the Git project. I share my progress and what's blocking me, and what I plan to do next. People on the Git mailing list will read this, and help me out where needed. So the first section will enable that feedback loop, and everyone can learn from this, hopefully.

# Project Progress

### What have I done since the last post

Recap: At the end of week 2, I sent [my first patch](https://lore.kernel.org/git/20210528081224.69163-1-raykar.ath@gmail.com/) based on Shourya's old patch.

Since then, I got  [suggestions](https://lore.kernel.org/git/CAP8UFD01-VJpUEGg3cEG7X=xU0KCv1AEgq2n_qhk=U+rXV5mvA@mail.gmail.com/) from Christian that made my code more readable and concise.

I also finished my next patch, which was to introduce another helper, called `git submodule--helper add-config` that handles the configuration on adding the submodules.

Back in week 2, I had said this:

> I have not yet fully understood how config handling generally works, and there's a case of a valueless value that, from what I can tell, was not handled in Shourya's patch.

I am glad to say, I _mostly_ understand the config part of the code now. My initial difficulty back then was because of a config variable called `active`. Or more specifically there are two configuration variables with that name, one called `submodule.<name>.active` and `submodule.active`. Past me had missed this distinction, and thus had sent my two brain cells on a motorized merry-go-round. Now I know – the former is a switch to turn on a submodule, and the latter option can be a series of pathspecs (like regex, they are pattern matching strings) that activate all submodules matching a pathspec. An empty pathspec, ie, valueless value, is an error in configuration. I added a warning so that when a submodule is activated, the user knows that the pathspec has not been used and Git has activated the `submodule.<name>.active` switch instead.

There is still this one part of the code that I do not fully understand yet. This comment:
```shell
	# NEEDSWORK: In a multi-working-tree world, this needs to be
	# set in the per-worktree config.
	if git config --get submodule.active >/dev/null
```

To complete my code translation patch, I did not need to understand the intricacies of multi-working-tree configurations. I just left that comment in my new C version, and let it be a black box. I'm still curious about why that comment was left there. I did read about git worktrees this week in an attempt to understand what's going on, but I stopped before I could be more thorough about it in the interests of getting my translation done.

My not-so-well-read guess: Some users want certain submodules active in one worktree, but not the other. For that, there presumably exists a per-worktree configuration, and the current implementation just assumes a configuration that applies to the full repo. Changing this is definitely a patch for some other time.

After I finished [this second patch](https://lore.kernel.org/git/20210605113913.29005-3-raykar.ath@gmail.com/), I had to combine them as a [series](https://lore.kernel.org/git/20210605113913.29005-1-raykar.ath@gmail.com/), because this patch depended on the first one for a particular `struct` so I could not make them a 100% independent.

I had initially developed them as separate branches, in the hopes that they would be independent, and could be applied in any order. Once it was becoming clear that would not happen, I thought of combining those two branches on another branch (that would hold all my patches so far).

### A painful merge

Getting those two branches was chaotic. I'll simplify things to get to why I didn't have a gala time.

My commits (here, A, B) looked like this:
```
--x----x
       |\  (A)
       | `--x     [patch-1]
        \
         `--x     [patch-2]
           (B)
```

I wanted an end result that looked like:
```
--x----x----x----x     [cumulative-patch]
       |   (A)  (B')
       |  [p1]
        \
         `--x   [p2]
           (B)
```

That is, to preserve the original branch states of (A) and (B), so that I can preserve 'snapshots' of how those changes were, when applied independently.

Simple enough, I thought. Just fast-forward merge (A) on a new temporary branch based off master, and then, uhh, cherry pick and apply (B) on top of the newly merged (A). Resolve any conflict in this stage. That way I get (A) and (B) in their pristine individuality, as well as a third branch that combines both the changes.

I thought the merge conflict would be fairly easy to deal with.

A file in (A) had a structure like this:
```
XXXXXXXX
XXXXXXXX

YYYYYYY
YYYYYYY
YYYYYYY
YYYYYYY
```

The same file in (B) looked like this:
```
XXXXXXXX
XXXXXXXX

ZZZZZZZ
ZZZZZZZ
ZZZZZZZ
ZZZZZZZ
```

ie, they have the `XXX...` part in common.

I wanted the end result (ie, C') to look like:
```
XXXXXXXX
XXXXXXXX

YYYYYYY
YYYYYYY
YYYYYYY
YYYYYYY

ZZZZZZZ
ZZZZZZZ
ZZZZZZZ
ZZZZZZZ
```

I thought Git might just be able to handle that merge with little to no conflict. When I ran my cherry pick though, I got this:
```
# assume the appropriate conflict markers were placed
XXXXXXXX
XXXXXXXX

ZZZZZZZ
YYYYYYY
ZZZZZZZ
YYYYYYY
YYYYYYY

ZZZZZZZ
YYYYYYY
YYYYYYY
ZZZZZZZ
```

All my functions were tangled together in that file. I tried switching merge strategies ('recursive' gave slightly better results) but it was still a messy conflict.

I thought there would be a lot of hair pulling, but Emacs' smerge had a pretty useful feature, that lets you combine the current conflict region to the next. While it did not solve all my problems, it greatly reduced it to a manageable state. I still wonder how non-Emacs users deal with situations like these.

# Reflections

This week taught me a lot about how to structure my work, especially when it needs to be divided into reviewable patches.

Before I end this (somewhat lengthy) post, I'll leave one last blooper.

### Blooper of the week

In the initial attempt of trying to combine my two patches, I performed a rebase that had a conflict. I did a shoddy job of it, and ended up accidentally wiping out _all_ the changes that I had made since the previous day. None of those changes had been pushed.

I realised that this was the day that I had been preparing for all my life. The day I mess up so spectacularly that I have to use Git's magic time machine – `git reflog`. The great thing about Git is I can work rather fearlessly because I know Git never actually deletes or modifies your commits anytime soon[1]. When you lose your work, it is because you have only lost your reference to it[2]. The reflog is kind of a like a transaction log that notes all the actions performed on the references. That means I can always look at this log and find a handle on the commit that I lost after a bad rebase, and let me restore my work.

I won't show a tutorial for how to use it, because better ones will exist. A nice and concise one that gets you to what you usually want is [here](https://ohshitgit.com/#magic-time-machine).

So what do I learn from this blooper?

- Push your changes more often
- Check your resolved conflict before continuing the merge

I doubt that will be the last time I make those mistakes though. So I'm glad there's the reflog.

[1] Git performs the occasional garbage collection that deletes unreachable references, something that is not a big deal if you very recently messed up.  

[2] To put it simply, references are things that point to your commit objects – branches, tags and the HEAD pointer.
