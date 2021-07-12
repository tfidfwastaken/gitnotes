# Week 8: A rocky path

# Project progress

I wanted to finish my first draft of the `submodule update` conversion by this week but that did not happen. I want to speed things up because of my exams being on the horizon. Although I have the logic fully [written down](https://github.com/tfidfwastaken/git/commits/submodule-helper-update-1), it's currently a heritage site thanks to the large biodiversity of bugs.

It has been quite a challenge to structure these patches, and I really hope it won't be too hard to review.

While I continue relentlessly bug-hunting, I'll describe some of the difficulties I am facing at the moment.

## Path pains

To handle the `--recursive --init` flag of `submodule--helper update`, we iterate through each submodule. For each submodule, we create a child process that calls `submodule--helper update` again, with mostly the same options, but we tweak the prefix path called `--recursive-prefix` in my code. This path is supposed to serve as the `--super-prefix` of the `init` command that is launched by the recursive child process.

Currently my code fails t7406.5 like so:

```
## THE TEST
expecting success of 7406.5 'submodule update --init --recursive from subdirectory':
	git -C recursivesuper/super reset --hard HEAD^ &&
	(cd recursivesuper &&
	 mkdir tmp &&
	 cd tmp &&
	 git submodule update --init --recursive ../super >../../actual 2>../../actual2
	) &&
	test_cmp expect actual &&
	sort actual2 >actual2.sorted &&
	test_cmp expect2 actual2.sorted

## EXPECTED
Submodule 'merging' (/Users/atharva/git/t/trash directory.t7406-submodule-update/merging) registered for path '../super/merging'
Submodule 'none' (/Users/atharva/git/t/trash directory.t7406-submodule-update/none) registered for path '../super/none'
Submodule 'rebasing' (/Users/atharva/git/t/trash directory.t7406-submodule-update/rebasing) registered for path '../super/rebasing'
Submodule 'submodule' (/Users/atharva/git/t/trash directory.t7406-submodule-update/submodule) registered for path '../super/submodule'

## ACTUAL
Submodule 'merging' (/Users/atharva/git/t/trash directory.t7406-submodule-update/merging) registered for path 'merging'
Submodule 'none' (/Users/atharva/git/t/trash directory.t7406-submodule-update/none) registered for path 'none'
Submodule 'rebasing' (/Users/atharva/git/t/trash directory.t7406-submodule-update/rebasing) registered for path 'rebasing'
Submodule 'submodule' (/Users/atharva/git/t/trash directory.t7406-submodule-update/submodule) registered for path 'submodule'
```

Notice the paths at the right end—they should be relative to the `tmp/` folder, but they are not. This is because the function in `init_submodule()` that creates this path needs the `--super-prefix` variable. This variable is unset because I am calling the `init` functionality from within C, so the superprefix remains unset. There is no `git --super-prefix=foo cmd` shell invocation setting this for me.

Here is an attempt to remedy this:

```c
/* ... In module_update() ... */
/* parse flags for submodule--helper update */
if (update_data.init) {
    struct module_list list = MODULE_LIST_INIT;
    struct init_cb info = INIT_CB_INIT;

    if (module_list_compute(argc, argv, update_data.prefix, &pathspec, &list) < 0)
    return 1;

    /*
     * If there are no path args and submodule.active is set then,
     * by default, only initialize 'active' modules.
     */
    if (!argc && git_config_get_value_multi("submodule.active"))
        module_list_active(&list);

    info.prefix = update_data.prefix;
    if (update_data.quiet)
        info.flags |= OPT_QUIET;

    if (update_data.recursive_prefix)
        setenv(GIT_SUPER_PREFIX_ENVIRONMENT, update_data.recursive_prefix, 1);

    for_each_listed_submodule(&list, init_submodule_cb, &info);

    if (update_data.recursive_prefix)
        unsetenv(GIT_SUPER_PREFIX_ENVIRONMENT);
}
/* init is done, now do the updating of submodules... */
/*
 * if --recurse is specified, run a child process that calls
 * submodule--helper update again, but from within the submodule path
 */
```

Right before the `init` invoked by the `for_each_listed_submodule()` function, I tacked on code that sets the `GIT_SUPER_PREFIX_ENVIRONMENT` environment variable to the recursive prefix, which should lead us to the right output.

(I also understand those if statements look kind of awkward, I can fix the hackiness once I get it to work)

But unfortunately this did not work. Some print debugging showed me that the value is set right upto this point in `init_submodule`:

```c
static void init_submodule(const char *path, const char *prefix,
			   unsigned int flags)
{
	const struct submodule *sub;
	struct strbuf sb = STRBUF_INIT;
	char *upd = NULL, *url = NULL, *displaypath;

	// output: ../super
	printf("%s\n", getenv(GIT_SUPER_PREFIX_ENVIRONMENT));
	// output: (null)
	printf("%s\n", get_super_prefix());

	/* this function retrieves superprefix with get_super_prefix() */
	displaypath = get_submodule_displaypath(path, prefix);

	sub = submodule_from_path(the_repository, null_oid(), path);

```

Also a note: the `displaypath` variable is the one that is ultimately printed wrong, and the immediate cause of my failing test. If you are curious about what `get_submodule_displaypath()` is doing, you can have a look [here](https://github.com/git/git/blob/211eca0895794362184da2be2a2d812d070719d3/builtin/submodule--helper.c#L253-L271).

When queried by the [`get_super_prefix()`](https://github.com/gitgitgadget/git/blob/d486ca60a51c9cb1fe068803c3f540724e95e83a/environment.c#L237-L245) function, the answer is `(null)`. ~~This boggles my mind to no end~~ (see update). The implementation is basically the same `getenv()` call?

---

UPDATE: I have found out the immediate cause of why the environment variable is not being read properly. On another look at this `get_super_prefix()`:
```c
const char *get_super_prefix(void)
{
	static int initialized;
	if (!initialized) {
		super_prefix = xstrdup_or_null(getenv(GIT_SUPER_PREFIX_ENVIRONMENT));
		initialized = 1;
	}
	return super_prefix;
}
```

The static variable `initialized` is already set by the time `get_submodule_displaypath()` is called, and thus the environment variable is not re-read, and the old value of NULL is returned.

I am not sure how to tell Git that the environment variable has in fact been modified, and that it needs to be reinitialized. Maybe I am going about this whole thing wrong?

---

It is quite possible I am approaching this problem wrong, and I have narrowed my thinking because of me being in more of a rush than usual.

Fundamentally I want to ensure that the paths are handled correctly across recursive calls. My understanding is for that to happen, I need to pass `--super-prefix` to the next command somehow. One way to do it is to fork out yet another git child process with the `--super-prefix` set, but that seems like a bad solution.

Another way is to change the `init_submodule()` code to be explicit about handling prefixes and add that information to the `init_cb` callback parameter. My gut does not like this solution either, because it feels like I would be touching something that works perfectly fine right now, and it feels like a bit of a band-aid fix.

## Other challenges

There's a lot of other bugs, that I haven't looked at yet. I wanted to get this one out of the way first. But in general, the patch is starting to look large, the kind where it might take several seatings to get a review done. It does not help that `submodule update` is doing a lot more, especially with the recursive calls along with handling `init` when specified.

Another thing worth working on, once this is out of the way is the structure of the data passed around in all the update-related functions. Currently I have a [giant struct](https://github.com/tfidfwastaken/git/blob/09dcb05c4035dba46aaaa62cd1f03fc271067cde/builtin/submodule--helper.c#L2044-L2077) sitting around carrying all the flags passed to the update command, as well as a lot of parameters that are needed to be handed over to `struct submodule_add_clone` for the parallel cloning operations.

I wonder if that struct knows too much, and if there's a way to collapse or divide those two structs in a better way. I have deferred that for now, because these things are a lot easier to think about once I get the program to work correctly first.

## What has been done this week

Other than working on update, I re-sent my [first batch](https://lore.kernel.org/git/20210710074801.19917-1-raykar.ath@gmail.com/) of `submodule add` conversions to the list, which should be fairly easy to get merged.

And despite the problems with update, I have at least all the code converted. It is a matter of ironing out the bugs. I had given my mentors the date of 19th July for the completion of a first draft of submodule update, and I remain optimistic about being done by then. I will be having my final exams in the weeks after, so I cannot be as active as I normally am (hence the rush right now), although I still plan to respond to reviews and adjust my patches during that period.

# Reflections

Most weeks breeze along fairly well, without too many issues, while some weeks are hard. This was one of those hard weeks, the kind that had me sitting and debugging code at ungodly hours because of my compulsion to make things work. I do not encourage doing this. All it has given me is an illusion of productivity, because the gains are marginal in that state of mind and time. Alas, I am only human, and the monkey brain snatches the controls of my rational brain every once in a while.

A key feature of days that went smoothly for me are the ones where I spend less time working, but that time spent would be during my peak of focus and productivity, thus having a lot "bang for the buck".

Another important objective for me is to not burn out during all of this (a common phenomenon for a college student here, especially as of the last two years). I believe I am managing that fairly well, although the university's guerrila scheduling that is pervasive in these times threaten that. For the sake of my continued enthusiastic involvement in the Git community, I will ensure that those threats are not realised ;-)
