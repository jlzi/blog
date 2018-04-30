---
layout: post
title:  "Happy Birthday, Mr. Johnson!"
date:   2018-04-30 11:00:00 +0300
image: /assets/old-geezer.png
categories: development testing
comments: true
---

This happened long time ago when I was supporting production deployment of a big enterprise system of a bank.

The deployment was not automated. Special people with production access were sitting abroad and doing necessary changes. I, as a developer, was telling them via Skype what to do. The people were not familiar with the system. They were not even developers. Due to such an approach the deployment usually took very long time. It was already late evening when this happened...

<img alt="Old Geezer" style="margin: 0 auto; display: block;" src="{{ site.url }}{{ page.image }}">

Suddenly, a Jenkins job that was deploying a component became red. That was an unpleasant surprise as the component was the last one in the long list to deploy. I opened the log and saw that the unit tests failed. They expected 60 € as a price of a health insurance, but the actual price the component calculated was 80 €. That was really weird: nobody touched the code for a month and all the previous builds were green.

Frankly, the first idea that came to my mind was to ignore unit tests and continue deployment. After all, that's how developers usually deal with failing unit tests, don't they? However, given a criticality of this component, that was not a good idea. It was responsible for calculation of prices. People can forgive broken UI layout. But they never forgive any issues with their money. So I had only one choice: to fix the tests as soon as possible and finish the deployment. I started digging into the error.

"OK", I thought, "yesterday everything was fine. Today the tests fail. This means that either the code still changed (though it was not supposed to). Or the service was taking some data, like, for example, pricing coefficients, from the outside (that can be either another service or a database) and the unit tests were written in a wrong way, i.e., they relied on the unstable data from the outer world."

I started with the first option. I checked the commits history, the history of the job itself, nothing changed since last month. Then I took the green build and the build that failed and compared their checksums. They were identical. This meant that nothing changed, definitely.

Then I switched to the second option. It was a bit trickier as the service itself was the legacy, with thousands of lines of spaghetti code per class. I was looking for all signs of that the service was consuming something from the outer world. JDBC connection strings, JNDI bindings, REST clients, SFTP to external file system and so on. Nothing, except for one small RESTful service that provided coefficients for the prices (probably, to make it possible to change them without changing the code). However, that service was properly mocked in the unit tests, i.e., the tests were consuming its static fake twin, not the service itself.

The only possibility that left was start digging into the price calculation business logic. That was the scary place that was written long time ago and nobody really wanted to touch it. After some time of futile attempts and several questions like "How much longer should we wait?" from my abroad deployers, my attention was attracted by the following line:

```java
int clientAge = DateUtil.countAgeUsingPersonalCode("280752-7918")
```

The insurance was more expensive than the test expected, right? The insurances usually become more expensive when the clients get older. Wait a second...

Today is July, 29th. Mr. Johnson (that was the name of the fictional client from the unit tests) celebrated his sixtieth birthday yesterday. And sixty years is one of the frontiers when the price for the insurance gets higher!

Now everything was completely clear to me. The code didn't change indeed. And the unit tests didn't consume any services from the outer world: the only outer service was properly mocked. However, one external thing the tests still "consumed" was the time. Yesterday, and all the time before, the tests were passing because Mr. Johnson was still "young" and was eligible for cheaper insurance. Starting from today, however, he turned sixty, the price for him got higher, while nobody adjusted the tests and they still expected the price to stay the same...

That deployment took a little longer than expected. Still it was successfully finished. I also created a ticket to make the `DateUtil` independent on current moment of time and re-write the unit tests. And, the most important, I have learned a good lesson.

### The Lesson Learned

Never allow your unit tests depend on _anything_ external. Time is also the external thing, though it may seem stable and predictable.

One good way to validate your unit tests can be changing your PC clock to some very strange time, like to a distant past or future, and check that the tests keep passing without raising false-positive errors.