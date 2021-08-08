# Week 12: Nearing the finish line

# Project Progress

This week was more eventful than the last week. I sent a [patch](https://lore.kernel.org/git/20210807071613.99610-1-raykar.ath@gmail.com/) to finish the conversion of whatever is left of submodule add. The response has been favourable, and my first batch of conversions have already made their way to master! While this is all fine, it would be nice if someone could give some insight into [my question](https://lore.kernel.org/git/20210805071917.29500-1-raykar.ath@gmail.com/) on that series about the cache API. My use of the API in the series seems to give correct results, but I don't understand what it's actually doing, which is a not a good sign.

The [update conversion series](https://lore.kernel.org/git/20210802130627.36170-1-raykar.ath@gmail.com/) needs more eyes on it. Shourya Shukla has left some helpful comments, but other than that I don't quite feel confident about the way I structured the [`run_update_command`](https://github.com/tfidfwastaken/git/blob/6ce8fd2dc912f9073d6760b0aa77e0a5543ee26e/builtin/submodule--helper.c#L2340-L2436) function. I wonder if I should hold on to the follow-up of the update series until this one gets more comments, or if I should just send the next series in the hopes of renewing interest in what's already on the list. For now I am leaning on waiting a bit more.

I have also started some work on [making `submodule` a builtin](https://github.com/tfidfwastaken/git/tree/submodule-make-builtin-2). That series is still incomplete as of now, but should be in a reviewable state in a couple of days.

With all of that it does feel like I am inching closer to the finish line with my project. So is this stint with the Summer of Code. I don't think my work will be merged before the Summer of Code ends, because it still needs a lot of reviewing, but that's okay for me, since I plan to stick around for a bit even after the program ends. That said, I'd appreciate more people volunteering for reviews so we can wrap up the submodule conversion work faster :-)

It would be nice to see this work, which has spanned for [at least 6 years](https://github.com/git/git/commit/74703a1e4dfc5affcb8944e78b53f0817b492246) finally come to an end!

## Other Updates

I will be busy from 15-19th August because of some family-related business, which requires me to travel. It should not affect my work much, but I may be slower to respond in that period.
