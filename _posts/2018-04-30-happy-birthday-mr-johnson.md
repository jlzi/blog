---
layout: post
title:  "Happy Birthday, Mr. Johnson!"
date:   2018-04-30 11:00:00 +0300
image: /assets/old-geezer.png
categories: development testing
comments: true
---

This happened a long time ago, when I was supporting production deployment of the large enterprise system of a bank.

The deployment was not automated. IT support guys with production access were working abroad and making necessary changes. I, as a developer, was telling them what to do via Skype. These people were not familiar with the system. They were not even developers. Because of this, the deployment usually took a very long time. It was already late evening when this happened...

<img alt="Old Geezer" style="margin: 0 auto; display: block;" src="{{ site.url }}{{ page.image }}">

Suddenly, a [Jenkins](https://jenkins.io) job that was deploying a component turned red. That was an unpleasant surprise, as the component was the last one in the long list to deploy. I opened the log and saw that the unit tests had failed. The tests had expected 60 € as the price of a health insurance policy, but the actual price the component calculated was 80 €. That was really weird: nobody had touched the code for a month and all the previous builds were green.

Frankly, the first idea that came to my mind was to ignore the unit tests and continue deployment. After all, that's how developers usually deal with failing unit tests, isn't it?

However, that would be too risky this time. People can forgive broken UI layout. But they never forgive any issues with their money.

So I had only one choice: to fix the tests as soon as possible and finish the deployment. I started digging into the error.

"OK", I thought, "yesterday, everything was fine. Today, the tests failed. This means that either the code changed (though it was not supposed to), or the service was taking in some external data (for example, pricing coefficients) from outside sources, such as a database, resulting in faulty unit tests."

I started with the first option. I checked the commits history, the history of the job itself. Nothing had changed since last month. Then I went further, and took the green build and the build that failed and compared their [checksums](https://en.wikipedia.org/wiki/Sha1sum). They were identical. This meant definitively that nothing had changed in the code.

Then I switched to the second option. It was a bit trickier as the service itself was the legacy, with thousands of lines of spaghetti code per class. I was looking for any and all signs that the service was consuming something from the outer world. JDBC connection strings, JNDI bindings, REST clients, SFTP to external file system and so on. Nothing, except for one small [RESTful service](https://en.wikipedia.org/wiki/Representational_state_transfer) that provided coefficients for the prices. However, that service was properly [mocked](https://en.wikipedia.org/wiki/Mock_object) in the unit tests, i.e., the tests were consuming its static fake twin, not the service itself.

The only route left to me was to start digging into the price calculation business logic. That was a daunting task. The logic was written a long time ago and nobody really wanted to touch it. After some time of futile attempts and several questions like, "How much longer should we wait?" from my deployers overseas, my attention was attracted by the following line:

```java
int clientAge = DateUtil.countAgeUsingPersonalCode("280752-7918")
```

The insurance policy was more expensive than the test expected, right? Insurance usually becomes more expensive as the clients get older. Wait a second...

Today is July 29th. Mr. Johnson (that was the name of the fictional client from the unit tests) celebrated his sixtieth birthday yesterday. And sixty years is one of the frontiers when the price for the insurance gets higher!

Now everything was completely clear to me. The code didn't change, and the unit tests didn't consume any services from the outer world; the only outer service was properly mocked. However, one external thing the tests still "consumed" was the time. Yesterday, and all the time before, the tests were passing because Mr. Johnson was still "young" and was eligible for cheaper insurance. Starting from today, however, he turned sixty, the price for his policy increased, while nobody adjusted the tests and they still expected the price to stay the same...

That deployment took a little longer than expected. Still, it was successfully finished. I also created a ticket to make the `DateUtil` independent on current moment of time and re-write the unit tests. And, most importantly, I learned a good lesson.

### The Lesson Learned

To make sure that no "Mr. Johnsons" spoil your evening again, never allow your unit tests to depend on _anything_ external. Time is also an external value, though it may seem stable and predictable.

One good way to validate your unit tests can be changing your PC clock to something out-of-date, like to a distant past or future, and check that the tests keep passing without raising false-positive errors.
