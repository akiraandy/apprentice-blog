---
layout: post
title: "The Clean Coder"
description: "PERT is so cool!"
categories: [books]
tags: [PERT, Clean Coder, Uncle Bob]
redirect_from:
  - /2018/02/23
---
This week I read **The Clean Coder** by Robert "Uncle Bob" Martin. It read like a programmer's manifesto and I thought he had some interesting stories and opinions about software and software craftsmanship. One of the things about becoming a crafter that makes me nervous and that I'm very curious about is time estimations of projects. How does a team know how long a feature will take to make it to production? What **is** complexity?

Uncle Bob provided one very interesting statistical method of determining how long a feature would take a team. The PERT (Program Evaluation and Review Technique) was created to support the U.S. navy's Polaris submarine project. To estimate a task you provide three numbers known as the **trivariate analysis**:
  * O: Optimistic Estimate: This number is **VERY** optimistic. This number is reached if NOTHING goes wrong and EVERYTHING goes perfectly
  * N: Nominal Estimate: This is the estimate with the greatest chance of success.
  * P: Pessimistic Estimate: This is the opposite of 'O', it is a wildly pessimistic estimate that assumes that nearly everything goes wrong

The probability distribution of the 3 estimates goes as follows:

$$\mu=\frac{O+4N+P}{6}$$

$$\mu$$ is the expected duration of the task.

To find the [standard deviation](https://simple.wikipedia.org/wiki/Standard_deviation) of the probability distribution we calculate the following:

$$\sigma=\frac{P-O}{6}$$

When this number is large, the uncertainty is large, when it is small (closer to 0) the uncertainty is low.

What if we wanted to know the probability distribution of multiple tasks? We have a calculation for that too!

$$\sigma \tiny sequence \normalsize = \sum \mu \tiny task$$

This means that the probability distribution of any sequence of tasks is the sum of each task's probability distribution.

Then to find the standard deviation of the sequence of tasks we find the square root of the sum of the squares of the standard deviations of the tasks.

$$\sigma \tiny sequence \normalsize = \sqrt{\sum\sigma_{task}}^{2}$$

This is all very theoretical but let's try to use the example used in the book to try to understand exactly what's going on.

| Task | Optimistic | Nominal | Pessimistic | $$\mu$$ | $$\sigma$$
| ------:| ------:| -------:| -------:| -------:| -------:|
| Alpha | 1 | 3 | 12 | 4.2 | 1.8 |
| Beta | 1 | 1.5 | 14 | 3.5 |2.2 |
| Gamma | 3 | 6.25 | 11 | 6.5 | 1.3 |

The numbers under optimistic, nominal and pessimistic represent days these tasks might take. $$\mu$$ represents the expected duration of the task and $$\sigma$$ represents the standard deviation of the probability distribution of the task.

We can get $$\mu$$ and $$\sigma$$ for each task by applying the previous calculations, let's take Alpha for example.

(1 + (4*3) + 12) / 6 = 4.2

4.2 equals our expected duration of this task represented by $$\mu$$.

Then to find our standard deviation we do the following calculation for Alpha.

(12 - 1) / 6 = 1.8

Our standard deviation ($$\sigma$$) here is 1.8.

Now that we've broken a task down to it's probable duration and standard deviation, let's calculate how long all the tasks might take. For this we sum up the probable duration of each task: (4.2 + 3.5 + 6.5) = 14.2 so a little more than 14 days.

Then to find the standard deviation of the sequence we square-root the sum of the squares of the standard deviations of the tasks.

(1.8^2 + 2.2^2 + 1.3^2)^.5 = (3.24 + 2.48 + 1.69)^.5 = 9.77^.5 =~ 3.13

So the two important numbers we have here are 14.2 (expected duration of all tasks) and 3.13 (standard deviation). Using these numbers we can tell that the tasks will LIKELY take AT LEAST 14 days but very well could take 17 days (1 $$\sigma$$) or even 20 days (2 $$\sigma$$).

I found this little foray into stats and probability distribution fascinating and I thought the PERT method is a really interesting way of determining estimations for software development. I'm curious to see how 8th figures out estimations and if they use a mathematical model.
