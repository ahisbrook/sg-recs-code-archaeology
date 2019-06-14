# Off-the-Dome Recommendations
------

### Care and Feeding of Devs:
Create a "precommit" command which can run all the tests in the codebase, as well as any linting and/or analysis that is needed. This *should be* achievable via a Rake task, will probably be a bit tricky to get it working though.

Spend some effort to create Postman collections for exercising the endpoints and checking them in to source control.

### Team practices:

These are structured with a problem statement, the description of the recommendation as a solution, and my opinion on the value it provides.

All of these are practices that I, personally, have "used in anger" (i.e. on a project, for real, in the wild).

#### + Kickoff & Desk Check

*The problem:*
***Lack of clarity on story requirements causes story churn and increases dev time***

*The solution:*
Do kickoffs & desk checks with the tech lead and BA (or whoever the appropriate people are). The time frame for each of these huddles should be ~5 minutes.

Kickoff happens when a dev picks up a story. The purpose is to come to a final agreement on the purpose, scope, and acceptance criteria of a story. Anything missing from the ticket can be added, or anything that should have been removed can be removed. This conversation is held under the assumption that all grooming etc. has been done - the dev can ask clarifying questions, but this is not the time to try to re-negotiate the scope or ACs. If the agreement is hard to come to, taking too much time, etc. then the story should be moved back to the analysis stage for further grooming. 

Desk check happens when a dev feels they are dev-done with a story. This is a mini-showcase of functionality to the tech lead and BA. The dev repeats back the previously agreed upon purpose, and runs through the functionality AC by AC. The BA asks clarifying questions and asks to see any scenarios they see fit. The tech lead asks about any technical or cross functional requirements ("Logging? Metrics?"). The purpose of this is to get faster (compared to the development -> QA lifecycle) feedback on any glaring misses. 

*Value:*
This is a very effective way of minimizing misunderstandings, misses, and subsequently reducing story churn (the number of times a story goes from development to QA and back).

#### + Captains

*The problem:*
***Things which are everyone's responsibility end up being no-one's responsibility***

*The solution:*
Have a rotating point person ("captain") *who **isn't** the tech lead* take ownership of whatever troublesome aspect or task needs attention.

Example: a "release captain" who is responsible for making sure the release goes smoothly and nothing is missed. In this example, the release captain is not personally responsible for solving any issues that arise, but **is** responsible for looping in the right people, should problems occur. They sre also responsible for chasing down devs who need to merge commits, etc.

*Value:*
This is a wonderfully effective way of ensuring that important tasks are given the attention they need, as well as generally increasing team knowledge, feelings of ownership, and taking some of the workload off of the already over-burdened shoulders of the people to whom these tasks would otherwise fall (tech leads, QA leads, PMs, etc).

#### + Champions

*The problem:*
***The tech lead has too much on their plate***

*The solution:*
Have designated point people for different aspects of the project. Their role would be to take on the tasks and responsibilities of the tech lead with regard to that specific aspect (meetings with third parties, analysis, working with the BA to slice stories, etc).

Example: a "feature champion" who is responsible for the technical analysis and context of a feature. 

Variation: a "tech improvement champion" who is responsible for a single identified tech improvement or cross-functional concern (e.g. adding contract tests) and owns and drives the implementation of that tech task. (This is a great way to give junior devs a manageable and relatively low-stakes responsibility, as well as a feeling of ownership, and as manage the issue of how and when to address badly needed tech improvements that otherwise wouldn't be prioritized),

*Value:*
This is another great way of taking load off of the tech lead's shoulders, as well as giving all members of the team the opportunity to practice leadership and feel ownership over some aspect of the project. 

#### + Pairing

*The problem:*
***Knowledge sharing, upskilling, and onboarding issues***

*The solution:*
See "Pairing with a capital 'P'" (link goes here)

*Value:*
Pair programming as a practice has so many benefits that pay dividends as time goes on: it reduces knowledge silos, it upskills less experienced devs, it onboards new devs, it promotes communication and collaborative working, it leads to innovative and creative solutions. 