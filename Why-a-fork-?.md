# TL;DR

1. the darktable team is violating every best practice of rational design and engineering: focus first on the problem to solve from the user-perspective, deal with the technicalities later, never start designing a solution before being done defining the problem and its context, and first and foremost : KEEP IT STUPID SIMPLE !
2. the darktable team is getting distracted by niche features and cosmetics while forgetting the real goal of the application: culling what came out of the camera's memory card and outputting post-processed images as fast and as good as possible,
3. every attempt of redesign ends in threads flooded with irrelevant statements and unqualified opinions where past mistakes are used as precedent to keep digging in the same hole,
3. the darktable team sees no problem with the three previous statements, so why change anything ? Everything is as good as it will ever be.
4. after several tries, there is no reason to think that it will ever change, and ill-designed features keep on piling (lighttable overlays, module groups, collections filters), harming both usability and performance for the sake of secondary fancy features,
5. going forward, there is no future that doesn't involve compromising usability, and no reason to keep on trying. Irreconcilable priorities need to end in break-up.
6. anyway, the darktable project has become a victim of its hype and reached a number of contributors incompatible with its present form of management (aka none) or design vision (again, none), which ends in [design by committee](https://en.wikipedia.org/wiki/Design_by_committee) riddled with [groupthink](https://en.wikipedia.org/wiki/Groupthink). The internet is full of examples showing why design by committee is damaging despite its benevolence:
   * https://blog.codinghorror.com/the-pontiac-aztek-and-the-perils-of-design-by-committee/
   * https://www.productplan.com/learn/how-to-avoid-design-by-committee/
   * https://medium.com/swlh/how-to-avoid-design-by-committee-and-why-you-should-at-all-costs-8e8ce78c1058

# Details

Even I don't have the slightest idea of what many preferences in darktable do. The problem is all those preferences/options trigger special behaviours that may interfere with each other and produce edge cases (heuristics) difficult to predict, debug, refactor and simplify. They make the code hard to read and therefore hard to maintain. But "cool new stuff" is favoured over maintainability and ease of use, and I'm against adding more code that will add more problems that will need more code to fix, because it obviously is a self-feeding loop of madness.

But that wouldn't be so bad if so many changes since 2020 hadn't been just options for the sake of it. Triple click support in shortcuts declaration ? Timeline view ? Modules groups with presets ? Lighttable overlays options ? Nobody needed them, which would be fine if most of them were not broken in subtle ways (look out for "I don't reproduce" or "works for me").

Let's run a quick computation : there are 7 flavours of thumbnails overlays × 5 different lighttable views × 3 base color themes = 105 combinations of options to style individually in CSS themes… Guess how many are currently broken in upstream darktable ? 3 overlays × 3 views was more than enough, but no… We had to make it impossible and annoying for everyone involved.

This habit, now grounded in darktable's DNA, to keep piling up useless options makes darktable more difficult to learn, understand, use, debug and maintain. This is not progress, this is not design (aka the process of solving problems through the simplest path), this is just geeky kindergarten to keep middle-aged middle-class white men busy with code on saturdays. It's typical Linux bullshit where people enjoy the feeling of power they get from an overwhelming and cryptic UI more than the result of the work they achieve through it. Breaking news: software is a tool, a tool is meant to do a job, only the result matters and only the idiots fall in love with their hammer.

When it comes to offering options, we need to also mind a very important thing: the best way to hide a tree is in the middle of the forest. When you find yourself lost in a forest of options, it becomes hard to argue that all these options empower you : they don't. Good luck finding your way now in darktable's preferences.

Many GUI interactions in darktable don't comply with what's deemed standard in 2022, and not even with Gnome or KDE defaults, because apparently we need to outsmart everyone by reinventing even the most basic things like keyboard navigation and mouse hover events. Sorry, but the 2020's is not the right time to pretend to reinvent GUI interactions, and the result is anyway not as clever or usable as the devs think.

Actually, the harsh truth is very few of these many options make you more productive. But they surely lose your time since you need to read the doc or (re)configure shortcuts to be able to perform basic tasks like bulk files selection or filtering. And since your preferences get nuked at pretty much each update now, the time you spent tuning and tweaking all that cannot even be seen as a long-term investment: it's just a dry loss.

Finally, the abuse of Gtk events and poorly-coded custom widgets makes the soft sluggish and laggy since too many things happen "real-time" whenever you move the cursor in the window, simply because too many Gtk widgets capture mouse events for disputable tasks, often duplicated. Gtk is a disgrace (single-threaded on CPU only and overall inefficient), let's keep its use the the strict minimum, shall we ?

Basically, git pulling darktable now is an anxiety-triggering task meant to answer a daunting question: "what did they break this time ?". I have had enough of that shit. Adding solutions in search for a problem is not ok when it degrades basic usability. Defending solutions with "me likey" is childish and stupid, the quality of a tool is to be assessed by the number of steps and the time needed to achieve a specified task, and what people (devs and users) like is irrelevant.

Trying to solve all that upstream has failed because old-time users have grown fond of all the quirks and we can't even agree on a definition of the problems through endless threads where mock-ups and sketches appear before we even know what specific problem we need to solve for what specific type of user. I have grown weary and tired of that nonsense. I have no faith in a team of amateur photographers playing amateur devs to ever improve UI and I'm tired of arguing with people who think designing is just drawing mock-ups and will not take it seriously.

Let's do less better. That is, remove the bloat, stop trying to account for every anecdotal sub-optimal workflow, and make actual management choices and decisions to build a consistent set of tools for a bunch of real, defined, use cases.

# The one about compromise

Compromise is a bullshit buzzword that people are taught to like even though they never questioned its premisses. Compromise is when everybody have their needs only met half-way, which is supposed to maintain unity by sharing equally half of the losses. Though half-satisfied may well equate to fully frustrated, by the rules of the arithmetic of needs.

See, if tonight my wife wants to go to the cinema and I want to go to the restaurant, we have incompatible needs that must be resolved. The compromise would be to go the restaurant, have half a meal, then go to the cinema and watch the end of the movie. That's a shitty time for both : better stay home and do nothing, it will cost less. That's compromise.

Now, if we back up a bit, and dig deeper into our respective needs, we might find out that I want the restaurant because I'm too tired to cook tonight, let alone do the dishes, and she wants the cinema just to venture in imaginary worlds. Now that we have our real needs expressed, it's easy to fix because we actually want the same thing : to relax. The solution is to order food to take out and have a Netflix-and-chill at home. If she wants the cinema because there is this movie she really wants to see and it's the last day it's displayed, the fix is to fast-food somewhere and go watch it.

See, nobody has a **need** to go to the restaurant or to the cinema, per-se. The need is on the experience that these places give you: relax and forget your day, perhaps even get pampered and cared for. So the way of avoiding compromise is to dig deeper into those real needs, beyond the needs people expressed. And compromise should be avoided, and since that takes some skill and time, compromise is chosen by default.

But there might be cases where the real needs are still incompatible after careful analyses, in which case compromise is the only way to go. However, if your whole interaction mode with your significant other (or whatever social group you are in) ends up in systematic compromise, and you start compromising on the important things, you need to reevaluate the very existence of your relationship because it's fulfilling for nobody. But the fear of ending up alone is stronger for most people than the promise of having a more satisfying life, so people make up all sorts of excuse to not do it and to preserve the "unity of the community" above everything else, including the basic needs and happiness of its members.

Which is crazy. 