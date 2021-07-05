# Week 7: The effects of history

## Project Progress

Plenty of fixups this week. My mentors and I have been going back and forth over my [submodule add](https://github.com/tfidfwastaken/git/commits/submodule-helper-add-4) series, which is starting to look much better to me now.

I have also finished *some* work on converting `submodule update` to C. That work can be found [here](https://github.com/tfidfwastaken/git/commits/submodule-run-update-proc-1). I will be looking at how object walking works, so I can save an extra fork over [here](https://github.com/tfidfwastaken/git/blob/95b666bf6d2f690f52a4b891bf5d69cc9e3d55d6/builtin/submodule--helper.c#L2405-L2422). If it turns out to not be easy enough for me, I'll probably come back to it after a full conversion is done.

My next steps are to work on my conversion of `submodule update` while simultaneously refining the `submodule add` patch as I wait for more feedback on it, especially in the [list](https://lore.kernel.org/git/20210615145745.33382-1-raykar.ath@gmail.com/).

This week gave me a real experience of how software that is worked on by many people can collect some imperfections. There were many inconsistencies that come along with years of conversions in `submodule--helper.c`—lots of error messages don't follow the conventions normally adopted by Git, the `die()` routine works differently in shell, and a lot of repeated code is found. The last point is very real—many people repeat code that is already found elsewhere, because in a project so big, it is easy to miss. I [almost did that](https://github.com/tfidfwastaken/git/commit/d63665605d38831360e6262caa01221594738719#r52995413) this week as well.

The good thing about the Git project is that it presents many opportunities to find and repair these kind of mistakes, thanks to the detailed commit messages. Every explanation is a `git blame` or `git log --grep` away.

# Reflections

## `git test`

Kaartic Sivaraam showed me an effective tool by Michael Haggerty called [`git-test`](https://github.com/mhagger/git-test). I used to only test the tip of my Git branch. Even though that passed, a few fixups on my old commits led to the intermediate commits being broken. This tool automates the process of running tests on every individual commit in your branch so you don't have to worry about that anymore. It helped me repair a whole bunch of mistakes.
