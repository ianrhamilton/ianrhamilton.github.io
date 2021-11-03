---
title: "Common pitfalls of using cucumber within an Agile Team"
categories:
  - Blog
tags:
  - Cucumber
  - BDD
---

## Introduction
[Cucumber](https://cucumber.io/) when used efficiently can be a very powerful tool to support agile development teams. The problem is, sadly, if you’re using cucumber there’s a strong likelihood your not doing ‘BDD’ (Behavioural Driven Development) and you are missing out on cucumber’s full potential.<br>

## The Problem
I recently introduced cucumber as a method of communication and collaboration to a large organisation with many Scrum teams, and I quickly noticed many pitfalls and misconceptions of using cucumber. Cucumber has rapidly become the tool I have a love/hate relationship with – simply because some people just didn’t understand it, or even try to? Some teams got it, most teams didn't.<br>

Your team is Agile, right? Each sprint you have a goal, everyone knows their role, BA’s write the user stories, developers write the code, and the tester at the final stage in the flow writes the automated checks using cucumber. Sound familiar? <br>

If this is the case you are not using cucumber to its full potential and you are certainly not doing BDD. Your team is most probably getting hung up cucumber being all about the automated checks. **Cucumber is not an automated test tool** it is a **collaboration tool**. [@FriendlyTester](https://twitter.com/FriendlyTester?ref_src=twsrc%5Etfw) sums up Testers using BDD tools to write automated checks up very well [here](https://thefriendlytester.co.uk/search/label/BDD).<br>

Following Behavioural Driven Development is not easy and it requires commitment and discipline from the **WHOLE** team.

## The Solution

<p align="center">
  <img width="600" height="200" src="/assets/images/put_the_cukes_down.jpeg">
</p>

If you are currently using cucumber, or planning on using it – stop and ask yourself or your team;

> - Why you are using it?
- What are the benefits in changing your way of thinking/working?
- Is it really worth it if the team isn’t going to fully commit to using it?

Take stock and understand the purpose of Cucumber and BDD to support your team’s development process. It is not going to be an instant solution to your problems ***BUT*** when done right you will see **productivity increase** through **collaboration** and a **shared understanding** of features. With that in mind before even beginning with cucumber the first step is to bridge the gap of communication within the team. By team I mean the Business Analyst, Developers, and Testers. How?

<p align="center">
  <img width="600" height="200" src="/assets/images/three-amigos.jpeg">
</p>

Before a user story can be determined as ready for development or estimation it must have been through a "Three amigos session".

- The BA (Business Analyst) working with the Product Owner has the idea of the user story.
- The Developer is there to represent the developers; he/she may use this session to determine whether or not there are any technical challenges behind the user story, and discuss the implementation.
- The Tester is there for their expertise in defining test scenarios.

The communication begins when the Business Analyst describes the motivation behind the story. This is set out in the form of:

> Feature: Title <br>
    As a [Role], <br>
    I want [feature], <br>
    So that [Benefit].<br>

It doesn’t really matter who leads these sessions, however in my experience it is generally the Automated Tester/ developer-in-test, simple because they have the experience in writing feature files.

Features contain a story and then a set of manageable ‘scenarios’ (Acceptance Criteria). Scenarios are written in plain text  using a Domain Specific language called Gherkin. These scenarios contain Steps in the form of Given-When-Then-And-But, for example:

> Given - pre-condition <br>
When - action <br>
Then - outcome/assertion <br>

The following are perfectly fine to use, however I try to use sparingly in features as I feel they get abused (more on this later).

- And – could be used as a second precondition, an action, or in some cases a second outcome/assertion.
- But – could be used as a second precondition, an action, or in some cases a second outcome/assertion.

### Example
We won’t go into too much detail of writing scenarios. This is a very basic example, but lets say a BA has presented the following story:
> Feature: Basic Login <br>
  As a customer,<br>
  I want to log in the site,<br>
  So that I can use its services.<br>

The first topic of conversation will most probably be what the expected outcome of the login page is? What are the preconditions and actions to get to the expected outcome? When writing scenarios you are putting yourself in the end user’s shoes – what would a customer do here? What would they expect to see there? Etc. I like to break scenarios into two categories positive, and negative.

An example of a positive scenario may be *"A previously registered customer can login to the site using their username and password"*, i.e. successful login. This expected outcome would be become scenario title.

```gherkin
Scenario: A previously registered customer can login to the site using their username and password  
    Given a registered customer is on site (precondition)
    When I enter my "username" and "password" (action)
    And click on the submit button (action)
    Then the login attempt should be successful (outcome)
```

This may trigger a conversation with questions on the implementation, for example:
- How does the user submit their details – will there be a link, or a button?
- Will there be a success message?
- Where should the user be navigated to on successful login?
- What are the different login options? Will you have social login options?
- What if a customer is not registered? Where will the register option be located?

An example of a negative scenario may be *“A previously registered customer has incorrect login credentials (the password is incorrect)”*

```gherkin
Scenario: A customer has incorrect login credentials (the password is incorrect)
  Given a registered customer is on site
  When the customer tries to log in with incorrect password
  Then the login attempt shouldn't be successful
   And an error message should be shown 
```

Again this should trigger a conversation with questions on the implementation, for example:
- What type of error will be displayed? And where?
- What is the error message?  Do have translations for it? (if needed)

The benefit of writing the features collaboratively is that the whole team have a common understanding of a user story and anyone not involved in the session can read the feature and understand exactly what is expected in order to estimate, or begin development. <br>

A session will generally flash out most of the highest priority scenarios. However when a story is picked up in development, or when sessions are over the Tester or BA may identify edge cases not covered in the session. The team can then add any remaining scenarios and then update.

### Do's and Dont's when writing Features
### <span style="color:green">*Good example*</span>.
```gherkin
Scenario: Basic personal registration
  Given a potential customer is on the registration page
  When the customer completes the registration form 
  Then a "Registration successful" message should appear
```
### <span style="color:red">*Bad example*</span>.
```gherkin
Scenario: Basic personal registration
  Given a potential customer is on site
  When they click on the "Registration" link
  Then they should be on the "Registration" page
  When I fill in the "first name" field with "Joe" 
   And I fill in the "surname" field with "Blogs"    
   And I select "Mr" from the title dropdown
   And I fill in the "address line one" field with "Test Address line one"
   And I fill in the "address line two" field with "Test Address line two"
   And I fill in the "postcode" field with "NE TE5T"
   And I fill in the "email" field with "joe.blogs@example.com"
  When I click on the "Register" link
  Then "Registration successful" message should appear
```

Which of the above do you think looks more readable? Try to keep scenarios short by hiding implementation details. If your scenarios look like the second scenario above, ***stop it!, Stop it now!***.

### Test first, test together!
Now we have a feature that has been “Three Amigo’d”, it is now ready for development, or ready for estimation by the team.

The next step is to begin automating features using cucumber. This is a point where you come to a crossroad, and you must decide who will be responsible for automating features.

<p align="center">
  <img width="600" height="200" src="/assets/images/process.jpeg">
</p>

1. Developer begins to automate features before a single line of application code has been written.
2. The tester begins to automate features and the developer begins work on the application.
3. Tester and Developer collaborate and automate features before a single line of application code has been written.

I have worked on quite a few projects where the above responsibilities have divided opinion of the team. There are arguments for all three cases and I personally have worked in environments following options 2 & 3.  It usually boils down to whatever suits your current team’s skill set and time.

### Option 1
Can work great if the develop have experience in test automation, and its best practises? The developer can be guided by the specification outlined in the Three Amigos session.

### Option 2
When a tester begins writing checks he/she will want to know how best to interact with the application and that can be answered easily by the developer. The developer will also need to know what he/she can do to make the application as testable as possible – the tester can help here. There is a definite need for collaboration at this stage, which leads me to option 3...

### Option 3
Communication between test and development is essential, there is always going to be questions. Pairing can easily solve this. The tester and developer can automate the feature before a single line of code has been written. Throughout this stage, any implementation details can be discussed.

This method is very useful as the tester can use their experience in writing maintainable automated checks to guide the developer. The developer will then gain an understanding of the checks and the framework, and the tester can gain an understanding of how the feature will be implemented. This way testing becomes a team effort and eventually developers may become self-sufficient when automating features.

If the developer chooses to write unit tests before writing Acceptance Tests (most still do when doing BDD) there is also an added advantage in that any unit tests they write can influence what is/isn’t written in Acceptance tests because both the Dev and QA will fully understand the coverage of both.

You’re probably starting to wonder where that leaves the role of the Tester, right? Again, ***BDD is not about testing, and certainly does not replace the need for Testers***.

Testers can implement a testing strategy and generally set the standards for testing. Build a framework from scratch, making it as easy to understand and maintainable as possible.

Think about cross browser/OS/device testing, continuous integration, visual diff testing, and load/performance testing. Don’t get hung up on developers doing ‘your job’. Don’t forget that automated checks ***DO NOT find bugs***, they ***prevent them happening further down the line*** by ***planning*** and ***testing early***.

## Summary
Some teams may chose to use cucumber and tools such as webdriver and calabash as a means of writing automated checks that are written in plain English – this is fine if you see benefit in that, but it is not BDD.

Hopefully this very basic introduction is enough for you or your team to understand the true BDD process and prompt you to think about the importance of communication.

**Remember** that **cucumber is not a testing tool**, it is a **colloboration tool**, and If you can get the communication part right, you are well on your way to doing ‘BDD’.
