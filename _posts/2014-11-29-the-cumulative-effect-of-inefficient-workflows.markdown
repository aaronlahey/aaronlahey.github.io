---
layout: post
title:  "The Cumulative Effect of Inefficient Workflows"
date:   2014-11-29
categories: workflows
---

In 2009, I started an internship at a Los Angeles based audio/video post
production studio. The audio engineer I was learning from was an avid musician
just like myself and we quickly bonded over our mutual interest of recording and
producing music. One day we started talking about audio equipment and I
mentioned how I thought it was ridiculous that anyone could justify spending
thousands of dollars on a vintage piece of outboard equipment. After all, when
you’re producing a song it’s not at all uncommon to have 25 or 50 tracks of
audio (and sometimes more). How could the tiny bit of vibe that a vintage piece
of gear might have be worth the three-fold price increase over a modern reissue
of the same model? Especially when it would be just one tiny part of the
ensemble?

He smirked. I could tell from his face this was an argument he’d heard before
and that he happened to have a well-thought-out rebuttal. He was quick to
point out that I wasn’t considering the effect that elevating every track to
that level, or alternatively, allowing every track to be of less-than-optimal
quality would have on the final product. I was completely ignoring the
cumulative effect of striving for the best possible solution at every step.

I think, as software craftspeople, we’re equally, if not more, affected by this
same cumulative effect. Generally speaking, I believe the code we write is a
prime example that we’ve recognized that idea. We relentlessly contemplate the
best name for classes, variables, etc. because we know that effort will reward
us with a system that unquestionably reveals its intent. We scrutinize every
design decision we make because we know the maintainability of the system
ultimately relies on it. We strive to produce software that cumulates into
clean, easily-modifiable, performant systems and we know the path that gets us
there is paved with countless small, yet important decisions. I do, however,
question whether we as professional developers give that same attention to the
way in which we develop -- all of the seemingly insignificant actions that
ultimately result in the software we create.

It seems logical that we wouldn’t. The companies we work for and the consumers
of our products don’t care how we develop as long as the things we deliver are
on time, on budget, and meet or exceed any quality expectations. However, as
*producers*, we should always be focused on whether the *act of producing* is as
efficient as it could be.

It wasn’t until I started working with other developers that I really started
thinking about inefficiencies in development workflows. I remember the first
moment it popped into my head… I was pair programming with a colleague and the
code we were working on lived near the bottom of the file. I don’t remember
exactly why I needed to jump to the top of the file (I probably needed to
`require` something), but I proceeded to type: `shift, colon, one, enter`. I’m sure
my pair grit his teeth a little but nothing was said. I did my business at the
top of the file and to get back to the bottom I typed `shift, colon, nine, nine,
nine, nine, enter`. That was obviously too much for him to bear so he promptly
chuckled and informed me that I could get to the bottom of a file simply by
typing `G`.

It’s not that I didn’t know `:9999<cr>` was a ridiculous way to get to the bottom
of a file, I just didn’t care. It was only seven keystrokes and it worked 99.99%
of the time. Later that day, I thought about how many times I would actually
need to dive to the bottom of a file in a day, a week, or a year. When taking
into account the effort required to perform those five extra keystrokes over the
course of a thirty year career, it seems *crazy* that I would choose that over
sacrificing sixty seconds to Google a better way. Yet, similar (and worse)
inefficiencies exist in every developer’s repertoire of bad habits. We complain
about the JVM startup time yet spend twice that holding `j` to traverse 100 lines
in Vim. We repeatedly manually rename series of variables instead of spending
four hours one Friday afternoon really harnessing the power of regular
expressions.

The idea of streamlining your workflow doesn’t end with leveling up your Vim
skills. As software professionals we are committed to test-driven development.
We reap many benefits from this discipline, one of which is a fast feedback
loop. We create fast unit tests that give near-instantaneous feedback when
functionality has been fully implemented or a regression has been introduced and
we run them repeatedly, often many times per minute. Surprisingly, however,
testing workflows still exist that are not conducive to this instantaneous
feedback loop.

It’s incredibly easy to switch panes in tmux, hit the up arrow and enter
(assuming you’ve previously run the tests), and watch your entire test suite
run. It certainly gets the job done but it eliminates one of the major benefits
(in my opinion) of a fast feedback cycle: minimizing mental context switches.
The second you have to think about switching windows in tmux or watch a parade
of passing tests fly across your screen you’ve made a mental context switch; the
problem at hand is no long front and center in your mind. Worst case scenario,
you may even have time to fire up Twitter or check your email.

I firmly believe in taking breaks from your work. It can help you form new ideas
and approach existing problems with new perspectives, and its worth mentioning
that despite the varying amount of steps you’ll find the creative process broken
into, letting an idea stew in your brain is pretty consistently one of them. It
has not been my experience, though, that these kind of micro-breaks
mid-problem-solving serve as effective incubation periods. They merely serve as
distractions causing you to lose your hard-earned mental state.

In addition to the effect this has on your mental state and how you approach
problems, it suffers from the same problems as before: wasted time and
keystrokes. I have a few custom tmux bindings, but even with those in place it’s
eight keystrokes and a hand shift off home row to leave Vim, run my tests and
get back into Vim. Think of how many times in your career you should be running
tests—the answer is a lot. Now think of the productivity you could gain by
reducing this one thing you do multiple times per day by 75%. How much more
often would you run those tests? How many more ideas would you experiment with?
How much faster would you finish a feature if running your tests was an
instantaneous, mindless reflex instead of a conscious mental activity?

This is, in fact, something I did decide to tackle. A large majority of the work
I do is in Ruby so it probably comes as no surprise that I run RSpec tests
countless times per day. I wanted to be able to execute the exact specs I needed
with the minimum amount of effort possible and to accomplish this, I decided to
write my own RSpec runner.

The most important thing I wanted out of a runner was to be able to run the
specs for the file I currently had open in Vim. When on a spec file, I wanted it
to run that file. When on a production code file I wanted it to run its
associated spec file. In addition to this, I wanted a way to run all the specs
for the project and a way to run only the spec my cursor was on. Another
requirement of the runner was that it support running the specs in Vim (as you
would any other shell command) and running them in a different window from Vim
(by using a named pipe<sup>1</sup>). With this new runner I can have both my
tests and my editor in view at the same time, run the whole suite with
`<leader>a`, run the current file’s specs with `<leader>r`, and run the spec under
my cursor with `<leader>l`. There is next-to-zero ceremony involved in running
the specs I want to run. It has become a mindless reflex instead of something
I shift to, setup, and execute.

I’m not explaining my RSpec runner because I think its some sort of
game-changing test runner. I bring it up simply as an example of a solution to
something I saw as a detriment to my development experience. Something I, for a
long time, felt wasn’t an issue, but after taking a step back and reflecting on
the way in which I was developing, realized would add up to significant wasted
time and effort. It was incredibly simple to write that script and modify my
.vimrc accordingly and that small investment of time and effort is still paying
dividends.

I feel its important to note that I’m incredibly far from having a consistently
efficient workflow for all situations. There are developers who most definitely
have this worked out far better than I do. The beauty of it is, and the reason a
post like this is worth writing and can possibly exist, is because there is no
end goal. There is not an “ideal” workflow that everyone should be using, partly
because personal preferences vary and partly because there’s always room for
improvement. We do, however, owe it to ourselves to be constantly critical of
the way we do things and to be on a never-ending search for something better;
not just a way to create better software, but also a *better way* to create better
software.

Classical guitarists invest thousands of hours stressing over millimeter
differences in finger placement or what hand position should be used to
contribute to the overall fluidity of the piece being played. As software
craftsmen and craftswomen on a path to mastery we owe a similar attention to
detail to our own techniques and the cumulative effect it has on the development
process as a whole.

<hr />

<sup>1</sup>I learned about the named pipe technique [here](https://www.destroyallsoftware.com/screencasts/catalog/running-tests-asynchronously).
