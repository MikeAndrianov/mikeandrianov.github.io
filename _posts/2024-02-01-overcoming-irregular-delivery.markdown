---
layout: post
title:  Overcoming irregular delivery
date:   2024-02-01 02:45:00 +0200
categories: mentorship
excerpt: "What are the main problems, and root causes of non-regular delivery, and how do start delivering consistently as a software engineer? You can find answers in this article."
---

Lately, my mentee and I have been focused on enhancing impact within the team. One goal we set was to improve reliability and visibility by starting to deliver regularly. 
Ideally, we would be delivering at least one commit daily even though it doesn't necessarily mean to close a new ticket each day.

To assist my mentee in overcoming potential obstacles that might stop him from accomplishing the goal, I have created a diagram, which I decided to share with the wider audience as I believe it can be useful for more engineers.

I broke the abstract "can't commit daily" question into the 3 main sections:

1. **Problems** \
What prevents engineers from regular delivery. Often, these are the responses you'd receive when inquiring about the delay in completing a feature.

2. **Root causes** \
Identifying the root causes that give rise to issues in the delivery process.

3. **Solutions** \
Offering practical solutions for addressing each of the identified root causes. These serve as recipes for overcoming the challenges associated with regular delivery.

[![Irregular delivery diagram](/assets/img/irregular-delivery-diagram.png)](/assets/img/irregular-delivery-diagram.png)


## Problem 1. Ticket's complexity

Addressing this challenge can indeed be tricky, and it requires a thorough identification of the root causes.
Let's go through the most popular root causes one by one.

### 1. **Unclear requirements**
Engineer dove into feature development without a comprehensive understanding of the outcome, resulting in numerous iterations or, in extreme cases, 
the need for complete rebuilding of the delivered work. 
Ideally, initiating work should only commence when the final goal is crystal clear, well-understood, and logically sound. 
Depending on the feature, it may be beneficial to also include such things as the acceptance criteria, development breakdown, link to the design, etc.

**Solution:** don't be shy and seek input from relevant colleagues, including product managers, designers, other developers, and stakeholders. Don't hesitate to request additional details until the requirements are thoroughly clarified.
Proactively seek input from relevant colleagues, including product managers, designers, fellow developers, and stakeholders. Don't hesitate to request additional details until the requirements are thoroughly clarified. 
Open communication ensures a comprehensive understanding of the project, reducing the likelihood of ambiguity and facilitating smoother development processes.

### 2. **Ticket is too big**
In some cases, a ticket may be well-defined from a business perspective, yet engineers working on it might encounter challenges with the delivery. 
Some people may develop locally and only commit changes when they finish the ticket. 
Additionally, based on my observations, the review process tends to slow down as the size of the corresponding pull request increases. 
This, in turn, can lead to a prolonged period of stagnation in the review phase.

**Solution:** break down large tickets into smaller, more granular tasks, which are easier to understand, implement, review, and test.

### 3. **Ticket is too difficult**
Even if a ticket is exceptionally well-defined, there are instances where it's too challenging for the assigned individual to tackle it alone.

**Solution:** discuss with colleagues implementation details and set milestones. When necessary ask for the review while the pool request is still a draft.

## Problem 2. Stuck or blocked
Working in an environment of uncertainty is common, with potential issues like buggy libraries, missing documentation, and complex codebases. 
Recognizing the right time to seek help is crucial to avoid unnecessary time wastage. 
Please keep in mind, that before asking you did everything you could to resolve the problem by your own: 
from googling and usage of AI tools to reading library and internal documentation. Asking for help should be the next step after giving your best shot at solving the problem independently.

**Solution:** ask for help.

There are a couple of tricks to keep in mind, and how to do it properly to receive necessary help faster and effectively:

1. **Ask questions in public channels instead of pinging people directly** \
It increases the probability of resolving problems faster and also increases visibility. 
That means a wider audience will know about your current status. This is especially important when working remotely.

2. **Specify as much information as possible** \
Don't be afraid to overcommunicate. It's always better to provide more information, rather than less. Please keep in mind, that other people need to understand the context and the problem quickly.
Not everyone is eager to play ping-pong with additional questions and answers. So try to prevent it.
Write an extensive description of what you have done so far, your conclusions, or suspicions. Share links to the ticket, code, logs, etc.

3. **Be proactive** \
After all you are responsible for resolving your issues. If nobody helps, don’t reassure yourself that you asked and can rest until someone finally help you. 
Ping your team members, ask question in different channels, send follow-ups, do everything you can to facilitate resolving the issue.

\
Do you struggle with something else and don't know how to deal with it? [Send me an email](mailto:mikeandrianov@gmail.com). Perhaps I could help. It's always worth asking.
