---
title: Future Skill GPT Instructions
contributors:
  - Lamiya
width:
  standard: 250px
  wide: 500px
---

To create exercises quick and more efficient Future Skill GPT can be used. It is based on the original GPT but provided with specific datfa that helps to create parts of the exercise in an effecient way. You can access it using the link below:

[Future Skill GPT][def]

[def]: https://chatgpt.com/g/g-680102f8d4b48191a341df1e0b393ce2-future-skill

## How to Use the Custom GPT for Creating Exercises

In the GPT Exercise, the output is divided into four main parts:
   • Task → Information for the Description tab
   • Solution API → Code for the Solution tab (function skeletons)
   • Implementation → Code for the Implementation tab (core mechanics)
   • Flow → Theoretical background for the Flow 

**Important:**

Giving a command like "create exercise for <topic>" is not recommended.

**Why?**

Because the output will be too large to display properly all at once. Instead, create each part separately using the commands below.

## Available Commands

`create task`
: Generates the description of the task for each level.
: Includes examples (input/output) and explanations.
: GPT will ask:
	• How many levels?
	• Should levels be progressive (easy to hard) or independent (separate tasks)?

`create solution API`
: Creates the function skeletons users will fill in.
: Helps with function names, argument types, and expected outputs.

`create implementation`
: Generates core backend logic for the exercise.
: Some manual adjustments might still be needed to match your exact needs.
: Note that if the configuration (test data) part is missing due to long code, use the next command

`create configuration`
: Generates only the configuration (e.g., random test data generation) if needed separately.

`create flow`	
: Produces all theoretical knowledge needed to understand and solve each level.