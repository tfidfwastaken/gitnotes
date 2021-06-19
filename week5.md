# Week 5: A nice round-ish number

## Project Progress: What's Cooked in my personal git.git

Since 5 is a nice round-ish number, I thought now would be a good time to summarise *everything* I have done so far, and then talk about what's cooking in my personal git.git.

My short-term goal is to convert the [bulk of `cmd_add()` of `git-submodule.sh`](https://github.com/tfidfwastaken/git/blob/master/git-submodule.sh#L89-L308) from shell to C. By the 'bulk' I mean all the work after parsing those flags should be done by a call to a C helper, which will be called `submodule--helper add`. This pattern has already been applied to other converted functions such as `foreach`, `init`, etc.

As you may have observed, that is a lot of lines to be changed. I had decided to avoid doing it in a big-bang kind of way where I send the whole conversion as one series. Instead, I started by converting parts of the shell code into various smaller helper subcommands written in C, so that I can get feedback on whether my general approach is correct.

I first introduced the `add_clone` helper that converted the part that handles the cloning when you add a new submodule. [This link](https://github.com/tfidfwastaken/git/commit/944a07ff33d74c4357a467f6acafe4abae905b74#diff-6471df7489a7698cb8d1be613dba970edbc6c5460b296c7e256c809beecb8cfd) shows the portion of the shell code that has been converted.

I then introduced the `add-config` subcommand which converts the part which sets up the configuration of the submodule when you add it. [This link](https://github.com/tfidfwastaken/git/commit/383f0b6217dc9aa1e04554e11f5d90c73169795e#diff-6471df7489a7698cb8d1be613dba970edbc6c5460b296c7e256c809beecb8cfd) shows the portion of the shell code that has been converted.

These two conversions have been sent as a patch series, whose latest iteration can be found [here](https://lore.kernel.org/git/20210615145745.33382-1-raykar.ath@gmail.com/). I shall wait for reviews on that series.

That brings me to what's cooked this week, but not yet sent on the list.

I have a new patch series ready which basically completes the short-term goal I stated at the start of this section. It is built on top of what I have already sent on the mailing list, it just needs a little bit of restructuring. You can find my branch for this [here](https://github.com/tfidfwastaken/git/commits/submodule-helper-add).This was by far, the most challenging part of my work up to this point. You may have got a hint of it [last week](week4.md) when I was trying to find a good replacement for a particular `ls-files` invocation.

The way I have solved it is by rolling up my sleeves and doing the dirty work--I traced out what every bit of `ls-files` was doing and how it found whether a path was already in the index in some form.

While doing that, I realised that `ls-files` was doing a lot of work that was not needed at all (the command has a lot of options and switches and I just needed one specific *side-effect* of that command, which checked the index while trying to print the names of files).

I also consulted [Paul's patch](https://github.com/git/git/commit/d553f538b8#diff-c919215a006bdf88661fad082403f9242aae66dece60ec3463d2cc573ba6383cR1265-R1286) that did a similar conversion, and it closely matched the solution I was trying for.

And my conversion seemed to have worked well--all the test cases passed, including some extra ones. I also tried triggering the edge cases by hand, and those worked as well.

Getting your code to work after sincere, raw effort is one of those joys right up there with [good food](https://upload.wikimedia.org/wikipedia/commons/3/35/Biryani_Home.jpg).

### Some challenges with the changes that are cooking

While my complete conversion the bulk of `cmd_add()` seems to work, there's one part of it that bothers me a bit.

Here's the logic of the part that checks whether a particular path exists in the index or not:
```C
/*
 * I should be checking the return value,
 * but for the sake of this example, let's
 * leave that out.
 */
read_cache_preload(NULL);
if (ps.nr) {
	int i;
	char *ps_matched = xcalloc(ps.nr, 1);

	/* TODO: audit for interaction with sparse-index. */
	ensure_full_index(&the_index);

	/*
	 * Since there is only one pathspec, we just need
	 * need to check ps_matched[0] to know if a cache
	 * entry matched.
	 */
	for (i = 0; i < active_nr; i++) {
		ce_path_match(&the_index, active_cache[i], &ps,
			      ps_matched);

		if (ps_matched[0]) {
			if (!force)
				die(_("'%s' already exists in the index"),
				    path);
			else if (!S_ISGITLINK(active_cache[i]->ce_mode))
				die(_("'%s' already exists in the index "
				      "and is not a submodule"), path);
			break;
		}
	}
	free(ps_matched);
}
```

Before iterating through the cache entries of the index, you need to populate it.

There's two functions for this: `read_cache()` and `read_cache_preload()`. I have used the latter in my code. The thing is, when I swap it with the former, I could not find any change in the behaviour of my code. They appear to function equivalently.

I understand that the `*_preload()` variant takes a pathspec which preloads index contents that match the pathspec in parallel. I don't know what passing NULL to it does. Moreover, does this imply that `read_cache()` loads the cache on-demand, ie, it does no preloading?

I am not sure about what *exactly* are their differences, and when is one variant preferred over the other.

Consider this the classic case of "my code seems to work, but I don't know why". Bah.

## Reflections

I'm starting to realise how quickly your work can get stalled, and how, just as quickly, you can make a lot of progress. At least in the way I operate, progress is not a steadily increasing straight line, but a spiky erratic line that generally trends upwards.

So to those in a similar position, who will read this in the future--don't let the spikes around your immediate neighbourhood sway you mentally. Just work towards the larger upward trend, and stick to sound methods of analysing what made your code not work, and solve your problems patiently. That has served me well so far.

### Blooper of the week

Well I forgot to include this section last week, but it was kind of implicit with my ["it's obvious" debacle](http://atharvaraykar.me/gitnotes/week4#on-obviousness).

Well this week, I had a small blooper. To quote the [MyFirstContribution](https://git-scm.com/docs/MyFirstContribution) documentation:

> While you’re looking at the email, you should also note who is CC’d, as it’s common practice in the mailing list to keep all CCs on a thread.

After Rafael and Eric commented on my patch, I forgot to CC them when I rerolled. New contributors: please check your CC's before hitting send!

And my other blooper was to forget [signing off](https://git-scm.com/docs/SubmittingPatches#sign-off) one of my patches. Oops.
