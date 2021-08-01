# Week 11: A slump

# Project Progress

To be honest, this week has probably been my least productive week for my Git work. I don't have much to show concretely. I'm not too worried about this slump though, because I am confident that I will get the project done, and my intention is to see my conversion work make it into the mainline even if it goes past my GSoC period.

I have been responding to the reviews I got on my [add](https://lore.kernel.org/git/20210801063352.50813-1-raykar.ath@gmail.com/) and [update](https://lore.kernel.org/git/20210722134012.99457-1-raykar.ath@gmail.com/) series, and sent a new version for the former. I will reroll for the latter either today or tomorrow.

I have also decided a rough strategy for how to make the submodule command a builtin after my conversions are done. Studying the general pattern of how these have been previously done[1], I think this is what would be my best bet:

- Rename `git-submodule.sh` to `git-submodule-legacy.sh`.
- Create `builtin/submodule.c`, that will read from a config switch called 'submodule.useBuiltin'. If this is set to false, just call the legacy shell script, else use the builtin versions.
- Copy the functions from `builtin/submodule--helper.c` to `builtin/submodule.c` one by one. Make necessary changes in the flag parsing.
- Once all the functions have been successfully copied, make the default value of submodule.useBuiltin to true.
- ...eventually remove `submodule--helper.c` and the shell script entirely, and deprecate the 'submodule.useBuiltin' option.

[1] References:  
https://lore.kernel.org/git/20180806193111.12229-2-predatoramigo@gmail.com/  
https://lore.kernel.org/git/c659174b1be950072580ee176d59f85574cdb96c.1533753605.git.ungureanupaulsebastian@gmail.com/#r  

# Reflections

## Last week's survey: a summary of the responses

Last week, I asked the following question to members of the Git List about their development setup and workflows.

> What tools, systems and workflows do you find valuable in your day-to-day work? In particular I’d be happy getting insights like:
> * Any strategy or approach to work, kind of like the example I quoted above
> * Any scripts and tools that assist you
> * Opinionated handling of multiple in-flight series and methods to approaching reviews
> * Atharva, you are overthinking this! I just use a straightforward { editor + MUA + git } stack and go with the flow!

Here is my summary of the responses:

[Christian Couder](https://lore.kernel.org/git/CAP8UFD3j4SpgiZ9UZOhWp7CdChpxutEyyKRb1EnXeprQssML_g@mail.gmail.com/)  
My mentor uses Gmail for handling all the email, and Emacs as his primary editor. He uses the usual command line tools for development, like make, gdb, etc, with a bunch of shell aliases for building, debugging and sending patches.

(Pointers for Git development with Gmail can be found [here](https://git-scm.com/docs/git-format-patch#_gmail).)

[Kaartic Sivaraam](https://lore.kernel.org/git/b07fe877-f0ba-9a20-47b2-16c8efaa447c@gmail.com/)  
Kaartic Sivaraam uses a straightforward workflow, with Thunderbird for handling email and the usual command line tools for any development work.

[Felipe Contreras](https://lore.kernel.org/git/60ff06ad2b298_31bb20891@natae.notmuch/)  
Felipe Contreras uses a combination of mbsync to download emails, and notmuch to create a database from his maildir, that can be queried easily. His primary editor is Vim, and he uses [notmuch-vim](https://github.com/felipec/notmuch-vim) as the frontend. He uses a combination of tags and queries to organize his work.

[Philippe Blain](https://lore.kernel.org/git/ee679a57-0851-962d-a63a-6a0bdba35b2e@gmail.com/)  
Philippe Blain is a small-time contributor, and he prefers using the public-inbox archive directly to read mail; he isn't subscribed to the mailing list. He currently uses Thunderbird to review code, by others, after importing the relevant mail into a designated IMAP folder of his Gmail account.

He uses [b4](https://pypi.org/project/b4/) to import a thread, and `git imap-send`.

He has made a detailed [Gist](https://gist.github.com/phil-blain/d350e91959efa6e7afce60e74bf7e4a8#abusing-git-imap-send-to-upload-any-email) that does a good job of explaining this workflow.

When it comes to making contributions, Philippe uses [Gitgitgadget](https://gitgitgadget.github.io/), along with Git's branch descriptions to handle cover letters, and the GitHub CLI to make PR's that get converted to patches.


---

If any other Git developer is interested in adding your workflow for others to see, please do send an email to me, and I'll add it here.

---

What I learnt from this small sample size is that everyone's setup is different, and there seems no single way that is considered the "best". This is good news in one sense--a lot of people might assume a high barrier and a lot of configuring of email to get started, but even a vanilla Gmail + Git scripts is good enough, even for veterans. Or you could just use Gitgitgadget.

## Email pain: redux

Last week I [complained](https://atharvaraykar.me/gitnotes/week10#blooper-of-the-week-woe-is-email) about my not-so-ideal email situation. It got worse this week, as Thunderbird started giving [strange problems](https://lore.kernel.org/git/CADi-XoRNRrtC6bQ-DETj=0Bmy=WJzv3mk++QkrpDOZ6THGhaZQ@mail.gmail.com/) with sending email to the list. From what I can tell Thunderbird has been setting the wrong value of Content-Transfer-Encoding before I send email to the list, which gets rejected.

It was starting to become rather clear to me that I might have to configure my email to get it working anyway, which I do not enjoy. I thought, hey, might as well make it fun if I have to configure anyway. So I am trying out a weird setup, of using [mbsync + mu4e](https://github.com/hlissner/doom-emacs/blob/develop/modules/email/mu4e/README.org#mbsync). The weird part is that I have chosen to only sync my "All Mail" folder. I don't have an inbox folder at all. Instead I have setup some bookmarks that are just `mu` queries for things I need, like mails from the list, mails that involve me, personal messages, spam from my uni's placement department etc. I am essentially 'filtering in' mail rather than 'filtering out' mail from my inbox.

How do I know if a new mail arrives without an inbox? I have setup a query for seeing "all mail that arrived today and addressed to me", and "all mail that arrived this week and addressed to me".

So far it has served me well, and my emails don't bounce anymore.

I also have achieved the easiest way to reach inbox zero—remove the inbox altogether ;-)
