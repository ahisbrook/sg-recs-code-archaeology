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

Caveat: I will be focusing primarily on talking about testing in this document, but there are many different aspects to talk about regarding this subject, and this is by no means a full and complete

OK - let's get messy.

------

## Part 1: Getting Your Bearings

The first hurdle to working with any legacy codebase is just understanding what on God's green earth is even happening. Establishing an understanding of current state and functionality is key. There are many tools in your toolbelt that you can use for this task. My top three are:

#### Write a test
- Use the interface as it appears to be meant to be used and see what breaks
- Kick the tires, abuse the code a bit -- play with the inputs, dependencies
- The test is your playground for exercising the code, you can always delete it later
- **Writing a test for existing functionality gives you peace of mind when making changes that you aren't causing regression**

SG Example: `Gravy::Orders::ASAPTimeslot` had no unit tests around it, was (is) being implicitly tested by the tests around `Gravy::Throttle::AvailableWantedTimes`. Adding a test around existing functionality helped understand what it's doing, why, and gave confidence for proceeding with changes.

#### Debug
- Effective debugging is crucial to understanding and navigating legacy code
- `pry`, `byebug` etc. (to name  a few) are the kinds of tools you will need
- Exercising the code you need to understand in debugging mode will increase your understanding of data flow, data mutation, and state changes

#### Make a diagram
- Map component interactions, create a sequence diagram, whatever works for you to help you understanding.
- Use frameworks like UML and C4 as guidelines or jumping-off points; don't get hung up on the implementation details. Do what feels right and makes sense for you.

SG Example: Making a component(ish) diagram of `Gravy::Api::V1::OrdersController#update` helped understanding of all of the interactions, shed light on the sheer number of code paths as well as the level of orchestration needed for this flow.

#### Static analysis
- Use any of the myriad static analysis tools at your disposal to quickly crunch some numbers
- Code coverage (with a report on covered and uncovered code) - this can help you figure out which parts of the codebase are probably the "least safe" to make changes in, and target places for increased test coverage.
- Cyclomatic complexity scores - A cyclomatic complexity score for a given method is the number of independent code paths through that method. You can use this as a baseline for the number of unit tests needed around that method.

#### Use git blame to your advantage
- If you find something particularly knotty or hard to understand, look for clues as to who in the organization may have any context
- Git blame is useful for this
- Comments - a heavily commented section of code is probably one which someone else has wrestled with in the past; see if you can find out who and talk to them

------

## Part 2: Testing

Testing is important in general, but it is absolutely crucial to understanding legacy code. Using your automated testing frameworks is a key component to understanding your legacy codebase.

Making sure the code is covered to your comfort level is also crucial to working with legacy code. Writing tests for existing functionality is an exercise you should engage in frequently. **Once you are confident that the existing code is adequately covered, you can safely make changes to it.**

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

A basic test (of any level) would consist of the following:
- **Subject** - the subject is the thing under test (can alternately refer to a class or individual method)
- **Setup/Input** - this is the data you are invoking the subject with
- **Expected** - this is the expected return value of the subject given the input
- **Actual** - this is the output of the subject, and you should assert that it matches the **expected**
- **Dependencies** - these are any external code called by the subject, and should be expected on and mocked/stubbed

***The degree to which dependencies are stubbed is primarily what differentiates test levels from one another.***

Here's an example of a class which does independent logic (summing the prices) and also calls a dependency (`RecommendedTipCalculator`).

In this example, `CheckSubtotaler` is the **subject** under test.

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

And here is its test file:

```ruby
describe CheckSubtotaler do
    context 'items are present on the check' do
      # Given - set up the data
      let(:items) do
        [
          { name: 'potato skins', price: 9.95 },
          { name: 'turkey burger', price: 12.05 }
        ]
      end

      it 'should sum all item prices and generate recommended tips' do
        # Set up a controlled for value
        recommended_tips = {
          fifteen_percent: 1.1,
          eighteen_percent: 2.3,
          twenty_percent: 4.6
        }

        # Expectation + stubbing with controlled for value
        expected_subtotal = 22.0
        RecommendedTipCalculator.expects(:call)
                                .with(expected_subtotal)
                                .returns(recommended_tips)

        # When - invoke the method under test (subject)
        actual = CheckSubtotaler.call(items)

        # Then - expect on the return value (actual vs. expected)
        expect(actual).to eq(
          subtotal: expected_subtotal,
          recommended_tips: recommended_tips
        )
      end
    end

    context 'items is nil' do
      # Given - set up the data
      #   This level of explicitness, while technically unnecessary,
      #   is helpful for maintainability
      let(:items) { nil }

      it 'should throw an exception' do
        # When + Then - invoking the method and expecting on the raising of
        # an error
        expect { CheckSubtotaler.call(items) }.to raise_error('Nothing here')
      end
    end

    context 'items does not respond to sum' do
      # Given - set up the data
      #   This level of explicitness, while technically unnecessary,
      #   is helpful for maintainability
      let(:items) { Object.new }

      it 'should throw an exception' do
        # When + Then - invoking the method and expecting on the raising of
        # an error
        expect { CheckSubtotaler.call(items) }.to raise_error('Unable to sum')
      end
    end
  end
```

The dependency is stubbed and also expected on, all variables are controlled for, and the every code path is covered.

### General best practices

**Don't stub the subject under test**

This defeats the purpose and means that whatever is contained within the stubbed portion isn't being tested. If the code within the method you want to stub is calling a dependency, stub the dependency instead. If it is doing some complex logic, consider extracting this to its own class where it can be tested in isolation.

SG Example: see commit `fae030e2b0190d2e219f87eaac7ada334aa0738c`
`OrdersController#authenticated_device_prepaid_order?` (which is inherited from `BaseController`) was being stubbed in the tests. As a result, commenting out the code inside that method (which was checking the authorization scopes) did not result in any tests failing. The correct thing to do here is to expect on/stub the dependency being called (in this case, the `GetDeviceAuthorizationScopes` service object).

**Avoid implicit testing at the unit level wherever possible**

Implicit testing happens when dependencies of a class are not mocked/stubbed, so every case is covered at the higher level. This leads to tests with long, complex data setup and lots of test cases. The preferable thing to do is to test the dependencies in isolation and then stub them in the test.

SG Example: see commit `7b027da3327886fef869b73aa36caa4bab4b2d56` for the state I'm talking about
`available_wanted_times_spec.rb` is doing a lot of implicit testing of underlying functionality. Case in point: Coverage of the functionality of  `ASAPTimeslot` takes up approximately **400 lines** of the test code in `available_wanted_times_spec.rb` (between lines `322` and `718`). The correct thing to do in this case is add coverage in isolation around `ASAPTimeslot` and then stub out the dependency in the test around `AvailableWantedTimes`.

**Don't exercise non-test code that is not part of the subject**

Depending on implementation outside of the subject under test for the test to work can lead to issues later, because if that implementation changes, the tests will break for incorrect reasons. 

SG Example: see commit `7b027da3327886fef869b73aa36caa4bab4b2d56` for the state I'm talking about
`available_wanted_times_spec.rb` calls `Restaurant#available_wanted_times` in order to set up expectations (among other things, but let's focus on this one example). See below (with added comments):
```ruby
it 'shows WANTED_TIME as available time' do
    parent_restaurant.update(throttle_default_size: throttle_default_size + 1)
    # !! Setting up expectation via the behavior of a class outside of the class under test
    timeslots = parent_order.restaurant.available_wanted_times(
      parent_order.first_possible_wanted_time(parent_throttle_settings.lead_minutes)
    )
    available_slots = Gravy::Throttle::AvailableWantedTimes
                        .new(parent_order, parent_throttle_settings).call

    # !! If the implementation of Restaurant#available_wanted_times changes,
    #    this expectation could (is likely to) fail for an obscure reason
    expect(available_slots.count).to eq(timeslots.count)
    expect(available_slots.first[:original].strftime('%I:%M%p'))
      .to eq '01:30PM'
  end
```
The correct approach here would be to make sure there are adequate tests around `Restaurant#available_wanted_times`. Then expect on the call to `Restaurant#available_wanted_times`, and return a known quantity. An improvement on this specific test would look like this:

```ruby
it 'shows WANTED_TIME as available time' do
  parent_restaurant.update(throttle_default_size: throttle_default_size + 1)

  # Setting up the timeslot array as a known value
  Time.zone = 'Eastern Time (US & Canada)'
  timeslots = [
    {
      original: Time.zone.parse('2020-04-01 13:30:00'),
      formatted: 'Wednesday, 1:30PM',
      max_slots: 3
    }
  ]

  # Expect on the call, and return the known value
  # Ideally there would be a `.with()` between `.expects()` and `.returns()`
  # so we can verify that given an output from the previous dependency,
  # this dependency is called with that output
  parent_order.restaurant.expects(:available_wanted_times)
    .returns(timeslots)

  available_slots = Gravy::Throttle::AvailableWantedTimes
                      .new(parent_order, parent_throttle_settings).call

  # Expect based on the known value above
  expect(available_slots.count).to eq(1)
  expect(available_slots.first[:original].strftime('%I:%M%p'))
    .to eq '01:30PM'
end
```

------
### Conclusion

This is meant as a helpful guide, a dump of thoughts and experiences, and is not meant to be prescriptive, definitive, or final, etc. You know the drill.

------

##### *-- adrienne ([@ahisbrook](https://github.com/ahisbrook), [ThoughtWorks](https://www.thoughtworks.com/profiles/adrienne-hisbrook))*