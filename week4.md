# Week 4: On obviousness, and being a mere mortal

# Project Progress

### What I have done since the last post

I got reviews from several developers. Danh [reviewed my patch](https://lore.kernel.org/git/YL9jTFAoEBP+mDA2@danh.dev/), and suggested a change that eliminated the need of two helper functions I wrote to parse tokens. I did not really need to do that at all. This situation reminds me of what Donald Norman had to say about getting to the root of problem solving:

> Harvard Business School marketing professor Theodore Levitt once pointed out, “People don’t want to buy a quarter-inch drill. They want a quarter-inch hole!” Levitt’s example of the drill implying that the goal is really a hole is only partially correct, however. When people go to a store to buy a drill, that is not their real goal. But why would anyone want a quarter-inch hole? Clearly that is an intermediate goal. Perhaps they wanted to hang shelves on the wall. Levitt stopped too soon.
>
> Once you realize that they don’t really want the drill, you realize that perhaps they don’t really want the hole, either: they want to install their bookshelves. Why not develop methods that don’t require holes? Or perhaps books that don’t require bookshelves. 

I didn't need to find a good way to parse tokens. I needed to display fetch remotes.

Junio also gave me a helpful review, and [rightly pointed out](https://lore.kernel.org/git/xmqqim2l3p92.fsf@gitster.g/) the needless parsing and unparsing that was being done to call `module_clone()`. I am working on factoring out that function to separate the parsing of flags from the core logic. It's almost done, and a reroll should be out tomorrow.

### Current obstacles

There is this bit of code that has been giving me a hard time:

```shell
if test -z "$force"
then
	git ls-files --error-unmatch "$sm_path" > /dev/null 2>&1 &&
	die "$(eval_gettext "'\$sm_path' already exists in the index")"
else
	git ls-files -s "$sm_path" | sane_grep -v "^160000" > /dev/null 2>&1 &&
	die "$(eval_gettext "'\$sm_path' already exists in the index and is not a submodule")"
	fi
fi
```

Usually there's an equivalent C function I can find to do most things so far, but to my surprise, I could not find a function that checks whether a given path already exists in the index. It really needs to satisfy these criteria:
 - It should check for file entries in the index
 - It should check for directory entries (ie, directories that have
   tracked files)
 - It should check for unmerged entries (ie, stage number > 0)

I could find a lot of functions that _almost_ did what I wanted to do, but always did something extra, or fell short in other places. I shall describe my findings so far, but I should warn you, the next few paragraphs are a brain dump. Apologies if my legibility has taken a hit there.

My first approach was to see what Shourya's [previous](https://github.com/periperidip/git/blob/ff3725adbfe71303467ab0dfde57d26067e37f4f/builtin/submodule--helper.c#L2797-L2818) [attempt](https://lore.kernel.org/git/20200826091502.GA29471@konoha/) was.

He used a combination of `cache_name_pos()` and `directory_exists_in_index()`. With this approach, the downside as Junio pointed out is that [it cannot account for unmerged entries in the index](https://lore.kernel.org/git/xmqqo8ldznjx.fsf@gitster.c.googlers.com/). The other potential pitfall I found with this approach is that it isn't really using what `ls-files` was using in the first place--which is what a faithful implementation should ideally do.

There's also `index_dir_exists()` and `index_file_exists` that sounds like what we want, but unfortunately they only seem to be for the case when core.ignoreCase is set to true.

There is also `repo_read_index_unmerged()`, but looking at the comment for it, it appears to be actually modifying the index, and dropping index entries to stage 0.

Finally, I thought of looking at what `git ls-files` is actually doing.

And it's doing [a lot](https://github.com/git/git/blob/211eca0895794362184da2be2a2d812d070719d3/builtin/ls-files.c#L607). The part that seems relevant to our problem is that it parses the pathspec, and then iterates through every index entry via 'repo->index[i]', gets the fullname, and calls a function that further down the stack, ends up calling `match_pathspec()`. This updates a byte called `ps_matched`, that contains information about whether a file matched any pathspec item. This is sent to `report_path_error()`, that figures out which entry it was and prints accordingly.

In summary, the actual working of `ls-files` is very different from what was attempted in the older patch.

The only way I can think of right now that would work, is to write my own helper, which goes through the index structure, and calls `match_pathspec_item()` (where the pathspec would be made from `sm_path`) for each index entry, and the moment it provides a positive match, we return, knowing that sm_path does exist in the index. This should cover tracked files and directories, as well as unmerged entries (ie, with stage numbers > 0).

If Git developer reading this, happens to know something that will meet
the criteria I mentioned before this brain dump, do let me know :)

**Update**: I had communicated this problem to my mentors and while I was writing this, [Kaartic Sivaraam](https://cs.stanford.edu/~knuth/news16.html) pointed me to a [previous instance](https://github.com/git/git/commit/d553f538b8#diff-c919215a006bdf88661fad082403f9242aae66dece60ec3463d2cc573ba6383cR1265-R1286) of how this conversion had been handled. So far it looks like the most promising way to do it, although there are a few papercuts. Thanks Sivaraam!

### What's next?

Once I send the next patch to refactor `module_clone()` and call the helper without un-parsing the flags, I'll get straight to work on a fully porting over the core logic of `cmd_add()` to a `submodule--helper.c:module_add()`, and exposing it via an `add` subcommand. In order to do that, I'll have to overcome the obstacle I described above, and currently I feel like I am converging on a solution.

# Reflections

### i am just ok.

I'm plenty privileged to be given a chance to work on a project that is used by millions of users. This was extraterrestrial levels of incomprehensible to me just an year ago.

As a consequence of working in a project of such importance, the developers I am working with, and the ones I see on the mailing list regularly, come of as a little bit... godlike. Unlike the various Gods, Git developers are approachable. But to someone like me, who's never worked any kind of job before, and has never approached programming as a craft, the technical ability of these folks are on another level. It really does feel to me like they could do my two-month long project in 2-3 weeks or lesser.

This of course, opens up a dangerous rabbit-hole of thoughts. The feeling of being the least competent person in the room is not a pleasant one. So to remind myself, and those of you reading this who might be in a position similar to mine in the future:

- My understanding is that the Git community (or any other community for that matter) did not choose you to work on this problem because you will be extremely efficient. It is true that they can do what we are doing in a fraction of the time. The reason they are doing this is to grow and diversify the community – an investment into their future. Your role is not an additive role, it is a multiplicative one. Use the platform you are given to help your peers break into Open Source, and build their careers, while making software better for everyone.
- Reframe your situation (a classic Stoic trick). Yes, you will be less accomplished and technically weaker than others in the community, and that means you will improve in a way you couldn't otherwise. Being the smartest person in the room will not make you smarter, but being the stupidest one will level you up greatly.

I'd like to emphasize that the second point is why I am so excited to work with Git. I almost feel like I have learnt more useful skills in these four weeks than I would at an year in a University (of the level in my vicinity). This whole thing is unusually out of my comfort zone, and yet I feel at home doing this. The technical challenge is something that I have been craving for a really long time, and this is really satisfying that for me.

Really, a big thanks to the Git community for doing this. There are many students across the world with poor access to good quality education (knowledge is very abundant, thanks to the internet) and high quality hands-on experience. Going out of your way to mentor students in a project of this quality is a big deal.

### On obviousness

It's easy to assume what is obvious for others. Near the start of the week, I noticed the IRC links to #git-devel in the MyFirstContribution documentation were out of date. I sent a patch to update it, and CC'd Emily, who wrote that excellent guide (a huge reason as to why I am here writing this blog at all).

I was also being a lazy oaf. The old link directly opened the webchat with #git-devel as the channel in the input field. The webchat used by the new network (Libera Chat) also had a similar field, but I could not figure out how to share the link with that field pre-filled. So I just thought, "Well, it should be obvious, I suppose". A dangerous thought.

Emily was quick to correct me on [this](https://lore.kernel.org/git/YL+ndHSLowy%2FqyZV@google.com/). It occurred to me the idea of things being obvious is extremely relative, and people's knowledge of things do not overlap. 

This reminds me of a [paper](https://link.springer.com/article/10.3758/s13428-012-0307-9/tables/1) that supports this view really well. A lot of what is considered "general knowledge" is not known to most people. In this research paper, a pop quiz was conducted with 300 questions, of increasing difficulty. Only for 45 of those questions did the majority answer correctly. This is not because people are stupid, it is because knowledge is highly contextual and dependent on a person's background. You may find a lot of questions in that obvious, and be surprised to know that not many knew the answer to it. And this is coming from a narrow population of American University students. Apply this to the diversity of this whole planet, and I would imagine the amount of common knowledge would be really low.

This is an important lesson – _never_ assume something is obvious just because it is known by you or the people in your circles.

Which also brings me to a practice that is common in the Git community, especially when reviewing patches. I am guilty of this as well.

Often when soemone makes a typo, people in the community correct it by replying like this:
```
> Often when soemone makes a typo
s/soemone/someone/
```

Now that I think of it, this form is not obvious at all. In fact I am in the minority in using vi keybindings (or `sed`?), in my generation. When a correction like that is made, it assumes knowledge about this specialized tool, one that is not essential to actually participate and contribute to Git.

It's not the end of the world if you do correct typos like this, but using a different form is a simple step to help a new contributor feel less intimidated by an experience that is already out of their comfort zone.

I am not advocating sacrificing internal culture for diversity. A common lingo helps bring communities closer, but it is equally important to give space to people coming in from diverse backgrounds to adopt that lingo (and over time, expand the lore with their unique world view).

So if someone new does hop on the mailing list, I have decided that if I want to correct a typo, I will try to make it clear as to what that form means.

Maybe something like this?
```
> Often when soemone makes a typo
I see a typo there, it should be 'someone'.
s/soemone/someone/ ;-)
```

It should go without saying, but this is ultimately a small step. There almost always are much bigger and more actionable things to improve contributor involvement. In my experience, some communities do the small and easy steps like these, and stop at that, because it is deceptively easy to fall into a pattern of thinking that they have removed impediments to contribution.

Real improvements to thriving, inclusive communities come from a lot of effort and hard work (like a mentorship program).
