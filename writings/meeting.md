# Fantastic meetings and where to find them

Anecdotally most people in the software industry hate meetings; a meeting is a soul-sucking dementor. Jokes aside, I think it's fair to say that people feel meetings are taking too much time and the process is often painful.

In this post, I'll try to provide a few rules of thumb that I believe can make meetings less painful and more efficient.

## Rule 1: Always reiterate your goal before start

How many times do you attend a meeting without a goal in mind?

> Me: Dozens of them

The `why` is the first question you should answer before stepping into a meeting room. If you can't tell why are we are having a meeting, there is something missing in your communication process. You should fix it ASAP.

In many cases, only a few people know the goal clearly. If we conduct a meeting without communicating the goal to the rest of team, one or more of the following might happen:

1. People realize they don't know the goal and start asking questions to figure it out
2. The rest of the team thought they knew the goal well enough and are using this incorrect understanding to make decisions
3. People who feel the goal isn't clear and then tend to follow an other's opinion

In first case, you lost time but still can reach the goal eventually. For #2 and #3 it gets worse:
you will get false sense of validation or invalidation on a decision as people agree (or disagree) without enough information.

**A quick remedy for this problem is to simply reiterate the goal in a straightforward manner before meeting start.** This will likely be a responsibility of meeting initiator. If you call for a meeting, you should make it clear why to have one. If you are attending a meeting without a goal, ask for one to be added to the invite.

This extra step only cost you a few minutes but the benefit is huge. It's important to discover what people don't know as soon as possible (like catching bug in software).

## Rule 2: Limit the level of details in discussion

A common theme in meetings is `discussion`. Software development is largely about making decisions. Quite often individuals can make decisions on things they own but sometimes a decision happens to have bigger implications. This warrants a meeting among the team or between several teams.

It is important to conduct discussion in an efficient way. Unsurprisingly, this is not the case for many organizations.

From my experience, one big problem in a technical meeting is that people tend to open up related questions before a discussion point is concluded. I believe this has to do with human nature; discussing things that they find familiar.

For example, when we are trying to reach agreement on domain model, an engineer might unconciously
start discussing implementation, like how do things map into relational model of our favorite RDBMS. From a programmer's perspective, it's like thinking on different levels of abstractions and ignoring the details beneath it to speed up decision making.

Sometimes, derailing is a necessary evil. Some high level decisions depend on some very specific details on low level component, for example when deciding what latency a service should guarantee. We must consider technical feasibility to make informed decision.

To avoid this problem, we have to **scope the level of detail during discussion**. There are several techniques can be used:

1. Assign someone to control the direction of discussion and try to resolve discussion point by point
2. Be explicit throughout the process (like a sport commentator) by
   * Repeating the discussion point (preferably in question form) before start
   * Note sub-conclusion points whenever an agreement/disagreement is reached
   * Note a conclusion to close the discussion before proceeding to next point
3. Derailment is only allowed if justification is provided, e.g. we need to discuss project budget first as it affects our decision of team resource distribution.

**Note:** The `sub-conclusion` can be really simple. Just reiterate on every significant progress in the meeting, eg. "It seems like no one thinks option B is going to work, so we are left with option A and C. Let's focus on A first to see if it fits our usecase"

Despite it's simplicity, you'll be surprised how effective point #2 is when put into practice. By explicitly stating the progress of meeting, you'll be able to grab people's attention and give the team a sense of progress by having fast feedback loop (We all love feedback loop!)

## Rule 3: Always have an agenda

We've talked about having clear goals and limit the discussion. It's natural to think aboud meeting agendas now given it will facilitate both points we've discussed.

**An agenda should be derived from the goal(s) of the meeting and be arranged in a logical way that eventually leads to the goal** (this sounds recursive, bear with me).

For example, if we are trying to decide what protocol to be used for multiple microservices, the agenda might look like this

```
* Problem to be solved including requirements
* Proposals
* Benefit cost analysis of each proposal
* Discussion and decisions
```

By trying to come out with an agenda, you will discover more details before having a meeting. Items like analysis and proposals should really be prepared before the meeting; people only care about the result of the analysis, not the progress. It is easier for people to prepare before meeting with a clear agenda.

The agenda definition can be driven by meeting initiator and collaboration with the team, allowing the rest of the team to add missing points in advance.

Things like this may sounds obvious but still it was frequently neglected and with us paying an avoidable price.

## Rule 4: Choose the right people to be in the meeting

Another common problem of meetings is that it involves the wrong people. Sometimes we include more people than necessary. Sometimes we leave out important people that can provide crucial information. Sometimes both.

People are sometimes lazily think about who to invite and decide to invite the whole team. Such a strategy might work but it is not very efficient. People specialize for a reason and letting people focus on their
expertise is more productive.

On the other hand, it's also easy to leave out important people, especially when they are in different team, and we need to make a decision involving cross team effort.

Choosing the correct people is only possible if you have a clear understanding on what to discuss and more importantly what NOT to discuss (see #1 and #2).

This rule, when executed properly, will save your organization substantial amount of time.

## Conclusion
To summarize, a fantastic meeting usually consists of

* **Clear goal**
* **Well defined agenda**
* **Close to zero derailment**
* **People with the right information**


I hope you find this post useful. Feedback is welcome. Feel free to drop me a message =)
