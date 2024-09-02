---
tags: programming, cpp, c#
---

Code documentation
- https://lemmy.world/post/8594774
- Generally:
- Functionality of the code, so ppl can just read the comment and know how to use it without reading the code
	logseq.order-list-type:: number
- Any edge cases or quirks with the code
	logseq.order-list-type:: number
- Why does this exist?
	logseq.order-list-type:: number
- Why not another way?
	logseq.order-list-type:: number
- In-line comments similar to point 1, that briefly describe what each line/chunk is doing. Avoid reader having to read code.
	logseq.order-list-type:: number
Documentation is like reading a storybook: if i have to stop my reading for abit and do a double take to figure out what the fuck a portion of the code does, it is not good code documentation.
- Self documenting code is a must. Meaning, properly named variables and functions, used in proper and expected ways. Reduced use/creating of functions with side effects.
- Does NOT mean no comments. Make it as readable like a book as possible. Simple shit like `int totalDist = distA + distB;` dont need comments. but more complex ones will need comments to maintain readability.
https://liw.fi/40/ 40 years of programming

Naming Conventions:
- Be consistent with vocabulary use. E.g. if you use "print", stick to that word for all other functions related to this. Don't use other similar words like "display", "show" if there isn't any subtle difference.
- Be clear. Don't use acronyms that people need a cheat sheet to look up. Don't use vague words for function, class and variable names.
- Put as much meaning as possible in around 4 words.