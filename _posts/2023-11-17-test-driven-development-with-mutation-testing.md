---
layout: post
title: "Test-driven development with mutation testing (TDD+M)"
description: "Beyond conventional TDD: exploring mutation techniques"
date: 2023-11-15 17:15:00 +0200
background: '/img/posts/post05.jpg'
image: 'https://ibb.co/x3zTBY5'
---

# Beyond conventional TDD: exploring mutation techniques

Hey everyone! Today, I want to delve into a fascinating area of software testing,
specifically incorporating mutation testing into the traditional Test-Driven Development (TDD) process.

The results of TDD+M (Test-Driven Development + Mutation) experiments are intriguing and, 
I believe, are set to become a standard part of future software development processes.

As mentioned earlier, this post is based on the article *Test-driven development with mutation testing – an experimental study* by Prof. Adam Roman and Dr. Michał Mnich 
from Jagiellonian University in Kraków, Poland.
&nbsp;
## Table of Contents

1. [Test-driven development (TDD)](#tdd)
2. [Test-driven development with mutation testing (TDD+M)](#tdd+m)
    1. [Concept](#concept)
    2. [Experiments](#experiments)
    3. [Results](#results)
3. [Conclusion](#conclusion)

&nbsp;

Based on:
```
Roman, A., Mnich, M. Test-driven development with mutation testing – an experimental study. 
Software Qual J 29, 1–38 (2021).
```

Full text available at: [https://doi.org/10.1007/s11219-020-09534-x](https://doi.org/10.1007/s11219-020-09534-x)
&nbsp;
## Test-driven development (TDD) <a name="tdd"></a>

TDD is a well-known software development technique where tests are written before the actual code, 
and the code is then developed to pass these tests.

Check out this image below that illustrates the TDD process beautifully:

<a href="https://ibb.co/DrrRBcB">
    <img src="https://i.ibb.co/hYYmGQG/p1.png" alt="tdd">
</a>
<p style="font-size: 11px; font-style: italic;">Source: Roman, A., Mnich, M. Test-driven development with mutation testing – an experimental study</p>


The process involves creating a test, writing code to pass the test, and optionally refactoring. 
This cycle continues until all required functionality is implemented.
Simple idea, right? 

However, implementing TDD in practice can be challenging. 
A 2020 survey found that 41% of developers reported their organizations have fully adopted TDD, 
highlighting its benefits like improved code quality, early bug detection, and enhanced documentation, 
but also acknowledging its drawbacks such as the initial time investment and maintenance of test suites.


One significant issue is the quality of unit tests. 
Writing tests before coding isn’t easy, and it's common to miss covering all business cases and software defects. 
This is where mutation testing comes into play.

&nbsp;
## Test-driven development with mutation testing (TDD+M) <a name="tdd+m"></a>

For more on mutation testing, check my previous post: [Introduction to mutation testing](https://wszlosek.github.io/DevDawn/2023/07/15/introduction-to-mutation-testing-scala.html).

If you're new to this concept, I highly recommend reading it.

&nbsp;
#### Concept

TDD+M merges TDD and mutation testing. It adds a step to the TDD process to check the quality of our tests, improving them with additional test cases as needed.
This enhanced process is depicted in the image below:

<a href="https://ibb.co/V9nB5SP"><img src="https://i.ibb.co/xL95djR/Zrzut-ekranu-2023-11-12-o-15-16-19.png" alt="tdd+m" border="0"></a>
<p style="font-size: 11px; font-style: italic;">Source: Roman, A., Mnich, M. Test-driven development with mutation testing – an experimental study</p>

Nowadays, there are many tools that can help us with mutation testing, so it is not a problem to implement this updated process in practice.
PITest, which about you can read in my previous post, is one of them.

&nbsp;
#### Experiments

The authors conducted an experiment to evaluate the effectiveness of TDD+M compared to traditional TDD. 
They focused on how mutations influence the TDD methodology. The experiment involved a student project on matrix operations, 
with teams divided into TDD and TDD+M groups.

For a detailed account of the experiments, refer to the linked study.

&nbsp;
#### Results

The authors posed the following questions (quoted from the paper):
* *RQ1. Do the tests written with the TDD+M approach give better code coverage than the ones written in a pure TDD approach with no mutation process involved?*
* *RQ2. Are the tests written with the TDD+M approach stronger (more effective) than the ones written using a pure TDD approach?*
* *RQ3. Is the external code quality better when the TDD+M is used than in case of using the TDD approach only?*
&nbsp;
##### Results for RQ1:
The TDD+M group achieved 49.3% code coverage, compared to 31.1% in the TDD group, indicating significantly better code coverage with mutation testing.

#### Response to RQ2:
The TDD+M group achieved an average of 63.3% mutation coverage, while the TDD group reached only 39.4%. 
This significant difference suggests that TDD+M is more effective in detecting errors than traditional TDD.

#### Results regarding RQ3:
In the code quality study, the TDD+M group detected an average of 10 defects in the TDD code,
whereas the TDD group found only about 1.75 defects in TDD+M code. Despite similar complexity and size of the projects,
TDD+M tests were more efficient, detecting an average of 12.62 defects per KLOC (thousand lines of code), 
compared to 2.33 defects per KLOC by TDD tests. These findings affirm that code developed with TDD+M is of higher quality than that produced by pure TDD.

&nbsp;
## Conclusion

The experiment demonstrated that the TDD+M approach (Test-Driven Development with mutation testing) is more effective than traditional TDD.
This effectiveness is seen in writing more effective tests with higher code and mutation coverage, and producing higher quality code.
TDD+M tests achieved 17% better statement coverage and 23% better mutation coverage than TDD tests.

Adding mutation testing to the iterative test-driven development process boosts confidence in code quality and can be successfully
applied in various software development models.


I believe that TDD+M represents the future of software development, and the experiment results clearly highlight its advantages over classic Test-Driven Development.
&nbsp;
##### Epilogue

Remember, this post only scratches the surface of the research and findings related to TDD+M. 
For a comprehensive understanding, including detailed experimental descriptions, questions, and responses, 
I highly recommend reviewing the article *Test-driven development with mutation testing – an experimental study*.


It's a concise yet informative read, and I encourage you to check it out for more in-depth details.
<br/><br/>

<script src="https://utteranc.es/client.js"
        repo="wszlosek/DevDawn"
        issue-term="title"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>
