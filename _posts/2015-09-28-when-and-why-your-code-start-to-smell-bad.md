---
title: When and Why Your Code Start to Smell Bad
layout: post
---

I enjoy reading papers and try to on a regular basis. I actually read them old school style. I print them and then use a pen to underline the quoteables that I liked. Sometimes I annotate them with thoughts, and rarely, I have even put a grade at the top, just for my own recollection.

This weekend I read "[When and Why Your Code Start to Smell Bad](https://dibt.unimol.it/fpalomba/documents/C4.pdf)". Overall, I enjoyed the paper. The first half was mostly explaining what they did and how they did it, which is less interesting to me, but when they hit the part about why and when, things got interesting.

## How

The authors of the paper analyzed over 500 million commits from 200 open source projects, including Android, Eclipse and several Apache projects. There, that saves you having to read the first several pages of the paper. Your welcome.

## When

> ...as well as the side effects of code smells, such as increase in change- and fault-proneness, or decrease of software understandability and maintainability.

**To me, the biggest issues with code smell are fault-proneness and understandability**. One of my primary goals in programming is understandability. It's worthing noting that I am far from perfect in that area, but when a project comes together and feels easily understood, boy does it feel great.

Too often programmers "get things working" and move on. Trust me, **it is always worth a second or even third pass to try to make the code more readable and easy to understand**. I also think that understandability leads to better performing code and obviously more maintainable code.

<hr />

> most of the smell instances (at least half of them) are introduced when a code entity is added to the versioning system.

This is surprising! Common wisdom suggests that smells are added as a result of maintenance, but they found that most were present when the code was first introduced. Knowing this, the action item for programmers when reviewing is to look hard and long at new code entities. Make sure that prior to merging that pull request, there are no smells, because half of smells are introduced that way.

<hr />

> when a smell is going to appear, its operational indicators occur very fast, and not gradually.

In addition to smells sneaking in with new code, the code that contains those smells behaves differently over time than clean code. In fact, there are...

<hr />

> strong differences in the metrics slope between clean and smelly files, indicating that it could be possible to create recommenders warning developers when the changes performed on a specific code component show a dangerous trend that could lead to the introduction of a bad smell.

This sounds pretty awesome. Right when you commit (or try to), the system could warn you and say, hey, this code you introduced might have a smell, in particular this smell, perhaps you should think twice about what you are doing.

<hr />

> most of the smell instances are introduced when files are created. However, there are also cases, especially for Blob and Complex Class, where the smells manifest themselves after several changes performed.

Blob is effectively referring to big classes. This doesn't surprise me at all. **The larger and more complex your class is, the more smells will accrue over time.** Complexity always leads to smells. The issue with large classes is mostly that large classes become dumping grounds, which increases their size and complexity.

## Why

> Smells are generally introduced by developers when enhancing existing features or implementing new ones. As expected, smells are generally introduced in the last month for issuing a deadline... [...] Finally, developers that introduce smells are generally the owners of the file and they are more prone to introducing smells when they have high workloads.

Shocker, right? When you are adding new functionality, sometimes things get messy. **They especially get messy when you are under a deadline or have a high workload**.

## The Lessons

In conclusion, they provide the following four lessons.

### Lesson 1

> Most of the times code artifacts are affected by bad smells since their creation.

### Lesson 2

> Code artifacts becoming smelly as a consequence of maintenance and evolution activities are characterized by peculiar metrics' trends, different from those of clean artifacts.

### Lesson 3

> While implementing new features and enhancing existing ones are, as expected, the main activities during which developers tend to introduce smells, we found almost 500 cases in which refactoring operations introduced smells.

### Lesson 4

> Newcomers are not necessarily responsible for introducing bad smells, while developers with high workloads and release pressure are more prone to introducing smell instances.

**Code is most often introduced as smelly. Smelly code is different than clean code (rots at a dramatically increased rate) and linked to high workloads and release pressure.**
