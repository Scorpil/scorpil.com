+++
title = "The Problem of Overfitting in Tech Hiring"
description = "and how to fix it"
author = "Scorpil"
date = 2020-10-08T08:35:52+02:00
tags = [
  "rant",
  "hiring",
  "management",
]
images = ["/img/xkcd_1293.png"]
+++

When a tech company needs a skilled developer to work on a complex task, they naturally tend to look for somebody who has done the same thing before. Sound and obvious logic. It was applied long before the tech sector even emerged. If you were to hire, say, an auto mechanic, you'd do something similar. However, tech recruiting is unique in one peculiar factor: the level of specificity in job ads and screening interviews is insane.

### Job ads

For the sake of the experiment, I searched for auto mechanic jobs around London. In the vast majority of job ads, I found **a single sentence** describing prerequisites for a job. Something like this:

> Only vehicle technicians/mechanics with appropriate hands-on experience and qualifications in the motor industry should apply.

No "experience in rebuilding the carburetor" requirement. None of "repaired 2018 Toyota Corollas for the last 7 years" nonsense. An ad describes a type of job the repair shop performs and then lets the applicant decide whether they have "appropriate hands-on experience". Some ads might ask for total years of job experience, even then often adding a "preferred" remark.

Compare it with a _typical_ job description for a senior developer (not a real one, since I don't want to pick on any company in particular):

> - X+ years of full-time software development experience
> - Knowledge of software development fundamentals (data structures, algorithms, etc.)
> - Comprehensive knowledge of modern Javascript (ES6+: Modules, classes, arrow functions, destructuring, async/await, etc.)
> - Ability to select the right tools/libraries for the job
> - X+ years of experience with VueJS
> - X+ years of experience with PostgreSQL
> - Solid understanding of good practices, design patterns, and writing idiomatic Javascript code
> - Accounting for performance implications and scalability of code
> - Desire to write meaningful tests and maintaining thorough test coverage
> - Ability to maintain large, complex codebases

Phew. If I wouldn't know how those ads are made, I would think somebody gets paid per character here. Half of the bullet points are useless: knowing best practices, maintaining test coverage, and the ability to work on large codebases are be covered by "senior" in the title. Other requirements are restrictively specific. Would a senior software engineer with React experience be puzzled by the Vue codebase? Not for more than an hour. Sure, they might some personal preferences in that regard, but to address those there's another section like "About Us" or "What we want from you".

There are very few quantitative metrics in the developer's profile, so companies just can't avoid putting "years of experience" everywhere. It's a simple boolean criterion that can be applied without thought. I find it is on average a quite poor predictor of the applicant's performance for _your_ environment and tasks. It's a value to be considered, but it has much too little signal to apply it on its own, as a filter.

In a particular case when you don't want to "train" fresh-grads, you _could_ put "1+ years full-time employment as..." requirement, but you could also just ask for "experience writing code for production systems" without setting an imaginary cutoff date. Experience is non-linear, and to compress into one number you need to leave out so many details that the result doesn't carry much weight.

![XKCD comic #1293: Job Interview](/img/xkcd_1293.png)
{{% subscript "Who needs unsplash where there's [XKCD](https://xkcd.com) on every topic?" %}}

### Interviews

Moving on to the interview, the same two extremes are often seen.

On one side are silly logical puzzles. Is there a correlation between the ability to estimate the number of piano tuners in NYC and the programmer's performance? I don't know. I doubt anyone knows for sure. Even empirically it seems like a stretch. Google, commonly known as a trend-setter for these types of interviews, stopped using logical puzzles precisely because they didn't found any evidence of them being effective. Those questions are supposed to test _generic_ critical thinking and, even if they do, that doesn't necessarily translate to the specific types of problem-solving software developers engage in.

On the other extreme, a list of questions about precisely the tools the company is using. Just like an exam. But exams exist to test student's **knowledge** on a particular topic, not to evaluate his **experience** in a field as vast as software development. The end result is similar to [overfitting](https://en.wikipedia.org/wiki/Overfitting) in machine learning, where the model represents training data a little too well, losing an ability to generalize and predict new data points. Filling the company with candidates that all have the exact same skillset and background means they will come up with similar solutions and make identical mistakes. Instead of building a synergetic team, it creates a hive-mind.

The importance of experience in a narrowly specialized field (on a level of particular libraries, frameworks, even languages) is overrated. When a developer joins a new project, there will be a warm-up period when they get used to the internal conventions, product and business specifics, and learn to work with existing codebase (often poorly documented). The overhead of learning a well-documented, state-of-the-art open-source library or framework is a drop in an ocean.

### But can we do better?

The most productive technical interview is a two-way experience exchange. The interviewee gets to learn about the challenges that the company faces and an interviewer gets to assess the candidate based on his strong sides. Whether you give a candidate a whiteboard task, coding challenge, or simply ask them a question, an answer is not supposed to be a means to an end, but rather a topic for conversation. Subjective and open-ended questions which are rooted in a subject area, but are not tied-in to a particular tool, give the interviewer much more insight into the developer's experience than "how would you join these three tables?". If it turns out this particular candidate is not a match, at least both sides have likely learned something new. Less time wasted.

Another important benefit of the conversational interviewing style is that it creates a much more comfortable psychological environment for the candidate by adjusting the power dynamic. Nobody likes being assessed. If an interviewer sets himself as an equal, instead of getting into a „boss“ persona, the whole meeting will be less stressful. The candidate is more likely to behave naturally and that’s how you want to see them because that’s how they will behave on a job. Unless developers in your company are constantly stressed, in which case you’re doing it wrong.

This interview style will probably never become very popular in large enterprises. It’s too un-formalized, doesn’t scale well, and relies entirely too much on the interviewer‘s professionalism. Yet, there are lots of startups with a strong company culture that care enough about tech hiring to re-think the process.
