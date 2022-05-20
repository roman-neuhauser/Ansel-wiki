The new filtering feature, forcefully introduced for darktable 4.0, was the straw that made the camel's back break and made me realize that I had been working with a team of idiots for too long, which was cause to start this fork. It is a solution in search of a problem, over-engineered, introducing features that nobody needed in the worst possible way, under cover of what started as a mere refactoring. Then, idiots got trigger-happy and couldn't resist adding bloat to what was just refactored, just because they could. Oversight of the problem to solve was long lost, because there was no problem in the first place.

The smell that the feature is designed with the ass is that each bugfix is adding more lines of code, because edge-cases need additional branching to be handle, which is telling that the base logic is flawed. Also, the whole point of a refactoring is to get rid of edge-cases, so mission failed. After a while, wise men would smell the problem and cancel the project, but the darktable's team [escalated commitment](https://en.m.wikipedia.org/wiki/Escalation_of_commitment) to protect the outcome of [3 months of hard work](https://en.wikipedia.org/wiki/Sunk_cost) from the only rational outcome : reverting. Even though, at that point, the goal of the fixes shifted from "making it better" to ["making it less bad"](https://github.com/darktable-org/darktable/pull/11848), acknowledging that it was too late to fix before release, while nobody said we had to stick to the release calendar if we knew that the release would not be ready and stable enough.

A quick log for shit and giggles of the [technical debt](https://en.wikipedia.org/wiki/Technical_debt) diff introduced from the new filtering feature:

*+x / -y are the number of lines of code added/removed. For a given feature, best practices are to have the lowest number of code lines at the end because each line is a chance of bug and long code is hard to maintain, debug and extend. See [overengineering](https://en.wikipedia.org/wiki/Overengineering)*

-  collect filtering widgets #11066 : +7631 / -2113
-  Filename filter speedup #11550 : +65 / -85
-  collect: fix range selection and click #11636 : +55 / -57
-  range : allow shift-click to select a range #11665 : + 7 / -0
-  filtering : add grouping filter #11721 : + 817 / -29
-  range : reset bounds correctly when typing in entries #11807 : +7 / -2
-  filters : keep off/remove visible but dimmed if pinned #11811 : +8 / -9
-  filtering : fix an issue with conf value overwritting #11819 : +17 / -5
-  filters : update rating numbers correctly #11809 : +57 / - 37
-  filters : always show the pin icon #11808 : +44 / -53

Total: + 8708 / -2390

----

Same for the phweyland's version, which achieved mostly the same goal minus the shitload of configuration options:

- Add text & color label filters #10694 : +502 / -87
- text filter: don't add wildcards on empty entry #11264 : +13 / -10
- Text filter: fix bug on query #11267 : +6 / -7
- color label filter: inverts icons symbols #11293 : +4 / -8
- filter (text & colors label): save their state between dt sessions #11303 : +134 / -94

Total: +659 / -206

----

PR sort-of merging both features:

- recent history and rating filters "soft" revert #11801 : + 793 / -46

Next time a refactoring project is started in darktable, please kill it before it ends.