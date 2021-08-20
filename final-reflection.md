# Reflections on Working With the Git Community

I got to work and "intern" for the Git Community in the Summer of ’21. This was the first "real" work I have done in my career in Computers. The only thing that can compete with this when it comes to great beginnings is starting your day with a South Indian breakfast [(1)](#footnotes).

![A South Indian Breakfast](https://foodhallcookerystudio.com/wp-content/uploads/2019/02/7th-march-south-indian-breakfast-1024x805.jpg)

Here are the ups and downs of my time here, presented as reflections around a particular theme.

## Git.

My introduction to Git was in my first year of college. I was trying to learn to finally contribute to Open Source after being a fly on the wall in various communities. But my true appreciation of Git's internal model came to me when I started using it more and more, as a part of an Open Source community that I started in my campus.

When 2020 rolled around, I wanted to apply to Git for the Google Summer of Code. I always wanted to work for something I used myself, and Git was an excellent candidate. But I never submitted a proposal. This was due to a combination of:

- me being extremely busy with coursework at college
- letting the words on Git's proposal ideas page intimidate me into not trying.

When 2021 rolled around, I decided to not let this go. I realised that _just starting_ is a big achievement here. The façade of difficulty fades once you start digging into the process of planning your project.

To qualify for being a Git intern, you have to submit a "microproject". The one I submitted as a was just me scratching my own itch with Git. It was about improving diffs for projects written in Racket and Scheme, two languages that I love playing around with. I always recommend contributing to projects that "scratch an itch" of yours. Contributing for the sake of contributing is not nearly as much fun.

This skin-in-the-game approach continued when I started working with my main project, which was converting submodule code from shell to C. I used submodules to maintain [_Atharva's Living Notebook_](https://atharvaraykar.me). This mild dogfooding went a long way into making my work feel far more enjoyable and meaningful (if working on a project used my millions was already not meaningful enough).

## CoViD.

The backdrop was dire. India was getting relentlessly flogged by a deadly second wave of the CoViD-19 pandemic. It was especially bad in our area because even our strict precautions could not stop my mother getting it. Barely a month after her recovery, [my brother got CoViD](https://advait.live/unfortunate-events/) [(2)](#footnotes). This was when the coding period of GSoC began. Because coronaviruses were moonwalking and nae-nae-ing all over my house and neighbourhood, I spent the first two weeks of my project locked down in the safety of my bedroom. The only thing that accompanied me was my laptop.

![Yikes](https://cdn.cnn.com/cnn/.e/interactive/html5-video-media/2021/04/21/20210421-india-cases-780px.png)

Good enough for my Git work. It was a convenient distraction for the chaos of my world in those first two weeks. I am glad that GSoC this year was a part-time commitment. As much as I loved the work, times like these needed other things prioritised.

Of course over the next few weeks, the devastation on the world somewhat subsided, and we got vaccinated. I still wouldn't go outside my door today without putting a lot of thought into it, but at least it's a lot safer within my house.

## Exams.

When I applied for GSoC, I took comfort in the fact that the coding period starts only after my end-semester exams end. But due to the raging pandemic, exams were postponed. To when? No idea. My University only proclaimed that we would know two weeks in advance. And the authorities _insisted_ that the exams be offline. That would mean a travel time close to four hours (I live far from campus), plus three more for the test duration. Add in the time I'd require to actually prepare for the exams [(3)](#footnotes). All in a terrible pandemic.

This threat loomed throughout the first 2 months of my time here, and I was always prepared for a wrench being thrown into my efforts at Git. I kept my mentors informed about this uncertainty, and the silver lining was that they were understanding and empathetic.

Then I got really lucky.

The University saw the light, and gave us options. I love options. The one that I got was: you either give the exams offline, or we will take your grades for the previous semesters and assignments. One of the easiest decisions of my life. I could work on Git uninterrupted again.

## Mentors.

The Git community has a lot of nice people. But I'll start with the ones I interacted the most with. Christian Couder was the most experienced of my mentors. He always came off to me as someone who believes in doing more with less. All my long-winded emails to him were met with terse, to-the-point replies that often got to the heart of the matter. I liked the minimal-intervention style of mentorship. It gave me more space to explore, mess up and learn things all on my own.

Shourya Shukla, my other assigned mentor was a GSoC student of 2020. It was a nice thing to have access to Shourya's code and documentation that helped the foundation of my project. Even nicer was that he was a mentor I had access to.

And then there was Kaartic Sivaraam. He was not actually assigned to me as a mentor on paper, but he mentored me nonetheless. He reviewed large parts of my code. He helped point me to potential solutions (and useful tools) when I was stuck. He also cleaned up the mess left after some of my flimsy code. Sivaraam's fault-finding is why my code has ended up fairly robust so far.

## Confidence.

There's often talk about "10x developers" in a team. In the Git community, I was the "0.1x developer". My output was a lot slower compared to the talented and experienced developers that you'd see on the Git mailing list. If they wanted to, any of them could have done my project in a fraction of the time, and got it over with. But they chose to use this opportunity to better the community by mentoring on projects like these.

![0.1x me](https://www.oldbookillustrations.com/wp-content/high-res/n-d-1884/talking-head-1200.jpg)

When your objective is to learn and gain experience, being a 0.1x developer is a good thing. By being the least intelligent person in the room, the delta of what I learned was enormous. Everything was outside my usual comfort levels in the best way possible.

It also gave me confidence.

I never knew what I was capable of doing out in the "real world". Working with the Git development community is as "real world" as I could possibly get. I have so much to learn. I am still a 0.1x developer. But I am also confident about what I do now. Being a practitioner, and messing up gives you that. My mentors still think I could do better with my confidence (which in their words stands at 4/5 points). I'll of course, gladly take that over the 2/5 I had several months ago.

## Gratitude.

I had a good run with this (yet to be complete) Git project. This was possible thanks to a lot of collaboration, direct and indirect, from the Git community. Thanks to all the people involved. Here's my attempt to credit them, in no specific order.

Junio C Hamano, the Git maintainer, has a reputation for being cool-headed and patient. I can attest to this. Junio reviewed a lot of my work himself, and put up with a lot of my rookie mistakes, and always showed me a path to to make them right. I was amazed by how he could catch on to far-reaching consequences of every change I introduced and see things that would have never occurred to me.

Đoàn Trần Công Danh, Rafael Silva, and Eric Sunshine reviewed my code, and shaped it to become something I can be proud of.

Ævar Arnfjörð Bjarmason also reviewed my work, and pointed me to some helpful resources for debugging Git.

Emily Shaffer, responsible for the excellent "MyFirstContribution" documentation, helped me break into Git development in the first place.

Phillip Wood and Johannes Sixt, along with Ævar reviewed my microproject, which was my first Git contribution.

Philippe Blain, who gave me a detailed guide about his Git workflow setup, and showed me alternate ways to work with Git. I will be adopting some of his approaches as I transition to a part time Git contributor like him.

Johannes Schindelin, who gave me some useful advice on handling the testing for the last part of my project (which is still ongoing).

Felipe Contreras, who created the very useful `git-related` tool that helped me with some of the contributions. He also showed me his Git setup which informed how I handle working with the mailing list now.

Bagas Sanjaya for helping me refine some of my weekly blogs, which has made them clearer.

Jeff King cleaned up one of my patches after it got merged to master.

Michael Haggerty wrote the excellent `git-test` tool that stopped a lot of my messy patch series making it to the list.

And of course, my mentors, Christian Couder, Kaartic Sivaraam, and Shourya Shukla get a boatload of credit for their hard to quantify contributions to my work and to Git.

I also would like to mention the great work done by the other intern for this Summer of Code, ZheNing Hu. English is not his first language, but that obstacle did not stop him from making Git better. You can check out his work [here](https://adlternative.github.io/GSOC-Git-Final-Blog/).

## What's next?

I have so much to learn and explore, and I will fondly remember my time at Git as the first significant milestone for that journey. I will still continue the [leftover bits](https://atharvaraykar.me/gitnotes/final-report.html#whats-next) of my work. I want to leave some kind of residual value beyond my active involvement in these last few months. I hope to set aside some time  mentoring and growing the community in the future. I will be spending more time diversifying my knowledge and learning other things as well.

This blog will continue, with a renewed purpose. I will be writing down important things I learnt in my time and bugs that new contributors could work on. I believe that writing guides and making contributions easier has some of the best effort-to-impact characteristics that I am looking for.

## How to contact me (and other plugs)

Feel free to contact me (raykar.ath@gmail.com) if:

- You are a new contributor looking for specific guidance or mentorship.
- You are hiring, and need someone who is passionate, persevering, and enjoys hard problems. I especially <3 working on Open Source projects.

If you are a college student from India, have a look at the community for bottom-up learning I am trying to build at [Universities for FOSS, India](https://unifoss.github.io/).

## Footnotes

[1] I am hungry as I write this.  
[2] Thankfully, no hospitalisations. We are all fine now.  
[3] This one is partially on me, but I could not get myself to pay attention to any online class this whole semester.  
