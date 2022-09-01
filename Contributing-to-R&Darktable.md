R&Darktable is not a community project in the sense that [design by committee](https://en.wikipedia.org/wiki/Design_by_committee) will not be accepted. I'm the designer, I lead the design process and decisions.

# About discussions

-----

*If your English writing level is really too low for communication, keep your messages in your native language and we will find a way to make it work.*

-----

We live in a World where the volume of information and communication has become overwhelming and humans don't have the bandwidth to process all of it. Endless threads and unregulated conversations are actively harming communication by diluting important information and exhausting the reader. **Discussion is solely meant to reach an understanding and proceed to actionable decisions**.

For chat or general questions, please use the [discussions forum](https://github.com/Aurelien-Pierre/R-Darktable/discussions). In PR and issues, please try to stay concise and on-point :

* Lists of technical details (like OS, use of OpenCL, screen size, etc.) should make use of bullet lists.
* Screenshots and drawings can go a long way. 
* If you are replying to a particular point or person, quote the section of text you are replying to. 
* Break your text into paragraphs of roughly 4 to 8 lines, but avoid sending each sentence to a new paragraph.
* Keep in mind everybody speaks English but very few people are native speakers, so try to stick to basic Globish.

Good principles on issues/tickets interactions can be found [here](https://www.yegor256.com/2014/11/24/principles-of-bug-tracking.html).

# Design process

The purpose of a sane design process is to avoid biasing the solutions too early with one's pet design/tech and to avoid getting lost in the technicalities, but to always come back to the core basics and principles of what we are doing: post-processing possibly large batches of raw images for all kinds of output media.

This is backed up by the fact that users rarely know their own needs, or rather, the needs they express are rarely the root of what they actually want. The difficult task of designing is to cut through the branches to go to the root, because solving the root problem usually ends up in more elegant, generic and minimalistic solutions.

## Problems come first

The first step of R&Dt design process is to submit an issue with a feature request. This feature request will focus on the problem to solve and refrain from proposing any solution. The problem will be defined in terms of tasks to achieve in a photographer's workflow or expected visual outcome of the processed image, aka in terms of the end goal to achieve, not in terms of tooling or technicalities thought to be needed. This may lead to a discussion to dig into the roots of the problem, which are usually [well hidden beneath what the user thinks their problem is](https://eng.aurelienpierre.com/2020/04/the-designer-and-the-drilling-machine/).

## Solutions come second

When the definition and scope of the problem is agreed upon between the people involved in the discussion, solutions may be proposed. Further discussion may be necessary to evaluate the drawbacks and benefits of each solution, leading to the best solution being adopted on principle.

Adopted solutions will lead to the issue getting triaged in the [project management Kanban board](https://github.com/users/aurelienpierre/projects/1/views/1).

They might be conditional to researching theoretical and technical aspects to assess their feasibility, in which case they will be triaged into the "To research" column of the Kanban board. The research findings will be added to the original issue until the feasibility of the solution is proven. When it is, the issue will be moved in the "To do" column of the Kanban board.

Adopted solutions might get directly triaged to the "To do" column if they require only well-known tools and techs.

Ideally, the points to test and the testing procedure to validate the prototype should be written even before having a working prototype. At very least, the tests should ensure no regression happened in related features and tools. 

## Prototypes come third

Only the issues triaged in the "To do" column of the [project management Kanban board](https://github.com/users/aurelienpierre/projects/1/views/1) will be worked on, by myself or by anyone willing to tackle them. 

The prototype of the solution will be proposed in a pull request of a topic branch linking the original issue. Topic branches need to be rebased on the `master` branch e.g. `git rebase ustream master` or, if you update your branch locally with new master commits, do `git pull upstream master --rebase` or [globally set up git](https://git-scm.com/docs/git-pull#Documentation/git-pull.txt---rebasefalsetruemergesinteractive) to pull through `rebase` rather than `merge`. This ensures your branch history is kept clean with minimal effort, and keeps the `master` history clean too when your PR gets merged.

When the prototype pull request is reviewed and if it fits the [code quality standards](#code-quality) (see below) while fitting the specifications of the adopted solution, it gets approved and automatically triaged to the "To test/validating" column of the [project management Kanban board](https://github.com/users/aurelienpierre/projects/1/views/1).

## Validation comes fourth

Approved pull requests will be merged early in the `candidate` branch for testing. This branch will always be the master branch with all pull requests pending validation on top. This is meant to help testing from people who are not necessarily up-to-speed with manual git branches merging. Unlike the `dev` branch, `candidate` should not break your edits.

If no bug or breakage is reported after some time and the prototype fulfils its initial purpose correctly, it will get merged in `master` and the related issue will be closed and moved to the "Done" column of the [project management Kanban board](https://github.com/users/aurelienpierre/projects/1/views/1).

If the prototype proves itself unsatisfactory, it may be rejected and another one will need to be worked out.

## When contributors want to go freestyle

If you don't follow those steps, you lose my and everybody's time, so your PR will be refused systematically with no exception. 

Don't ever start working on a PR before the connected issue is in the "To do" column of the Kanban board, because it means there is no adopted solution so you will most likely lose your time.

# Bugs

R&Darktable puts a different interface on top of darktable's core. If you face bugs only affecting the GUI, they most likely belong to R&Darktable's bug tracker. If you face bugs affecting picture processing (garbled output, crash during image processing, styles and presets issues, etc.), they most likely belong to upstream darktable's bug tracker.

# Testing new features

You can keep track of the new features and bugfixes recently merged into the `candidate` and `master` branches by using the project management Kanban board: they will stay in the "Testing" column for some time until I feel they got tested enough. The Kanban board references the base issues connected to these features/bug, which will contain the list of tests to perform to validate that the code is working as intended.

# Code quality

Pull requests that don't match the minimum code quality requirements will not be accepted. These requirements aim at ensuring long-term maintainability and stability by enforcing clear, legible code structured with a simple logic.

1. Procedures need to be broken into unit, reusable functions, whenever possible. Exception to this are specialized linear procedures (no branching) doing tasks too specific to be reused anywhere, but in this case use comments to break down the procedures in "chapters" or steps that can be easily spotted and understood.
2. Functions should achieve only one task at a time. For example, GUI code should not be mixed with SQL or pixel-processing code. Getters and setters should be different functions.
3. Functions should have only one entry and one exit point (`return`). The only exceptions accepted are an early return if the memory buffer on which the function is supposed to operate is not initialized or if a thread mutex lock is already captured.
4. Functions should have legible, explicit names and arguments name that advertise their purpose. Programs are meant to be read by humans, if you code for the machine, do it in binary. 
5. Functions may only nest up to 2 `if` conditional structures. If more than 2 nested `if` are needed, the structure of your code needs to be reevaluated and probably broken down into more granular functions.
6. `if` should only test uniform cases like the state or the value of ideally one (but maybe more) variable(s) of the same type. If non-uniform cases need to be tested (like `IF user param IS value AND picture buffer IS initialized AND picture IS raw AND picture HAS embedded color profile AND color profile coeff[0] IS NOT NaN`), they should be deferred to a checking function returning a `gboolean` `TRUE` or `FALSE` and named properly so fellow developers understand the purpose of the check without ambiguity on cursory code reading, like `color_matrix_should_apply()`. The branching code will then be `if(color_matrix_should_apply()) pix_out = dot_product(pix_in, matrix);`
7. The control flow of `for` loops should not depend on internal variable's state, that is no conditional `break` should be used. Indeed, if the number of loop iterations cannot be planned ahead by the compiler, it will not be able to parallelize efficiently and will be unable to vectorize the code. Breaking loops can happen only for non-computationaly-expensive tasks like finding files.
7. Comments should mention why you did what you did, like your base assumptions, your reasons and any academic or doc reference you used as a base (DOI and URLs should be there). Your code should tell what you did explicitly. If you find yourself having to explain what your code is doing in comments, usually it's a sign that your code is badly structured, variables and functions are ill-named, etc.
8. Quick workarounds that hide issues instead of tackling them at their root will not be accepted. If you are interested in those, you might consider contributing to upstream darktable instead. The only exceptions will be if the issues are blocking (make the soft crash) and no better solution has been found after some decent amount of time spent researching.
9. Always remember that the best code is the most simple. KISS. To achieve this goal, it's usually better to write code from scratch rather than to try mix-and-matching bits of existing code through heavy copy-pasting.
10. R&Darktable as well as darktable are written in C. This language is meant for advanced programmers and gives them absolute power over the machine. Yet most contributors are not trained for C, many of them are not even professional programmers, so C is a dangerous language for them. Keep away from pointers as long as they are not necessary and write the `free()` command right after having allocated a buffer. 

In an ideal world, any PR would follow [design patterns best practices](https://en.wikipedia.org/wiki/Software_design_pattern).

Some random pieces of wisdom from the internet :

> "Everyone knows that debugging is twice as hard as writing a program in the first place. So if you're as clever as you can be when you write it, how will you ever debug it?" — Brian W. Kernighan

> "Without requirements and design, programming is the art of adding bugs to an empty text file." — Louis Srygley

> "Any fool can write code that a computer can understand. Good programmers write code that humans can understand." — Martin Fowler, Refactoring: Improving the Design of Existing Code

> "Always code as if the guy who ends up maintaining your code will be a violent psychopath who knows where you live." — John Woods

> "Whenever I have to think to understand what the code is doing, I ask myself if I can refactor the code to make that understanding more immediately apparent." — Martin Fowler, Refactoring: Improving the Design of Existing Code

> Code is bad. It rots. It requires periodic maintenance. It has bugs that need to be found. New features mean old code has to be adapted. The more code you have, the more places there are for bugs to hide. The longer checkouts or compiles take. The longer it takes a new employee to make sense of your system. If you have to refactor there’s more stuff to move around.
>
> Code is produced by engineers. To make more code requires more engineers. Engineers have n^2 communication costs, and all that code they add to the system, while expanding its capability, also increases a whole basket of costs. You should do whatever possible to increase the productivity of individual programmers in terms of the expressive power of the code they write. Less code to do the same thing (and possibly better). Less programmers to hire. Less organizational communication costs. 
> – Rich Skrenta, <http://www.skrenta.com/2007/05/code_is_our_enemy.html>

> Good programmers write good code. Great programmers write no code. Zen programmers delete code. — [John Byrd](https://www.quora.com/profile/John-Byrd-2)


## Guidelines

1. **Do things you master** : yes, it's nice to learn new things, but R&Darktable is not a sandbox, it's a production software, and it's not the right place to get your training.
2. **KISS and be lazy** : R&Darktable doesn't have 50 devs full-time on deck, being minimalistic both in features and in volume of code is reasonable and sane for current management, but also for future maintenance. *(KISS: keep it stupid simple)*.
3. **Do like the rest of the world** : sure, if everybody is jumping out of the window, you have a right to not follow them, but most issues about software UI/UX have already been solved somewhere and in most cases, it makes sense to simply reuse those solutions, because most users will be familiar with them already.
4. **Be empathetic** : the very fact that you are reading this on Github means you are probably part of the 2% of the population who can read and write computer code. The remaining 98% don't and are usually lost with techs and computers. Be kind with them and avoid them any unnecessary workflow and/or GUI complication. Now, what's necessary or not is a case-by-case judgment to pass and that's where we need to discuss.
5. **Programming is not the goal** : programming is a mean to an end, the end is to be able to process large volume of pictures in a short amount of time while reaching the desired look on each picture. Programming tasks are to be considered overhead and should be kept minimal.

# Context and closing thoughts

The darktable project became high-traffic around 2019 because Lightroom switched to a subscription-based model and darktable had offered Windows versions since 2017 or so. Its main problem was its whole identity and management was, precisely, no management at all and that did not scale well. 

Having more people working with you divides the amount of work by that much, until you reach a team of 5-8 persons. At that point, you need to start structuring the collaborations, which means having more work overall because you have to divert resources and time into making it work as a team. This is more overhead, but with no structure, a growing team is actually less efficient than a couple of individuals. 

This has actively damaged darktable as a project, leading to much noise in the issues tracker, lots of them without follow-ups, a general lack of rigorous testing and a decrease in code quality as well as in usability and general UI/UX. Bugs get merged faster than they get fixed, and there is just so much activity on the master branch that it's impossible to follow carefully what get merged. 

More concerning, the amount of work needed to keep the project afloat is increasing, which is always a bad smell for a project reaching feature-completeness, and some bugs have been there for years now. Some much-needed executive decisions, that would allow to keep the project's ambitions in sync with the available resources, are not made in order to play it nice, which results in overreaching.

Finally, PR get conflicts very fast because different people work on the same parts without dialogue or supervision, and while Git provides some helpers to manage conflicts, ultimately it has to end with a good code reading and manual landmines defusing.

I just want to avoid any of that, and those bad habits now grounded are impossible to change in upstream darktable after 12 years.
