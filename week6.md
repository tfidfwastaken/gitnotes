# Week 6: On to update

# Project Progress

This week I made some refinements to my [submodule add patch](https://github.com/tfidfwastaken/git/commits/submodule-helper-add-3a) after taking feedback from my mentors Christian Couder and Shourya Shukla, and also Kaartic Sivaraam (who at this point I consider a de facto mentor—thanks for all the help!).

It's been a busy week for the Git list, and with the maintainer being offline this week, my [last patch](https://lore.kernel.org/git/20210615145745.33382-1-raykar.ath@gmail.com/) on the list has not not yet got too much attention. In these times, it's best to be patient. I have been advised by my mentors to wait until the next "What's cooking..." mail[^1], and take action accordingly.

So while that is blocked at the moment, I decided to start work on the other major part of my project, that is to convert `submodule update` to C, much in the same way. This functionality is a lot more interesting and complex compared to merely adding a submodule, and it can often be a source of [strange behaviour](https://lore.kernel.org/git/CAKjYmsELpf9r3bAJj_JUHgVegw_7z2KzyuR_6FYYngpC1XmNeg@mail.gmail.com/) and edge cases.

What makes my life easier though, is that a lot of the hard parts [have already been converted](https://lore.kernel.org/git/?q=submodule+update+dfn%3Asubmodule--helper.c+f%3A%22Stefan+Beller%22+&r=) to helper subcommands by Stefan Beller in the past.

## Current strategy

I'll be introducing a 'run-update-module' helper to replace [this bit of code](https://github.com/tfidfwastaken/git/blob/master/git-submodule.sh#L586-L649).

The main challenge in that portion is converting the `is_tip_reachable` and `fetch_in_submodule` shell functions. For the latter, it is quite clear the only way to make it work is to shell out a new child process and perform the fetch. For the former, it seemed like this should be something that should be easily doable with the internal APIs but unfortunately it isn't.

This is what I have to convert:
```shell
is_tip_reachable () (
	sanitize_submodule_env &&
	cd "$1" &&
	rev=$(git rev-list -n 1 "$2" --not --all 2>/dev/null) &&
	test -z "$rev"
)
```

The fancy rev-list call is basically saying, "print the SHA1, if it is not reachable from all of the refs". This kind of an invocation was [done before](https://github.com/git/git/blob/49f38e2de47a401fc2b0f4cce38e9f07fb63df48/submodule.c#L926-L971) in the context of submodules, but it does not look reusable to me because it seems to have side effects, that meddle with the repository's object store. As it is, this will have to probably be run as a child process as well.

The other challenge would be to find a sensible way to deal with the error handling during the conversion. Currently it is building up a string of error messages, separated by ';' and then splitting it again in the end. I have to ensure that the error string building is preserved properly at the interface boundaries of my helper.

## Miscellany

On the whole, this week felt slower than I'd have liked, but the project is still going ahead and I still find it fun to work on. That is what is important to me ultimately. The more I understand the working of what's happening underneath, the more areas of tiny improvements I can find in the Git project's code. I hope that even after this project I can get to those areas as well, and better yet, use those leftover bits to help other people make their first steps to contribute to Git.

# Reflections

## How does Git even start‽

Occasionally, I like casually touring the Git codebase. This time, I wanted to find out the entry point of all Git commands, and I was left a puzzled man.

The entry point of Git is not in `git.c` as you may expect (although it was at one point), but in `common-main.c`. For reference, this is Git's (truncated) `main()` function:
```c
int main(int argc, const char **argv)
{
	...
	git_resolve_executable_dir(argv[0]);

	git_setup_gettext();

	initialize_the_repository();

	attr_start();

	...

	result = cmd_main(argc, argv);

	...

	return result;
}
```

The key part here is `cmd_main()` is called, which takes you to the right builtin. Not all builtins are available in `git.c` however. Some of them like `git daemon` are in their own file.

Searching the code for `cmd_main()` gives me multiple results:
```
shell.c
128:int cmd_main(int argc, const char **argv)

remote-curl.c
1474:int cmd_main(int argc, const char **argv)

http-fetch.c
80:int cmd_main(int argc, const char **argv)

imap-send.c
1543:int cmd_main(int argc, const char **argv)

http-backend.c
739:int cmd_main(int argc, const char **argv)

http-push.c
1702:int cmd_main(int argc, const char **argv)

git.c
854:int cmd_main(int argc, const char **argv)

daemon.c
1270:int cmd_main(int argc, const char **argv)
```

So my question is: *how* does the `main()` function know which `cmd_main()` to call? From whatever I could understand from the makefile, all these files are linked together.

## Blooper of the week

This week's blooper is as usual with mail, but unlike the other ones, this one happened over several weeks. One of my mentors Shourya, has two email addresses associated with him, but one of them has fallen into disuse for Git work. All my patches of the last series from v1 to v6 were addressed to this wrong email address. I was made aware of the correct one, but I only updated the cover letter in my new patches, while the actual contents were just picking up the old address, so it unfortunately did not reach Shourya's inbox.

The only way to avoid this is to check the addresses before sending the patch out. It does feel a little manual and error-prone, and I think I'll just need to be more careful next time.

[^1] This Git tradition has a practical use, though I can't help but find this practice (especially with the choice of words) extremely charming.
