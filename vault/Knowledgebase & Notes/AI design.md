---
tags: ai, game design
---

Description: Random thoughts about how AI (or game AI) should behave to make it "more belivable"
Issue: Games that have voice lines sometimes get repetitive. E.g. press button to talk to character, character has nth to say so it repeats the same line
- Probably stems from the issue of being unrealistic, because the AI does not react to you prompting it to chat so many times.
- Possible Solution: Use generative AI to "patch" this behaviour. But if done long enough, the player can still recognize that the contents of the convo are essentially "i got nothing more"
- Possible Solution: Introduce a system to make the character irritated of spoken to too often. But this makes it a PITA to implement for every NPC and probably overkill.
Problem: Most AI don't react to player stimuli enough.
AI needs to "see" player intentions via the whole scope of the chat (or actions)
- or even, guess the player intentions. Just like a real dude! How to represent in data? no clue!
AI will need to have the foresight and plan ahead.

# Realistic AI Design
Difference btwn this and LLMs:
LLMs have NO understanding.

Model after how the human brain learns:
- Pieces of raw data are associated with ideas/meanings
	- Learning that a word is associated with a certain idea: "Tart" refers to a woman in a bad way. "Woman" also refers to a woman. Woman is an idea that raw data gets associated with.
- Raw data can come in many forms: Visual (still/moving), audio in, vocal audio out, taste, touch, even emotional. Need to make a system that allows an expanding data type list.
	- Classifying data will be hard. How do we tell a specific piece of data should belong in one of the existing sets, or make a new set?
	- Possibility of re classifying data type category
	- raw data can possibly be in multiple categories at once
- Groups of data associated together can mean something more. e.g. a sentence vs a word.
- After learning and classifying and storing data, how can this data be used??
	- In context of understanding a sentence, just do lookup??