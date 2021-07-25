# Week 10: Patch juggling and Email woes

# Project Progress

Some good news—the exams that I mentioned [last week](week10) will not happen. This means I can give more time to Git than I previously expected.

I finally started sending all my work to get ~~roasted~~ reviewed on [the](https://lore.kernel.org/git/20210722112143.97944-1-raykar.ath@gmail.com/) [list](https://lore.kernel.org/git/20210722134012.99457-1-raykar.ath@gmail.com/). The feedback from Ævar and Junio has actually been pretty great, because it has got me revisiting some of the choices I made, and also fixing some mistakes that look obvious in hindsight.

This is the first week in a while where I didn't feel blocked in any significant way at all. It's just a matter of fixing and rerolling my patches. The power of hindsight compels me to note that I should have been sending out my patches a lot earlier, and if I had to count one of the overarching mistakes during my time at Git, it would be this.

I had thought to wait until my [first](https://lore.kernel.org/git/20210615145745.33382-1-raykar.ath@gmail.com/) series makes it to master so that I can rebase my other series which depend on it to master and send it again. But the [SubmittingPatches](https://git-scm.com/docs/SubmittingPatches) doc was clear about the fact that it's okay to just keep sending the patches and mentioning the dependency, and probably more preferable than to base it off next, where my first topic now sits. Oops.

## Next steps

I will be working on refining my patches. With the free time that I get while I wait for responses, I will start having a look at strategies to [get rid of](https://lore.kernel.org/git/nycvar.QRO.7.76.6.2011191327320.56@tvgsbejvaqbjf.bet/) `git-submodule.sh` entirely.

# Reflections

## Blooper of the week: woe is email

Saying "woe is email" is not the best look for a new mailing list developer such as myself. There are many things I love about it, and many things I hate, with not much neutral in between. This dichotomy makes it all the more woeful. The part I love can be summed up succinctly as: everyone has it, everyone uses it, no one 'owns' it.

Now the part that bothers me: no tool does it right for my workflow. Recently there was a [bug reported by a user](https://lore.kernel.org/git/CAFSh4Uyr5v9Ao-j0j7yO_HkUZSovBmSg7ADia7XCNZfsspFUYg@mail.gmail.com/) in the list. I saw no replies to it, and I had some free time, so I decided to have a look into it. I did, and [responded](https://lore.kernel.org/git/3D8703D8-54E6-4CF0-9E9F-CCAFFAA8914C@gmail.com/) to it, along with pointers to what originated the bug.

Right after I sent that message, I scrolled down a bit in my mail client, and then I saw a mail from Ævar that was a [reply to that post](https://lore.kernel.org/git/patch-1.1-fc26c46d39-20210722T140648Z-avarab@gmail.com/), which was not only more helpful, for it was an actual patch, but it was sent almost a whole day before my message!

The reason I did not see that reply at first was because of Apple Mail, which I used out of laziness because it worked out of the box. Apple's mail client threads messages based on the subject line, and not by the In-Reply-To header. So when Ævar replied with a patch, he changed the subject line to reflect the fact that it was a patch, and my mail client did not add it to the thread.

I thought of solving this whole family of problems, by switching to a more reasonable mail client. Any kind of webmail was out, because I needed something that handles plaintext properly, [and does not mess with whitespaces](https://git-scm.com/docs/SubmittingPatches#send-patches) through features like an unconfigurable "format=flowed".

The other natural choices were: mu4e, since Emacs is my primary editor, and Thunderbird. mu4e was out for me, because I had used it once long ago, and it was just too much configuring. I can live with that, but the main issue is it's very annoying when I want to have some filtering features which again requires learning the format of the config file and endless fiddling (filtering is essential to me given the amount of junk mail sent to me by my college, occasionally peppered with actually useful information). From what I can tell this is the situation with any terminal-based MUA, which either needs `mbsync` or `offlineimap`.

Thunderbird is more like what I am looking for, which has a convenient GUI with loaded batteries, while still being more configurable than Apple Mail and actually handling threading properly. This is what I currently use, but my laptop is unhappy with it, as it eats up around 40-50% of the CPU usage in the background. It also eats up more battery, almost as much as my browser. This is apparently a [bug](https://bugzilla.mozilla.org/show_bug.cgi?id=1305207) that's been unresolved for many years now. Git developers reading this: if you have personal recommendations for something that is easy to use but also lightweight, please do let me know! (and it needs to work on macOS)

## The mailing list developer workflow?

The point above also got me thinking (10 weeks too late). What does the workflow of a developer who primary works via mailing lists look like? Not just in terms of tools, but in terms of systems?

Early on, I quite liked what [nasamuffin had to say about the patch flow](https://nasamuffin.github.io/git/open-source/email/code-review/2019/05/22/how-i-learned-to-love-email-patches.html), which helped me when I was first getting used to it. Especially this bit:

> Your reviewers will likely provide their comments inline, and it’s often good manners to reply to each review comment indicating whether you’ve taken an action or not. When I’m replying to review comments, I like to have a window open with a reply to the email, and another window open with my code. I make sure that I’ve begun an interactive rebase and paused to edit the commit I’m examining comments for, and then I open my source file. As I read comments in the email, I make a change as appropriate in my source, then type a response inline to the comment in question. (That way I know I didn’t miss anything!)

I haven't followed this suggestion too religiously, but it has worked generally well for me. But other than that, my method of working has been largely ad-hoc and unsystematic.

The tools, systems, workflows and practices of individual mailing list developers have not been well-documented anywhere at least from my brief searching. I think it would be valuable for future contributors, especially from backgrounds like mine, where I had never touched email contributions before, but now have to send patches that way quite regularly.

So I'd like to ask Git Developers reading this:  
What tools, systems and workflows do you find valuable in your day-to-day work? In particular I'd be happy getting insights like:

- Any strategy or approach to work, kind of like the example I quoted above
- Any scripts and tools that assist you
- Opinionated handling of multiple in-flight series and methods to approaching reviews
- Atharva, you are overthinking this! I just use a straightforward { editor + MUA + `git` } stack and go with the flow!
