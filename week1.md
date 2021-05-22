# Week 1: Getting acclimatised | My plan | Emacs setup | fixing macOS build errors

## Getting acclimatised

The Git developer community is a friendly, welcoming place. There is something that I find very refreshing about the lack of ceremony, and the unpretentiousness that follows every bit of work that the developers do. The discussions in the mailing list gave me the impression that the dominant mindset around here is that the craftsmanship of the code is above all -- all human ego and pride is set aside in order to ensure the best possible work.

Even with the friendliness and the guidance that I have received so far, I still feel like a visitor in a new country. There are many implicit customs in the Git development community that don't come to me naturally, especially given my inexperience in being a part of a FOSS project that operates from a mailing list.

It's always the small, seemingly silly things -- How much should I omit when quoting someone in a reply? Is it worth CCing this person for a question, or am I creating a distraction? If someone responds to me, would sending a mail just to acknowledge that response add noise to the mailing list? If it ever comes to it, how do I acknowledge other people's code if my patch adapts from their work? [1]

I did figure out some of these. The other 'customs' I'll know about soon enough, but I'll know I would feel a lot more at home the day I don't think about these things when working with Git. Till then it is a lot of trying, stumbling, and then becoming better at it. The craft is paramount.

## My Next Steps

I have spent this week trying to increase my presence in the mailing list (to mixed results), and more importantly, bring some structure to my work and plan so that I don't squander away my time and effort.

I am going to start my work on converting `submodule add` to C. The good news is that I already have Shourya's stalled patch as an excellent reference for how I can begin my work. Reading it this week has helped me understand the Git way of doing operations (for the lack of a better phrase), like knowing the functions that I can use for the shell conversions, the way strings are handled, etc.

While I will end up heavily adapting Shourya's work. One major distinction would be in how I divide it up. Shourya's patches for `submodule add` back in the day faced two challenges:

- It was sent out of the GSoC period, so there was limited reviewer bandwidth.
- The patch came in all at once -- the whole port from start to finish. That made it really hard for the other devs to review in one sitting. This also meant that a lot of the perfectly good portions of the code never made it in because of one buggy part.

I will be sending my patches in much smaller chunks, porting only parts of the shell code by wrapping it into a shell `submodule--helper` subcommand. More obvious and easy parts to wrap into subcommands is the functionality to [resolve the submodule and repo paths](https://github.com/git/git/blob/107691cb07aab771585844fcd39d5e1c7f1ed14b/git-submodule.sh#L153-L191), [normalize the paths](https://github.com/git/git/blob/107691cb07aab771585844fcd39d5e1c7f1ed14b/git-submodule.sh#L193-L204) and [handle custom submodule names](https://github.com/git/git/blob/107691cb07aab771585844fcd39d5e1c7f1ed14b/git-submodule.sh#L232-L242). After that, I will work on getting the [config handling](https://github.com/git/git/blob/107691cb07aab771585844fcd39d5e1c7f1ed14b/git-submodule.sh#L281-L307) converted to a subcommand, and this would be my first piece of functionality that I expect will pose a challenge to me, as I have not yet fully understood config handling generally works, and there's a case of a valueless value that, from what I can tell, was not handled in Shourya's patch [2].

A small downside of this approach is that I will have to create add a bunch of subcommands to the commands array of `submodule--helper` only to have them removed in future patches as I eventually convert everything to C. This seems fine to me, because it allows more smaller, easily reviewable patches.

## A basic Emacs setup for Git developers

I wish I had done this earlier. In my very first patch, I made a bunch of whitespace/indentation errors (Git prefers tabs of width 8), because I did not sort this out correctly. Those of you using Emacs, and wanting to send patches to Git, put this in your `.dir-locals.el` at the root of your Git project repo:

```elisp
((c-mode . ((c-file-style . "linux-tabs-only")
            (indent-tabs-mode . t)
            (show-trailing-whitespace . t)
            (c-basic-offset . 8)
            (tab-width . 8))))
```

## macOS error: libintl.h not found

When I first tried contributing to Git after recently switching from a Linux machine to a Mac, I got an error that looked like this, when I tried to build Git:
```
./gettext.h:17:11: fatal error: 'libintl.h' file not found
#       include <libintl.h>
            ^
```

Turns out this is a header used by the gettext library that is used for internationalization. Unfortunately, from what I gathered, libintl is a part of glibc, while Macs (and BSDs, where I believe the same error would happen) give us libc as the standard library for C. The initial workaround was to just run `make configure && ./configure` which would disable internationalization and allow Git to compile. But I did not feel fully satisfied by this. I think it is important that people on Macs should be also be able to work with translating and internationalizing.

After some fiddling, this is solution worked for me, which is fairly non-destructive and lets me compile with libintl:

(Assuming you use homebrew) Install gettext:
```
brew install gettext
```

Find the location of the libintl headers and libs:

```
find /opt -name "libintl.*" -print
/opt/homebrew/include/libintl.h
/opt/homebrew/lib/libintl.dylib
/opt/homebrew/lib/libintl.8.dylib
/opt/homebrew/lib/libintl.a
/opt/homebrew/Cellar/gettext/0.21/include/libintl.h
/opt/homebrew/Cellar/gettext/0.21/lib/libintl.dylib
/opt/homebrew/Cellar/gettext/0.21/lib/libintl.8.dylib
/opt/homebrew/Cellar/gettext/0.21/lib/libintl.a
```

Note the location of the gettext directories.

Now run the following (it might slightly differ if your CFLAGS and LDFLAGS are already set. Just ensure the gettext paths are included with whatever else is already there):

```
make configure
./configure "LDFLAGS=$LDFLAGS -L/opt/homebrew/Cellar/gettext/0.21/lib" \
     "CFLAGS=-I/opt/homebrew/Cellar/gettext/0.21/include"
```

After it has configured properly you should be able to build it with `make`. I hope this helps someone who needs it.

[^1] My Gen-Z is showing really strongly here.  
[^2] https://lore.kernel.org/git/xmqqlfdy7niy.fsf@gitster.c.googlers.com/
