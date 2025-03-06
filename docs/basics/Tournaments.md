---
title: Tournaments
contributors:
  - Cj
  - Henrik
---

A Tournament is a special type of Challenge where matches are used to create the toplist.

- In Tournaments, each submitted solution is run nightly against solutions from several different programmers and gets scored only in comparison with other solutions.
- In other (non-Tournament) Challenges, each solution gets scored on its own, immediately after it is submitted. These individual scores are then used in the toplist.

Each night, a tournament takes place where code from participating programmers compete with others in multiple matches.
Each submitted solution receives an initial [ELO-rating](https://en.wikipedia.org/wiki/Elo_rating_system) of 1000 and this rating is then modified with each match.
When the tournament is finished the final rating for the night is used for the toplist.
The principle behind this rating is that the rating increase is large when winning against a much stronger (higher rated) opponent, and only a minimal increase is received when beating a much weaker opponent.

ELO rating normally only handles 1 vs 1 matches, but since we may have three or more solutions competing in the same match we use multi-player ELO, as described [here](http://uk.diplom.org/pouch/Email/Ratings/JDPR/describe.html).
A solution starts the next nightly run with the ELO rating it had from the night before, and we use fewer and fewer matches for a solution once the ELO rating has stabilised.
If, however, a modified version is submitted we allow more matches to establish the ELO rating of the modified solution.

Read more about Challenges [here](Challenges.md).


## Create your own Tournaments

On Future Skill it is possible to create your own Tournaments!
This is done with the Freecode creator that you can access from the create option (the "+ Create" button to the right on the Exercises, Challenges and Freecode pages).

Read more about the Freecode creator [here](../create_an_exercise/Freecode_creator.md).
