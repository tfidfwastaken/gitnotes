# Week 13: Preparing for the wrap up

# Project Progress

This week's update will be relatively short, partly because I will be out of town and juggling multiple things this week, and also because I am saving up for the larger final report ;-)

This week I re-rolled my [`run-update-procedure`](https://lore.kernel.org/git/20210813075653.56817-1-raykar.ath@gmail.com/) patch. I was tempted to also send out my series that [converts the rest of `submodule update`](https://github.com/tfidfwastaken/git/tree/submodule-update-list-1) to C, but it is quite a big series, and since it is release week for Git, I felt I'd be a little more patient, and wait for `run-update-procedure` to get picked up first rather than send a storm of all these related series at once.

In other news, my attempt to making `submodule` a fully C builtin is shaping up well. Here's a brief of how the transition will work:

- We have a switch called `submodule.useBuiltin` (as well as `GIT_TEST_SUBMODULE_USE_BUILTIN` as an environment variable for the test suite) which defaults to false. When set to true, we use the builtins that are purely in C.
- If the switch is set to true, but the user requests a command that has not been converted yet, it falls back to using the old shell starting point. This allows us to test the newly ported builtins with the switch set to true, while still passing the test suite as a whole.

I expect this transition to be rather brief, because the commands are already implemented, all we need to do is manage the flag parsing. As of now [I have already ported the `submodule status` command](https://github.com/tfidfwastaken/git/commits/submodule-make-builtin-2) to a pure C builtin. I am open to taking comments on this approach, feel free to leave them on the GitHub web interface, or on the list.
