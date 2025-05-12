---
layout: post
title: The Hardest Problems Aren’t Technical
cover-img: /assets/img/posts/team-fist-bump.jpg
thumbnail-img: /assets/img/posts/team-fist-bump-sq.jpg
share-img: /assets/img/posts/team-fist-bump.jpg
subtitle: Or how I used technology to solve a people problem.
---

Throughout my career, I’ve solved all kinds of technical challenges—migrating data across systems, scaling infrastructure, debugging impossible issues at 2am and while none of those are easy, they tend to follow patterns. There’s usually a right tool, a best practice, a known tradeoff.
One hard truth that I have learned to appreciate over my 30 years of experience is:
>__The toughest problems aren’t technical at all, they are people and process.__

--- 

## The Setup: A Complicated Situation at the VA

Back when I was the tech lead for the core API team at the Department of Veterans Affairs, I found myself in the middle of a situation that wasn’t about code—at least not at first.
Our team was responsible for the backend systems and APIs that powered vets.gov, the platform helping veterans apply for benefits and services. We were building a solution using Ruby on Rails, and there were three different engineering teams working on different parts of the system.
Each of those teams, independently, had done exactly what they were asked to do, solve the problem in front of them to support veterans.  So they picked tools, made architectural choices, and started building.  This created some working code, some solutions heading towards production but also some real issues, and the problem was they weren’t asked to coordinate and align their work.
So we ended up with:

- Different versions of Rails across teams
- Different gems solving the similar problems
- Overlapping logic implemented in conflicting ways


This inefficiency and inconsistency of approach meant that their systems couldn’t be deployed together without complex, brittle workarounds. We had unknowingly created a fragmented, over-engineered setup that we didn’t have the capacity to maintain.
That’s when I was asked to figure out how to fix it.

---

## The Approach: Communicate 
I didn’t jump into solution mode. Not immediately. First, I talked to the entire engineering team explaining what I was trying to accomplish, a unified and consistent system that we could deploy without a lot of complexity.
Then I sat with each team, asking them about their solutions, the problems they were asked to solve and the decisions. I listened to what they were trying to solve and why they chose the tools they did and what I found was that everyone had good reasons and reasonable approaches and worse nobody had done anything wrong—they just hadn’t been asked to make one system..
Next we brought everyone together and mapped the duplication, discussed the tradeoffs, and asked a critical question: 
Do we keep moving forward with a microservices model, or is it time to consolidate into a single application?
in Ruby on Rails parlance, there’s a word for that kind of monolith: a monorail.

The Hard Choice (That Was Actually the Right One)
Ultimately, after all discussions and all opinions heard I was able to work with the team and come to a final decision to move toward a unified codebase.
This wasn’t about being “anti-microservices.” It was about being realistic. A single Rails app would allow us to:

- Standardize dependencies
- Reduce duplication
- Simplify deployments
- Focus our energy on veterans, not infrastructure


This solution wasn’t glamorous, cutting-edge or even that difficult in the end, but it was the right call for the team, the product, and our end users.

---

## What I Learned: Alignment
This experience reminded me—again—that solving people problems is way harder than solving technical ones.
The fragmentation didn’t come from bad code. It came from the best of intentions  a group of folks that were moving fast and trying so hard to create solutions that helped veterans.  What we created was a lack of alignment and the fix wasn’t a new tool, it was communication, trust, and shared ownership.
> __As technologists, we are asked to solve problems, create solutions and systems. But we have to remember that systems are built by people and if the people aren’t aligned and organized, even the best technical solution won’t hold.__

---

## Final Thought: Communicate
If you’re a tech leader, or hoping to become one, remember: your job isn’t just to write great code or make smart architecture decisions. Your job is to create alignment, help your teams work together. To succeed in your role you must listen, guide, and make space for collaboration so they can succeed in their roles.  That’s the real work and it’s the most rewarding work you can do:
>__Deliver solutions that meet the needs of your end users and are aligned, are maintainable and are secure.__

---

Photo by <a target="_blank" href="https://freerangestock.com/photographer/rawpixel/4032">rawpixel</a> from <a target="_blank" href="https://freerangestock.com">Freerange Stock</a>
