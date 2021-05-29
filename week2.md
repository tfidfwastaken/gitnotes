# Week 2: In praise of leaving things behind

Everything I do is built on top of the corpus of knowledge left behind by others. Some people leave things behind deliberately, while others do so unintentionally.

There's always much to learn from both.

My work with Git this week was made possible by a combination of both kinds of knowledge. I had to use several functions from Git's `strbuf` API, `strvec` API and `run-command` API, all documented as comments in their respective header files in the top level. These comments were the work of many contributors in the past. A deliberate attempt, knowing that some new developer many years down the line will be able to get their first start to the Git project, and maybe even their career.

### Putting cherries on top of someone else's cake

The other bulk of my work this week was possible only due to "historical accidents", that is breadcrumbs left behind by people in the past. The patch that I ended up sending this week is heavily based on the work of two generations of GSoC programmers who struggled over this problem. Thanks to this, my work felt like putting a cherry on top of someone else's cake.

I studied Shourya's stalled patch, parts of which had already been reviewed. Many of the developers who reviewed his code would not have known that someone in the future would learn from their comments and benefit from it, in ways that went beyond the patch at hand. For those reviewers it was just another day at work.

These breadcrumbs of the less deliberate kind are in no way inferior – people cannot always foresee what will be helpful to others in the future.

My advice to those who might be stuck when trying to contribute to Git for the first time – look for the breadcrumbs left behind by others, including the ones that are not deliberate. Read the mailing lists, and the practices followed by those in it. It will feel unnatural at first, like you are plunged into a conversation you have no context on. Over time, it makes more and more sense, and you can start making out the shape of how the community works.

### I want to leave things behind

I want to leave things behind for people who I don't know, in a time far into the future. Sure, there is value in the code I write, that benefits the users directly. But I can, and should always aim to do more than that – I want to help others grow through the things I leave behind. I want to make the community stronger. I hope these reports and my publicly available patches and messages help people in ways that I couldn't have predicted. And of course, there are the more deliberate attempts, where I will be sharing what helped me with Git development.

### Blooper of the week

In the spirit of Git being "the stupid content tracker", I have decided to actually live up to that phrase and keep a track of my stupid moments in this section of my weekly reports, in the hopes that there is something here to learn from.

This week had one spectacular incident. I was getting a segfault when trying to parse tokens out of a memory buffer that contained output from `git remote -v`. I wanted to move my pointer upto the first whitespace or newline, and my output was a whole bunch of garbage output followed by a segfault.

Here's an approximation of that code:
```c
static char *parse_token(char **begin, const char *end)
{
	/* ... */
	while (pos != end && (*pos != ' ' || *pos != '\t' || *pos != '\n'))
		pos++;
 	/* ... */
}
```

For whatever reason, I thought of the most convoluted explanations, not realising that they actually don't make any sense. Thoughts like: "Is the memory buffer not having tab separated tokens? Are they separated by some special whitespace character??"

Taking that half baked thought, I piped my output to `xxd` and observed the actual bytecodes of the buffer hoping to find a discrepancy. There was none of course.

Now you, the reader may have already figured it out – the conditions for checking the value of `(*pos = ...` should have been separated by `&&`s. Because they were not, my pointer ended up scurrying off all the way to the `end` which caused other problems down the line and blew up my code.

Lesson learnt: Always do the sanity checks first. The reason your program did not work is probably one of the more easily explainable reasons.
