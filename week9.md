# Week 9: A solution, just in time

# Project Progress

I left off [last week](week8#path-pains) with a problem of getting paths right in my `submodule update` conversion work.

I now have a solution that does get the tests to pass, and this is my attempt at documenting how I got there.

So to summarize,

**The Problem**: I cannot get 'submodule update --init --recursive' to work properly.

**Cause**: The paths get malformed in my first implementation[1] because I am not able to transfer the git 'superprefix' to the `init_submodule_cb()` callback with the current interface. The superprefix is required by the `get_submodule_displaypath()` function to create the correct display path.

Thus, t406.5 fails.

So here are the various solutions I tried.

### Launching the `init` part as a subprocess

This seemed like the most obvious thing to do, because a lot of the other commands that recursed into submodules were implemented this way. This also lets the default `git` handler take care of my superprefix. The only downside is the cost associated with spawning a new subprocess.

*Outcome*: It broke a *lot* of tests. What seemed like the most straightforward solution created very strange behaviour that I have still not been able to properly understand.

The strange behaviour is best demonstrated by the test case t7400.56.

Here's the relevant output:
```
expecting success of 7400.56 'update --init':
	mv init init2 &&
	git config -f .gitmodules submodule.example.url "$(pwd)/init2" &&
	git config --remove-section submodule.example &&
	test_must_fail git config submodule.example.url &&

	git submodule update init 2> update.out &&
	test_i18ngrep "not initialized" update.out &&
	test_must_fail git rev-parse --resolve-git-dir init/.git &&

	git submodule update --init init &&
	git rev-parse --resolve-git-dir init/.git

Submodule path 'init' not initialized
fatal: not a gitdir 'init/.git'
Submodule 'example' (/Users/atharva/git/t/trash directory.t7400-submodule-basic/init2) registered for path 'init'
Submodule path 'init' not initialized
Maybe you want to use 'update --init'?
fatal: not a gitdir 'init/.git'
not ok 56 - update --init
```

Even though the init command did actually run (as shown by the message
"Submodule 'example' registered for path 'init'") within the same run,
when update wants to actually perform the clone, it reads the submodule
as not active, and triggers the "Maybe you want to..." message.

I managed to locate the [source](https://github.com/tfidfwastaken/git/blob/7ad995f465072b60a8c33e51c4f91d08ee3d2484/builtin/submodule--helper.c#L2179-L2183) of this problem.

```C
/* Check if the submodule has been initialized. */
if (!is_submodule_active(the_repository, ce->name)) {
	next_submodule_warn_missing(suc, out, displaypath);
	goto cleanup;
}
```

This if branch should not have got triggered, because the previously initialized submodule is obviously active. I have verified this by manually running the commands and checking the git config file, and it is indeed marked as `active = true`.

And here is the other weird part. If I run the same `update` command again ie, modify the test case to this:

```sh
	mv init init2 &&
	git config -f .gitmodules submodule.example.url "$(pwd)/init2" &&
	git config --remove-section submodule.example &&
	test_must_fail git config submodule.example.url &&

	git submodule update init 2> update.out &&
	test_i18ngrep "not initialized" update.out &&
	test_must_fail git rev-parse --resolve-git-dir init/.git &&

	git submodule update --init init &&
+	git submodule update --init init &&       # let's run this again
	git rev-parse --resolve-git-dir init/.git
```

...it passes!

Git is able to tell that the submodule has been initialized only *after* the complete run of the command, but not *during* it.

So this makes me believe that even though the registration of the
submodule has happened, the Git internal machinery is not able to pick
up the new state in the same run. I have been unable to determine why
this is the case.

### Passing the superprefix explicitly

With my "obvious" solution failing, I instead resorted to a different solution, that can be seen [here](https://github.com/tfidfwastaken/git/commit/0ee268be35d19b147fffa870cdbc8e807969d4f7).

To summarize, it involves teaching `init_submodule()` to take an explicit `superprefix` argument that is received from the `struct init_cb` callback data. The nice thing about this method is it allows me to avoid spawning a subprocess. I also refactored `get_submodule_displaypath()` by splitting off a `do_get_submodule_displaypath()` that lets me have greater control of the superprefix parameter, rather than relying on whatever `get_super_prefix()` returns.

This makes all the tests pass, which is a relief.

Even though I intend to settle on this solution, I would be happy to know from any mailing list readers about why my other attempt might have failed.

## What next?

I was busy trying to get all my project troubles sorted, so my code is a little bit messy, both stylistically and in terms of polish. Now that I do have code that works correctly, it's time for me to make it a proper patch series, and have it look like I nailed it effortlessly in my first try ;-)

I am also waiting on my [list patch](https://lore.kernel.org/git/20210710074801.19917-1-raykar.ath@gmail.com/) to land on master, so that I can then send the next patch for the `submodule add` conversion.

This week has turned out fairly good and I have dealt with the hardest coding parts of my project, which is a good thing, because past this week I will be busy preparing for my exams.

# Reflections

## Blooper of the week

I'll start with a blooper of mine, as it ties well into the point I am trying to make here.

Since this week was a lot of debugging work while trying to meet a deadline I set for myself, it was easy for me to miss the fact that my mentors do not have the context that I do. They do not (and should not) keep the intricacies and layers of my code's problem replaying in their heads for many hours at a time.

When I was stuck with the problem I described in the first section, I did a somewhat subpar job at giving the context needed to help. I gave no links to relevant code, and in fact even made an error and linked to the *wrong* commit, so all that decontextualised technical details I gave did not even match the actual behaviour.

Kaartic Sivaraam gave it a try, but I did not favours in making it easy to help me. He politely told me,
> I'll try to answer these after I get to know the situation better.

Reading my original email again, present me might've just shot back a reply at myself with*,
> Nope. Try again.

Which is what I did. I made a much better attempt at explaining my problem, and sure enough, it got much more helpful responses.


`*` I do not endorse talking to anyone like this. It always helps to be more empathetic.

## On the art of asking questions

This brings me to my main point. In a project like this, there will be a lot of occasions where you'd need to ask questions. It helps to get good at it. I am far from perfect at this (as demonstrated). Feedback from my mentors, and the better attempts of getting answers have taught me this:

- Don't hesitate to ask questions when you are stuck.
- Ask questions only when you have struggled with the problem and gave it a good attempt.
- Give context to your problems. Please no link and code snippet poverty.
- Explain what you have tried so far, and make that detailed and specific.
- Read what you are about to send, preferably after taking a quick break away from your computer. Often it's easy to miss that the reader does not have the context you have.
- Interact more with the community by asking questions in your patch cover letters and replies to reviews.

It's easy to think that asking questions might be annoying, but if you have taken efforts on your side, and made it easy for the reader to understand your problem, it's not a problem at all. In fact, it can help you get along with the community members better.

I believe I have done fairly well in the first two points that I have stated. The rest definitely needs more work.
