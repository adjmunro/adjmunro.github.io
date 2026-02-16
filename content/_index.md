+++
title = "adjmunro's blog?"
description = "Notes and discussions of various things I've learnt"
template = "index.html"
+++

Well, there's nothing here right now, because I prefer to write in markdown (due to doc-comments, github, and *because i'm not a f\*cking web dev - servers **terrify** me and i **scream** in the face of javascript* lol; i'm only 80% kidding too *and 20% seen through by my parasocial relationship with theo@t3 too...*)

### I also hope to (eventually) post about the following topics:

- Clean architecture in app development, of which you can already read most of it [here](https://gist.github.com/adjmunro/048929eccfd8af176545e4ff609839e9). However, i feel i should still add a section on practical usecases and a note that many of the challenges may be uniquely kotlin problems, due to the visibility modifiers available to us.
- UI architecture: MVI, MVVM, etc.
- Why Android's has been completely borked for about 8 years, since FragmentManager went out of fashion, and before the hopeful promise that is navigation 3. Why XML navigation graphs, the safe args plugin, and compose navigation 2 are bandaids on bandaids, and a practical stratergy that ensures type-safe navigation without circular gradle module dependency issues (the trick is to get rid of that awful xml, and put navarg data classes in the domain layer, then have the app associate navarg classes to fragments in a map to instantiate with FM. Yes it works; yes, navigating in code is painless. Sorry, but your navigation library is dumb af.)
- The issues with Kotlin's error handling an Result class (and why I wrote my library, project-knomadic)
- The issues with Kotlin's number classes *that lack f\*cking math operators* and how i fixed it in project-kitty
- Wrestling with dependabot for maximum quality-of-life
- My convention plugin journey
- The hacks I pulled to migrate Dagger2 (yes, *that dagger2*, not dagger hilt) to Koin.
- Why/how I used Koin to eliminate build flavours, and redo our local feature flagging
- Git worktrees
- My dotfiles journey (this one's pretty cool... though it's far from over)
- How to publish a library
- How to build a custom webbrowser (it's shit, but it's mine!)
- Crazy experiments to learn linux by installing 5 distros at once... and then being unable to boot into arch because my bootloader can't find the file for some reason? idk. working on that

And that's not all - I plan to post about a variety of non-programming interests or just hard stuff i learn along the way. Also my CV... Speaking of CV, if you're trying to contact me on LinkedIn, i've been locked out for like a year because they want a MFA code i have no record of setting up or some very personal info... >.>
