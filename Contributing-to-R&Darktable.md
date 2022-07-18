R&Darktable is not a community project in the sense that [design by committee](https://en.wikipedia.org/wiki/Design_by_committee) will not be accepted. I'm the designer, I lead the design process and decisions.

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

## Prototypes come third

Only the issues triaged in the "To do" column of the [project management Kanban board](https://github.com/users/aurelienpierre/projects/1/views/1) will be worked on, by myself or by anyone willing to tackle them. 

The prototype of the solution will be proposed in a pull request of a topic branch linking the original issue. Topic branches need to be rebased on the `master` branch e.g. `git rebase ustream master` or, if you update your branch locally with new master commits, do `git pull upstream master --rebase` or [globally set up git](https://git-scm.com/docs/git-pull#Documentation/git-pull.txt---rebasefalsetruemergesinteractive) to pull through `rebase` rather than `merge`. This ensures your branch history is kept clean with minimal effort, and keeps the `master` history clean too when your PR gets merged.

When the prototype pull request is reviewed and if it fits the [code quality standards](#code-quality) (see below) while fitting the specifications of the adopted solution, it gets approved and automatically triaged to the "To test/validating" column of the [project management Kanban board](https://github.com/users/aurelienpierre/projects/1/views/1).

## Validation comes fourth

Approved pull requests will be merged early in the `candidate` branch for testing. This branch will always be the master branch with all pull requests pending validation on top. This is meant to help testing from people who are not necessarily up-to-speed with manual git branches merging. Unlike the `dev` branch, `candidate` should not break your edits.

If no bug or breakage is reported after some time and the prototype fulfils its initial purpose correctly, it will get merged in `master` and the related issue will be closed and moved to the "Done" column of the [project management Kanban board](https://github.com/users/aurelienpierre/projects/1/views/1).

If the prototype proves itself unsatisfactory, it may be rejected and another one will need to be worked out.

## When contributors want to go freestyle

If you don't follow those steps, you lose my and everybody's time, you piss me off and you get called names.

PR that don't come from an issue following those steps will be refused systematically with no exception. 

Don't ever start working on a PR before the connected issue is in the "To do" column of the Kanban board, because it means there is no adopted solution so you will most likely lose your time.

# Bugs

R&Darktable puts a different interface on top of darktable's core. If you face bugs only affecting the GUI, they most likely belong to R&Darktable's bug tracker. If you face bugs affecting picture processing (garbled output, crash during image processing, styles and presets issues, etc.), they most likely belong to upstream darktable's bug tracker.

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

In an ideal world, any PR would follow [design patterns best practices](https://en.wikipedia.org/wiki/Software_design_pattern).