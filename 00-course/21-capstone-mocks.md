# Capstone · Three Timed Mock Interviews

You've grown Crumb from one box to two regions. Now do it without the chapters open. Each mock is 45 minutes on a timer, alone or with a friend playing interviewer. Draw on paper or a whiteboard tool, talk out loud, then score yourself with the rubric below.

---

## Mock 1: Design Crumb (a recipe-sharing app)
Prompt to read out: *"Design a recipe-sharing app where users post recipes with photos, follow other cooks and see a home feed. 50 million daily users."*

Interviewer follow-ups to use (pick 2-3):
- A celebrity chef with 10 million followers posts. What happens?
- Photo uploads fail for users on slow mobile networks. Fix it.
- The comments table grows 30 GB a day. Now what?
- The primary region goes down for 4 hours.

Reference: chapters 1-17 of this course.

## Mock 2: Design a ticket booking system
Prompt: *"Fans pick exact seats on a live seat map for a stadium concert and pay. 2 million people try to buy 60,000 seats when sales open."*

Follow-ups:
- Two fans click the same seat at the same moment.
- Payment takes 40 seconds for some users.
- How do you stop bots?
- The seat map is hammered with refreshes.

Reference: [Ticketmaster](../03-problems/29-ticketmaster.md).

## Mock 3: Design a team chat app
Prompt: *"Design a chat app for companies with channels, direct messages and threads. One channel can have 100,000 members."*

Follow-ups:
- How do you deliver a message to 100,000 members?
- A user has the app open in three tabs and on their phone.
- Search all messages in a workspace.
- A gateway server crashes with 200,000 open connections.

Reference: [Slack](../03-problems/58-slack.md), [Chat system](../03-problems/09-chat-system.md).

---

## Rubric (score each 0-2)

| Area | 0 | 1 | 2 |
|---|---|---|---|
| Requirements | Jumped straight to drawing | Some features, no numbers | Core features, scale, latency and consistency stated |
| Estimates | None | Numbers but no conclusion | Numbers that drove a decision (CDN, sharding, caching) |
| End-to-end design | Incomplete at minute 25 | Complete but vague boxes | Every feature flows through a clear diagram |
| Data model and storage | Not discussed | Tables listed | Keys, partitioning and store choice justified by access patterns |
| Deep dives | None | One, shallow | Two or more with options and trade-offs |
| Trade-offs | No downsides mentioned | Mentioned when asked | Costs stated proactively, with "what would change my mind" |
| Failure handling | Not discussed | Mentioned redundancy | Failover, retries/idempotency, rate limits, monitoring |
| Communication and pacing | Long silences, ran out of time | Mostly clear | Stated a plan, checked in with the interviewer, recapped |

**16 total.** 13+ is strong, 9-12 solid with gaps, under 9: redo the chapters for the weakest rows.

---

## After each mock
1. Write down the two lowest rows.
2. Re-read the matching chapters and fundamentals notes.
3. Redo the same prompt three days later, then try a new one from [03-problems](../03-problems/).

**Back to:** [Course overview](README.md)
