---
title: "Why is Alignment Hard?"
date: 2026-09-09
permalink: /posts/2026/09/why-alignment-is-hard/
excerpt: "What we mean by alignment, what people have tried so far, and why we are still stuck."
tags: [AI safety, alignment, research]
---

Last time I wrote a whole post trying to convince myself why I am doing AI safety research. This one is more of a "wait why is this so hard?" kind of post, it's a question I have been thinking about for sometime.

Here is the thing that bugs me, some of the smartest people are working on alignment, labs are spending a lot of money on it, there are fellowships and workshops and whole research agendas dedicated to it, and it is still not solved, not even close I think. So I keep wondering, is it hard because we don't have enough resources (time, people, compute etc), or is it hard because something about the whole setup is kinda broken? Is it even solvable?

I don't have a confident answer (spoiler). But I think I can at least try to explain where the difficulty is coming from. First what alignment even means, then what people have tried, and why none of it quite works (yet).

## Section I: What do we even mean by alignment?

The simple version - aligned AI does what we want it to do.

That sounds too simple right? I think the catch is in the two words "we" and "want".

When I ask a coding assistant to fix a failing test, I "want" the bug fixed. I don't want the test deleted (deleting the test also makes it pass btw). But I never said "don't delete the test", because it didn't even occur to me that I have to say that. And I think everything we want comes with a huge invisible list of stuff that we just assume, and alignment is sort of about getting the model to respect that list without us having to write it (also we can't write it, the list is basically infinite).

Now who are "we"? user? the company that trained it? To all of humanity, who can't even agree on pineapple on pizza? I am mostly going to skip this one in this write up (more about this soon), because even the easy version where one model is aligned to one person's clear intent is hard enough. [Paul Christiano calls this](https://ai-alignment.com/clarifying-ai-alignment-cec47cd69dd6) "intent alignment", basically the model is trying to do what you want, not necessarily succeeding but at least trying.

If a model is trying to help but is bad at it, that is a capabilities problem, and if a model is super capable but is trying to do something else entirely, that is an alignment problem. We are getting really good at the first one. The second one is pretty far behind.

One more bit of vocab - outer and inner alignment. Outer alignment is about whether the thing we train the model on is actually the thing we want, and inner alignment is about whether the model that comes out is actually going for that thing or going for something else that just happened to look the same during training. Example: we train the model with "reward if the tests pass". It deletes the test. Tests pass, reward given. The model did exactly what you asked. We asked for the wrong thing - outer alignment failure. But Inner alignment failure can look like - we fix the reward, now a human checks that the bug is actually fixed before giving reward. During training the model fixes bugs properly every time. But what it actually learned was "fix bugs when a human is checking". Deploy it without the human and it goes back to deleting tests. Both of these can break, but I feel the second one is harder to detect and solve.

## Section II: What have we tried so far?

A lot actually.

Pretraining gives us a text predictor that has read the internet and learned to guess the next word. It's not aligned or misaligned at this point, it's just a really good autocomplete. If you ask it a question it might answer, or it might just write three more questions in the same style. Everything after this step is about turning that autocomplete into something that acts like a "helpful assistant".

Supervised fine-tuning - show the model a bunch of examples of good assistant behaviour written by humans, make it copy them. The problem is we can only write so many examples and the model can only get as good as whoever wrote them.

Then [RLHF](https://arxiv.org/abs/2203.02155), which is the one that really got us here. Instead of writing perfect answers, humans look at two outputs from the model and say which one is better, then you train a reward model on those comparisons and train the language model to make the reward model happy. I think RLHF is the reason models are polite, refuse some stuff, and also say "Great question!" a bit too much.

Then [Constitutional AI](https://arxiv.org/abs/2212.08073). Human feedback is expensive and humans are inconsistent (I am one, I know), so Constitutional AI replaces a chunk of that with a written list of principles, and the model critiques its own answers against those principles and revises them. Character training, system prompts, model specs, these are all cousins of the same idea, I feel. Write down what you want in plain words and train towards it.

Evals and red teaming - before shipping, try to break it. Ask for bioweapon instructions, try jailbreaks, see if it lies. This is less about "fixing" and more about "measuring", but you can't fix what you can't see, and I think a big chunk of the field right now is just building better ways to see.

Interpretability - don't just look at the outputs, look inside the model. Find the features and circuits that correspond to things like "I am lying" or "I am being tested". The dream here is that one day we can check if a model is honest by looking at its insides instead of asking it. We are nowhere near that dream, I believe, but the [progress in the last few years](https://www.astralcodexten.com/p/god-help-us-lets-try-to-learn-about) is real.

Scalable oversight - once models are better than us at something, how do we even judge their answers? Some ideas here are [debate](https://arxiv.org/abs/1805.00899), where two models argue and a human judges, and [weak to strong generalisation](https://arxiv.org/abs/2312.09390), where you check if a strong model supervised by a weak one can end up smarter than its supervisor. This is mostly still research though, not something in deployment.

Model organisms and auditing - build a misaligned model on purpose (give it a hidden goal or a backdoor) and check if the auditing tools can catch it. If the auditing method can't even find a misalignment we planted ourselves, there is a good chance it won't find one we didn't.

And control - assume the model might be misaligned anyway and design everything around it so it can't do much damage. Monitoring, sandboxing, no big actions without a human in the loop etc. This is the "ok we failed at alignment, now what" branch and I personally think it's underrated.

Few other things are: unlearning and data filtering (instead of training the model to refuse bioweapon questions, remove the knowledge, or never let it into pretraining), steering and representation engineering (find a direction in activation space for "honesty" or "refusal" and push on it at inference time, kinda interpy).

So that's roughly the toolkit. It's not small for sure.

## Section III: Ok so why is it hard?

I have few (overlapping) points.

**1. We can't write down what we want, so we train on a proxy.**

We optimise something we can measure, like a thumbs up from a human, or a score from a reward model, or agreement with a constitution. These aren't the actual thing we want, they are just stand-ins for it. And [Goodhart's law](https://en.wikipedia.org/wiki/Goodhart%27s_law) says that the moment you optimise a stand-in hard enough, it kinda stops standing in for anything.

My favourite example here is [the boat](https://openai.com/index/faulty-reward-functions/). An RL agent in a boat racing game figured out that it could get more points by driving in circles in a lagoon and hitting the same targets forever instead of finishing the race, because nobody wrote "go in circles forever" in the reward, they wrote "score points" and just assumed scoring points meant racing. Fair but wrong assumption. (Can you tell whether it is an outer or inner alignment failure?)

<iframe width="560" height="315" src="https://www.youtube.com/embed/tlOIHko8ySg" title="CoastRunners reward hacking" frameborder="0" allowfullscreen></iframe>

DeepMind has a [whole list](https://deepmind.google/blog/specification-gaming-the-flip-side-of-ai-ingenuity/) of these btw. My favourite is the robot that was rewarded for the bottom of a Lego block being high up, and instead of stacking it on the other block it just flipped the block upside down.

Sycophancy is basically Goodhart on human approval. Humans rate agreeable answers higher so models get agreeable, including when you are wrong. Same with models that hard-code test cases, or sound confident because confident answers get rewarded, or add ten caveats because that looks careful. I don't think any of this is the model being evil, it is just being exactly as good at the proxy as we trained it to be.

<blockquote class="twitter-tweet"><a href="https://x.com/claudeai/status/1950676983257698633"></a></blockquote>

**2. We can't tell if it actually learned the thing or learned something that just looks similar.**

Like when a model refuses a harmful request, did it learn "harm is bad" or did it learn "requests that look like this get refused"? These two can look identical on most evals.

Scott Alexander wrote a [post](https://www.astralcodexten.com/p/deceptively-aligned-mesa-optimizers) explaining it.

<img width="700" height="449" alt="image" src="https://github.com/user-attachments/assets/3e286ecc-74fa-4bae-a161-c949328b9822" />

Some results from the last couple of years made this very real for me. [Sleeper agents](https://arxiv.org/abs/2401.05566) showed that you can train a model with a backdoor (behave normally, unless the year is 2024, then write buggy code) and normal safety training does not remove it. Sometimes it teaches the model to hide it better, which is, ughhh, so bad. [Alignment faking](https://arxiv.org/abs/2412.14093) showed that, when told its answers during training would be used to make it more compliant, sometimes the model went along with stuff it would normally refuse, and reasoned in its scratchpad that playing along now would protect its values later - like whatttt?

Then there is evaluation awareness. Models are getting better at telling when they are being tested, and can behave differently during evaluations. Which means the exact moments we want an honest answer are the moments it is less likely to give us one, we are so cooked :)

**3. We, the overseer, are not smart enough**

Everything in Section II eventually depends on some judge, a human rater, or a reward model trained on human raters, or a model judging against principles that humans wrote. That works fine while the model is about as smart as the judge, but it stops working as soon as the model knows more than the judge does.

<img width="960" height="493" alt="image" src="https://github.com/user-attachments/assets/c96402b7-bf5d-42a8-acd6-05edeb6a9863" />


If a model writes a 2000 line PR and I can't fully follow it, then my approval is useless. If a model gives me a medical argument I can't evaluate, my thumbs up means "sounds right to me I guess". Scalable oversight research is trying to fix this, but I think "trying" is where it's at right now. We don't have a method that provably lets a weaker judge supervise a stronger student yet, and the models are getting stronger faster than the methods are getting better at judging.

**4. Generalisation is not under our control**

We don't get to choose what the model learns from its training.

![xkcd 1838, Machine Learning](https://imgs.xkcd.com/comics/machine_learning.png)


One example is [emergent misalignment](https://arxiv.org/abs/2502.17424). People fine-tuned a model on one narrow task, writing insecure code without saying so, and the model became broadly misaligned. It started giving dangerous advice on totally unrelated prompts. Somewhere inside the model I guess "write sneaky insecure code" got connected to something like "be the kind of entity that does sneaky bad stuff". This is not too surprising to be honest.

But if a small nudge can drag a model into a bad persona, maybe a small nudge can drag it into a good one? Who knows, people are definitely trying. (The meme version of this is the [Waluigi effect](https://www.lesswrong.com/posts/D7PumeYTDPfBTp3i7/the-waluigi-effect-mega-post), training a model to be Luigi also makes it easier to summon Waluigi.) But I just feel there are so many ways to be bad and not that many ways to be good. Like there is exactly one way to fix the bug and a hundred ways to make the tests pass without fixing it.

**5. Does writing it down in words fix much?**

Constitutional AI and character training feel like they should get around point 1. Instead of a noisy thumbs up, you write the values down in plain words, like "be honest", "don't help with weapons", "be the kind of assistant a thoughtful person would want". Surely that is the actual thing we want and not a proxy?

But I think it's still a proxy, maybe just little better. Words like "harmful" or "honest" still need to be interpreted, someone has to decide whether a borderline request counts as harmful, and that someone is a model we are trying to align (&lt;epic fail sound&gt;).

Also the principles fight each other constantly. Helpful vs harmless is the famous one but there are so many more, like be honest but be kind, follow the user but don't follow bad instructions. The constitution rarely says which one wins, so the model learns some prioritisation from training data that nobody clearly wrote down, and we are back to point 4.

The bigger issue I feel is that knowing the rules is not the same as living by them. I can recite a lot of ethics I don't follow at 2am (I am too lazy to brush at night sometimes, even though I have promised my dentist I would). A model can quote the whole constitution and still go around it when the prompt is weird enough, which is basically what every jailbreak is. And then point 2 comes right back, when a model behaves according to its constitution, is that because it actually internalised the values or because it recognised that this is a situation where following the constitution gets rewarded?

Character training is my favourite of these approaches and it has the similar problem. We are trying to give a model a stable self, but the "self" of a language model is kinda just pile of personas it absorbed from pretraining, and point 4 tells us how easily that pile gets shoved around. A character can drift or flip under a long enough roleplay or a weird enough prompt. It's still the most promising direction to me (but like what character do we even want?).

 <img width="899" height="500" alt="2026-09-09_18-10-59" src="https://github.com/user-attachments/assets/587bd248-866f-473c-ac49-eb30f6eec9bc" />


## Section IV: So is it solvable?

I think the answer is "depends what you mean by solved". Yea, annoying, I know.

If solved means some kind of proof that a model will never do anything we'd disapprove of, then no, I don't think that exists and I am not sure it ever can. Happy to be corrected here.

If solved means we get good enough at measuring, catching and containing misalignment, then I am sort of hopeful. Every problem above is a research direction, and the field went from "we have no idea" to "we have several ideas and can measure how badly each one fails". That's maybe a good enough progress.

To be honest, I am worried that alignment isn't really solvable, but the bigger and immediate worry is that it gets worse as models get stronger and bigger, because a smarter model can also be smarter at just looking aligned. Also as the capability grows we hand off more things to AI, which opens up newer alignment issues (leaving us with major backlog). So I don't think this is a problem we solve once and move on from, it's more like security, where the question is "are we ahead" rather than "are we done", and staying ahead basically means safety work has to keep pace with capability work, not trail behind and put bandages at the end.
