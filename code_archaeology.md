Yes, I wrote it for Manheim. We were writing a new platform that would, in time, replace the AS400 system as the system-of-record; however in the meantime we had to keep the two in sync. So we had data which originated by being entered into the AS400 green-screen; it would be picked up by our synchronizing app and propagated to the new platform. Now, how do we test that E2E? We need to seed data in AS400 somehow, and trying to unpack the byzantine table structure was a path that had already been explored and was more or less unsuccessful as an effective strategy. So the other way is to enter the data by going through green-screen flows. We have SMEs who could do this in their sleep. So: we need a DSL for this problem space, and we need it to be able to be run headlessly on CI. We need it to be intuitive for the people who know the green-screens backwards and forwards, as well as developers. Hence, antimony.

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

# Off-the-Dome Recommendations

### Care and Feeding of Devs:
Create a "precommit" command which can run all the tests in the codebase, as well as any linting and/or analysis that is needed. This *should be* achievable via a Rake task, will probably be a bit tricky to get it working though.

Spend some effort to create Postman collections for exercising the endpoints and checking them in to source control.

### Team practices:

These are structured with a problem statement, the description of the recommendation as a solution, and my opinion on the value it provides.

All of these are practices that I, personally, have "used in anger" (i.e. on a project, for real, in the wild).

............

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

------
------

# The Art of Code Archaeology

> Further reading note: see [Dealing With Creaky Legacy Platforms](https://jonnyleroy.com/2011/02/03/dealing-with-creaky-legacy-platforms/) by my esteemed colleague Jonny LeRoy.

------

> ar·chae·ol·o·gy, *noun*
> the study of human history and prehistory through the excavation of sites and the analysis of artifacts and other physical remains.

> code·ar·chae·ol·o·gy, *noun*
> the study of a codebase's logic through the excavation of classes and the analysis of tests, comments, static analysis, etc

So you've inherited a legacy codebase. A big, messy pile of spaghetti code; it's poorly tested, brittle, and seems to be held together with duct tape and wishes. You dread changing anything because any seemingly minor tweak seems to cause ripple effects, unexpected side effects, and causes the whole thing to fall to pieces. Just thinking about it makes your palms itch. You fantasize about nuking the whole thing from orbit and rewriting it from scratch. It's the bane of your existence.

If it sounds like I'm speaking from experience, it's because I am. 

First off, yeah, it's a bummer. All of your feelings and gripes are valid and correct and don't let anyone tell you otherwise. Hold on to those gripes; you can use them to drive change.

However, this is the reality you're in. No mistake about it, this is dangerous territory: **here be dragons**. The unfortunate news is, you ***do*** have to go into this dungeon and face the monsters within; but there's no reason you have to do it unarmed and unprepared. 

Here are some things to keep in mind as you're adventuring through legacy code:

- It's not your fault, but it **is** your responsibility
- Every bit of code that you hate, was written by someone who came before you in that hateful way for a reason
- Patience isn't just a virtue, it's a requirement
- Nothing about this code is beyond your understanding
- The journey is often worth as much or more than the destination, especially in the beginning

Some tips off the bat:
- Become deeply, intimately, *uncomfortably* familiar with your unit testing framework(s) (JUnit, RSpec, Mocha, Sinon, Jest, etc). It's your new best friend, mother, and erstwhile lover.
- Find an IDE/editor with really good intellisense that lets you navigate quickly and easily. Invest time in trying different things and experiementing with plugins and extensions - anything to reduce your cognitive overhead. Your brainpower will be needed elsewhere.

OK - let's get messy.

------

## Part 1: Establishing understanding of current state before making changes

- Often tests (if any) are at a "higher" level - this leads to setup fatigue (too many dominoes)

#### Write a test
- Use the interface as it appears to be meant to be used and see what breaks
- Kick the tires, abuse the code a bit -- play with the inputs, dependencies
- The test is your playground for exercising the code, you can always delete it later
- Writing a test for existing functionality gives you peace of mind when making changes that you aren't causing regression

SG Example: `Gravy::Orders::ASAPTimeslot` had no unit tests around it, was (is) being implicitly tested by the tests around `Gravy::Throttle::AvailableWantedTimes`. Adding a test around existing functionality helped understand what it's doing, why, and gave confidence for proceeding with changes.

#### Debug
- thingy

SG Example: dur dur

#### Make a diagram
- Map component interactions, create a sequence diagram, whatever works for you to help you understanding.
- Use frameworks like UML and C4 as guidelines or jumping-off points; don't get hung up on the implementation details. Do what feels right and makes sense for you.

SG Example: Making a component(ish) diagram of `Gravy::Api::V1::OrdersController#update` helped understanding of all of the interactions, shed light on the sheer number of code paths as well as the level of orchestration needed for this flow.

------

## Part 2: Testing

Testing is important in general, but it is absolutely crucial to understanding legacy code. Using your automated testing frameworks is a key component to understanding your legacy codebase.

- Use tests as your playground when trying to understand the codebase
- Any "I wonder..." moment could and should lead to a test being written (e.g. "I wonder what would happen if I called methodX with valueY?" - write a test, see what happens, find the answer)

### Levels and types of tests

This list is not by any means exhaustive, but does represent the most common test stereotypes for backend/API systems.

##### Unit

- "Lowest" level of test
- All variables are controlled for and dependencies are mocked
- Should be one test per independent code path (cyclomatic complexity score can be used as a guideline)
- Any external (to the application) systems are stubbed (such as database or 3rd party API)

##### Component (Intra-application integration)

- "Mid" level of test
- Application as a black box
- Given a JSON request body, when it is posted to endpoint X, the response status is 200 and the body is Y
- Stubbing/mocking should happen at the outer boundaries of the system (DB and third party systems) but nothing within the boundary of the application should be stubbed

##### Persistence (Database integration)

- "Mid" level of test
- Should test that the model/repository/ORM agrees with the actual DB schema

##### Contract (Third party integration)

- "Mid" level of test
- Should test that any third party systems respond the way we expect
- Focus is on shape and type of data, not values

##### Smoke / E2E

- "High" level of test
- "Where there's smoke, there's fire"
- This should be the simplest and most critical possible set of exercises of code that will tell you if something is desperately wrong (think happiest possible path)
- To be run post-deployment in all environments
- All integrations are real

### Anatomy of a Well-Formed Unit Test

Here's an example of a class which does independent logic (summing the prices) and also calls a dependency (`RecommendedTipCalculator`).

```ruby
class CheckSubtotaler
  def self.call(items)
    subtotal = items.sum { |i| i[:price] }

    {
      subtotal: subtotal,
      recommended_tips: RecommendedTipCalculator.call(subtotal)
    }
  end
end
```

And here is its test

```ruby
describe CheckSubtotaler do
  it 'should sum all item prices and generate recommended tips' do
    # * Given - setup the input data
    items = [
      { name: 'potato skins', price: 9.95 },
      { name: 'turkey burger', price: 12.05 }
    ]

    # Controlling the variables - this is the data that the dependency will 
    # return. It is intentionally nonsensical because all we care about is 
    # that it is present in the return value and the same as what the 
    # dependency returned
    recommended_tips = {
      fifteen_percent: 1.1,
      eighteen_percent: 2.3,
      twenty_percent: 4.6
    }

    # Expectation + stubbing
    expected_subtotal = 22.0
    RecommendedTipCalculator.expects(:call)
                            .with(expected_subtotal)
                            .returns(recommended_tips)

    # * When - invoke the method under test (subject)
    actual = CheckSubtotaler.call(items)

    # * Then - expect on the return value (actual vs. expected)
    expect(actual).to eq(
      subtotal: expected_subtotal,
      recommended_tips: recommended_tips)
  end
end
```

The dependency is stubbed and also expected on, all variables are controlled for, and the happy code path is covered.

### General best practices

**Don't stub the subject under test**

This defeats the purpose and means that whatever is contained within the stubbed portion isn't being tested. If the code within the method you want to stub is calling a dependency, stub the dependency instead. If it is doing some complex logic, consider extracting this to its own class where it can be tested in isolation.

SG Example: see commit `fae030e2b0190d2e219f87eaac7ada334aa0738c`
`OrdersController#authenticated_device_prepaid_order?` (which is inherited from `BaseController`) was being stubbed in the tests. As a result, commenting out the code inside that method (which was checking the authorization scopes) did not result in any tests failing. The correct thing to do here is to expect on/stub the dependency being called (in this case, the `GetDeviceAuthorizationScopes` service object).

**Avoid implicit testing at the unit level wherever possible**

Implicit testing happens when dependencies of a class are not mocked/stubbed, so every case is covered at the higher level. This leads to tests with long, complex data setup and lots of test cases. The preferable thing to do is to test the dependencies in isolation and then stub them in the test.

SG Example: humminnahummina *cough*. 

**Don't exercise non-test code that is not part of the subject in the test**

Depending on implementation outside of the subject under test for the test to work can lead to issues later, because if that implementation changes, the tests will break for incorrect reasons. 

Example: humminnahummina *cough*. 

------

###### *Thank you,*
###### *-- adrienne*
