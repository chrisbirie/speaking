<a href="../../"><img src="../../assets/wordmark.png" alt="chris birie — home" class="brand-wordmark" style="max-width:240px;width:55%;"></a>

# Transcript — Designing Data Pipelines That Don't Hate You Six Months Later

**Stir Trek 2026** · AMC Easton, Columbus, OH · May 1, 2026 · [Watch the recording](https://youtu.be/Dwg9FyWeezU?si=TA5TcICXv9Na8yFn)

> Lightly edited from the talk recording for readability — wording and meaning are the speaker's own.

---

## Intro

Welcome. My name is Chris Birie, and I'm doing a talk today on designing data pipelines that don't hate you six months later — because who cares if it hates you right now? That's what we're designing for. It's always six months down the road when things come back to bite you.

I'm hoping to make this as entertaining as possible, so this is going to be a sci-fi-mixed-with-kung-fu-legend story that happens to involve some data engineering.

So, the "why should you listen to me" part. I've been in data engineering for most of my career — about 13 or 14 years at this point — doing all kinds of things: DevOps, reporting, ETL pipelines. I've basically touched everything.

A warning about me before we go further: I have ADHD. I was diagnosed in college, and it explains so much about me — I have so many interests and hobbies, always picking up new things and juggling everything at once, whether it's homelabbing or 3D printing (which are kind of related), or getting back into yard work and gardening. I'm constantly going from thing to thing. When I was younger and undiagnosed, that got seen as something bad. Now I see it as kind of a superpower. And now that I have a five-year-old son who is exactly like me, I see it in him, and I get it — I'm glad I went through it so I can help him get through it.

Honestly, this is kind of what I thought my superpower would lead to — being a Power Ranger, a superhero. (And yes, I used AI to generate all the photos in this presentation. You're going to have to deal with it.) My true miscalling was probably WWE professional wrestler — I love entertaining people, being up here. I loved wrestling as a kid; somehow if I got hit in the head, people would apologize and I'd be like, "What? That didn't even hurt." I never pursued it, but it feels like my true miscalling.

Speaking of getting hit in the head — over COVID I picked up doing kung fu. I saw a Bruce Lee documentary, looked up kung fu near me, and a mile down the road was a studio. I did the intro lesson, it was awesome, and I've been doing it ever since — this July it'll be six years. I'm an assistant instructor at Western Lotus Athletics and Martial Arts Club here in Columbus. (Shameless plug: tomorrow at 10 a.m. at Seventh Son Brewing, we're having a free self-defense class — feel free to stop by.) My ADHD brain took over and I've learned so much, not just about Wing Chun — the branch of kung fu I do — but about Chinese culture and kung fu movies. There are so many life lessons in it, and it's going to influence this presentation a lot. So be prepared. (No, I'm not going to do a kung fu demonstration. I could. But no.)

## Why this talk

A little background. I was a tech lead for a predominantly data-focused team, and we had a legacy application data pipeline where I'm pretty sure some of the source code was written when I was in second grade. It just got built on top of itself over and over. As tech lead I was on call, and it was bad — I was getting called most nights I was on call, and even when I wasn't, my developers would call me because they didn't know what to do. It was a mess. That shaped this story.

## The 3 a.m. call

Our story begins with our hero, and this might look familiar. It's 3 a.m. You're woken up by several pages and calls. It's your boss: it's the end of the quarter, the business folks are freaking out, their reports aren't populating. You sit down, pull up the log, and there are so many error messages you don't even know what half of them mean — an inherited mess. Slack is full of "what's going on, why didn't QA catch this, when will it be fixed." Seven, eight missed calls. Dozens of missed messages. You don't even know where to begin, and you get frustrated.

And that's about where, if this were a movie, you'd get the record scratch — *"you might be wondering how I got here."* Like any good movie, it starts in the middle and goes backward. So how did we get here? It all traces back to six months earlier.

## Enter Bruce

Meet Bruce. He's the antagonist of our story, but he's a good guy — a brilliant developer. He was fast, he used AI, he shipped what he needed early. The pipeline worked beautifully for what the business wanted at the time. Everyone was happy; Bruce got a pat on the back, probably a bonus. We've all been there. Then Bruce moved on — inside or outside the company, we don't know — and the pipeline got thrown over the wall to you. This was Bruce's pipeline. Now it's your problem.

The main point of this whole thing: pipelines built on shaky fundamentals don't fail on day one. They fail when there's pressure — usually six months or a year later, when everyone's moved on, the warranty period ended, and it's now your problem.

Just like in most martial arts, they care a lot about your footwork. They want a strong foundation so you can't get pushed off balance. Same with any software or data pipeline: if it's built on shaky fundamentals, it's not going to last. The first thing I learned in martial arts was weeks of drilling stance, footwork, fundamentals. The first form I learned is called Siu Nim Tau — "the little ideas" — tiny ideas, one at a time, over and over, before you ever get to sparring. Without those fundamentals, I couldn't do anything.

Bruce wasn't a bad guy and he didn't build a bad pipeline. He built one that worked perfectly for the fight he had. The problem is the fight changed — it was no longer the small dataset; they started putting much more data through it. Bruce is gone, his pipeline lives on, and right now at 3 a.m. it's on fire. It's hard not to blame Bruce, but he delivered what was expected of him.

But what if we could go back six months and change the outcome? What would we teach Bruce? We're going to go over five lessons — five chances to help Bruce do the right thing so we don't get woken up at 3 a.m.: **schema evolution, idempotency** (I hate this word — I can never say it), **observability, testability, and designing for change.**

A warrior — or a developer — who trains only for the tournament will fall when the real fight comes. That's how we need to think about our pipelines: not just what's right in front of us, but how to make them flexible and lasting, so no one curses my name because it's on that commit.

## Lesson 1 — Schema evolution

Back to our story. What would a responsible data engineer do here? Dig in, find out what's going on, fix it. That's not what our guy did — he passed out on the couch and put on *Back to the Future*. (He has a mug that says "data > sleep," which is playing out right now.) Fading in and out of consciousness, he hears Doc Brown: "If my calculations are correct, when this baby hits 88 miles per hour…" Wait a second — what if we found the DeLorean and went back six months?

Six months ago, Bruce trusted that the upstream systems sending him data wouldn't change. They didn't during the warranty period. But before that 3 a.m., a source system changed its database, API, or file layout — and didn't tell anyone. They also sent some corrupted data: a delimited file with an extra comma that shifted every column over. Bruce's pipeline silently exploded, as things usually do. No warnings — just a business person saying "I don't have the right data, why?" and an array of errors with no idea where to start.

Someone wise — maybe a relative of Bruce's — once said: *adapt what is useful and reject what is useless.* In martial arts, if something isn't working, get rid of it. I'm not the quickest mover, so I don't lean on that — I'm strong, so that's where I make my hay. Nobody says I'm doing kung fu wrong, because I adapt what's useful to me.

Bruce trusted what the source sent and never built anything to test or question it. So our pipeline should let through the front door only what's useful, and reject the useless. Picture a bouncer in front of the pipeline: I'm not letting the data in if it doesn't have a valid ID — I'm going to test that ID, make sure it's real, and only then let it through.

Some concrete ideas:

- **Make fields nullable by default.** Instead of crashing when a non-nullable field gets a null, accept it and have a mechanism to say "there shouldn't be a null here — don't send it downstream, fix it, or put it in a rejection queue" so the pipeline doesn't crash.
- **Use schema contracts.** Well-architected APIs have versions and schema contracts — "this is what the data is going to look like." Trust your data like that. There are plenty of schema registry / validation services; if you're in the Apache world you probably know Avro. They're often used for APIs, but you can use them for files or database connections too: make sure the data is how it should be, and if not, set it aside and move on.
- **Defensive parsing and validation.** Never assume incoming data is how you want it. Use regular expressions and the like: "this should be a number — it's going to be a number; if not, you're not getting through the front door."

Your pipeline's job is to be paranoid at the front door so nothing weird makes it into the house. None of this is about predicting every change — things change, that's fine. It's about building a system that bends with change instead of breaking.

So we go back, tell Bruce, and he's like "yeah, I could do that" — types it into his prompt, done. Back to present day: you wake up to an urgent 3 a.m. call from your boss. History is somehow repeating, but it feels familiar. We — the audience — know we went back and got Bruce to implement some things: fewer missed calls, fewer messages. The error log still has a lot in it, but a little less.

## Lesson 2 — Idempotency

We all do what any responsible data engineer would do: fix the problem. Right? Wrong. You think about *Back to the Future* again, but decide your future isn't very good — you want something deep and serious, so you put on *Interstellar*. You hear Matthew McConaughey: "Murph — no, don't let me go, Murph." The whole plot, he goes into interstellar space, loses a bunch of time, and ends up able to see into his past. What if we could enter the tesseract? (I won't explain the tesseract — my friend Kirk is here, he'll explain it great; he was the first person to make it make sense to me.)

Looking through the logs: a lot of those messages were duplicate data. How? Six months ago Bruce built the pipeline to run once on a dataset, start to finish, no interruptions. That assumption was baked into every line: we'll only see this data once. But overnight the database went down halfway through a load and it errored out. The help desk — so helpful — said "it's just a database thing, let's rerun the failed job." The problem: it's not fine. Now you've got duplicate records and primary-key violations.

A wise man in a movie once said: what, you think a fight is one throw, one kick? Of course not. In a real fight you should expect to be hit more than once. The fighter who trains for one clean exchange — "boom, I won, we're done" — is useless in a real fight, where the other person isn't letting up. We need a pipeline that can run safely even when it gets the same information twice. The database will come back up mid-load; your orchestrator or help desk will retry the job on a timeout; people will ask you to rerun last week's dataset. Without idempotency (there — I said it right), it's a nightmare of errors and manual cleanup. We don't want to get woken up at 3 a.m. to "delete everything from yesterday."

Some fixes:

- **Deterministic keys.** If you see this data and it's repeated, don't process it again — skip it.
- **Upserts.** If you don't match on the key, insert; if you do, update with the new information. Super helpful.
- **Safe, bounded processing windows.** Design the pipeline to process data in clean, bounded windows — e.g. 15-minute increments. If something goes wrong you start where you left off, instead of "I just need everything new since the last run."

Ultimately you want the pipeline safe to retry — that's what idempotency means. A practical test: if your pipeline fails halfway through and orchestration retries it automatically, does it make things worse? It shouldn't. If it does, you've got some thinking to do — how can this run twice safely, what cleanup can happen automatically?

We taught Bruce the lesson; the past changed. Unfortunately the boss still calls at 3 a.m. — there are still errors.

## Lesson 3 — Observability

We're going to fix the problem. Right? Wrong — we fall asleep and play *Hot Tub Time Machine*, because *Interstellar* made our brain hurt. You think, "I could use a Chernobly energy drink and a soak in a nice hot tub" — wait, the hot tub *was* a time machine. We could go back and tell Bruce something.

Just like in a fight, you don't charge in blindly — you have to *listen*, like Bruce Lee says. Bruce got the data flowing and called it done. No logging beyond basic errors, no metrics, no alerts. The pipeline ran, and that was enough. Then it failed silently for three days and we had no idea — because we had no alerts. The only way we found out was a business partner saying "I'm not seeing data for the past couple days." That's three days of bad data.

The warrior who fights without information fights twice — you're fighting two battles, because you have to figure out what's wrong *before* you can fix it. With observability — row counts at every stage, null-rate tracking, latency monitoring, alerts and dashboards on anomalies (not just an exception when things blow up) — you fight that battle far more easily. You can be proactive. We don't want to be called at 3 a.m.; we want to fix problems between 9 and 5. No one's brain works at 3 a.m. And remember: **hope is not a monitoring strategy.** "I hope this pipeline works" is not a strategy.

You go back and tell Bruce. The past changes. You wake up at 3 a.m. to your boss — "this feels familiar, like Groundhog Day." Things are a little different: not as many errors, calls, or Slack messages. But there are still issues and the business partners' data still isn't right.

## Lesson 4 — Testability

Responsible data engineer — you fix the problem, right? Of course not. You fall asleep and, feeling nostalgic for young Keanu Reeves, put on *Bill & Ted's Excellent Adventure*. Strange things are afoot at the Circle K; there's a phone booth that could take me to the past. What would I tell Bruce? Testability.

This is a huge topic, but: six months ago Bruce tested his pipeline in production. "We don't have the right data, so I'll just test it in prod." It looked right, he shipped it. Today a bug was found — records with nulls in key fields were being silently dropped, there was far more volume than it could keep up with, and everything was super slow. Bruce's lack of testing meant every change was a gamble: "I hope this works, I hope I don't break anything else."

Bruce Lee said: *I fear not the man who has practiced ten thousand kicks once, but the man who has practiced one kick ten thousand times.* What does that have to do with testing? We need accurate, repeatable tests so we can go in with confidence that this works and is what it should be — the same thing over and over, making sure we didn't break anything. If you've never run it outside of production, you haven't really run it at all.

In the age of AI this is easy — you have your code, so ask it to write unit tests for these fixtures and transformations. Use **parameterized inputs** so you can point at different data sources, not always production — you want to test without consequences. A big one for data pipelines is **performance testing**: there's no way a lower environment has the same data, but you have to try. We're getting far more data than we ever did — more visitors, more information, all going through the pipeline. Test for more than you think you'll get, so you know what happens when you get it.

We tell Bruce: use AI, write unit and integration tests, do performance testing so that when volume doubles or triples we don't hit problems. "Sure, no problem." We hop in the phone booth, the past changes, and you wake up at 3 a.m. again. Bruce did a lot of good things, but there are still a couple he didn't do — we're still getting called.

## Lesson 5 — Designing for change

Responsible data engineer — fix the problem? Of course not. You fall asleep to *Avengers: Endgame*. Between "I see this as an absolute win," "Avengers assemble," and "I am Iron Man," you wonder: what if I got Ant-Man's van, went into the quantum realm and a parallel universe, and could snap a lesson into Bruce? This is the biggest lesson — if you walk away with one thing, it's this. It's what all the others build toward.

Six months ago Bruce's pipeline was right — tight, clean, did exactly what it was supposed to for the problem in front of him. It handled the exact volume, schema, and delivery cadence expected. It was right for the fight he knew. But now it's hundreds of times the volume, requirements shifted, the source restructured — so much changes in our world at a rapid pace. The pipeline didn't bend with the pressure; it shattered. At this point you might need a full rewrite — not because the original was bad, but because it was built for one specific fight, and that fight changed.

One of Bruce Lee's most famous quotes: *be like water, my friend.* Water is formless; it takes the shape of whatever container it's in. He also says: notice that the stiffest tree is most easily cracked. If you're from Ohio, we've had bad windstorms — brittle trees crack and fall on your house and you're filing insurance claims. But bamboo or willow survives by bending with the wind.

That's how we need to develop pipelines:

- **Modular, non-monolithic stages** so there's no single point of failure — if something changes you fix it in one decoupled place without touching everything else. This isn't just a software idea; it's for data too.
- **Idempotency** (there's that word again) — be aware of it.
- **Backfill-friendly.** At some point requirements change and you need to reprocess historical data. Plan for "what happens if we need to reprocess the last two weeks?"
- **Smart partition strategies** — on databases or data. Instead of processing a whole week at once, process a couple of days at a time, so you're not pushing as much volume. If volume spikes, shrink the window to every eight hours.
- **Loose coupling between layers.** Most segmented pipelines still fail because layer B depends on A, C depends on B, all the way down. Make layers as independent as you can while keeping the data flowing correctly — which is where good design and testing come back in.

The past has changed; we've got the Infinity Gauntlet. The pipeline Bruce delivered is magical. You wake up at 3 a.m. — sleeping with no worries, cuddling your favorite dinosaur. That's the 3 a.m. I want. Look how happy he is.

## The remake

Bruce is redeemed; this is the remake. Where before you had silent failures at 3 a.m., duplicate data from reruns, skewed numbers, an untestable app, no meaningful performance testing, and a full rewrite when requirements shifted — now, because the audience went back several times to fix this:

- Anomaly alerts flag data that doesn't look right before it becomes a problem.
- It's idempotent — safe to retry and reprocess, no duplicate-data worries.
- There's a full test suite — no consequences for changing, and we're confident it handles the data we throw at it.
- Schema evolution is handled gracefully — if upstream changes, we get alerts to look before it happens.
- It's modular — the pipeline bends like a good pipeline instead of breaking.

I had a pipe burst in my house a couple months ago — awful, and it was because it was brittle. It didn't bend when it should have. That's not where we want to be. And again, Bruce isn't the bad guy — at best uninformed, at worst just not thinking long-term. That's where we need to be with our pipelines, or software in general; these principles apply to anything.

## Recap

Before you go watch the movie, the principles again:

- **Expect your schema and data to change.** Plan for it.
- **Make it idempotent.** Run the same data multiple times and nothing bad happens.
- **Observe everything.** Don't go into the fight blind — I log even the smallest things so I can go back and see trends and dashboards.
- **Test without consequences.** Testing in production sounds cool and exciting, but if that's all you do, there will be consequences.
- **Be water, my friends — design for change.**

Bruce left before he could fix his pipelines, which is why we had to go back and help him. But you're still here — you can fix yours. Start small. Take one of these and say, "I hate getting woken up at 3 a.m. — I'm going to do something about that." There's nothing better than a good night's sleep when you're on call. You've already seen the real disaster film — Bruce made it for us. Now go enjoy the movie.

## Q&A note

On the pronunciation: when I looked up how to say *idempotent*, there was a soundbite where you could say "idempotent" or "ee-dempotent." We're going with idempotent. That's what we're going with.

Thank you — I appreciate you having me. I hope you take something from this you can go back and implement, whether that's with your sauna or your data pipelines. Good luck, and enjoy the movie. There's a feedback QR code up here, and you can connect with me on LinkedIn if you'd like.
