---
title: Implementation
contributors:
  - Cj
  - Admin
---

This article goes through most of the information that one needs to write an implementation for executor version 3.
It contains sections about API details, sections about best practices, and some example code tests and challenges to get a complete picture.
It does not cover the challenge editor and its options, neither does it cover the canvas API.

Read more general information about the freecode creator here: [Freecode creator](../create_an_exercise/Freecode_creator.md)

Read more information about the graphical canvas here: [Freecode graphics](Freecode_graphics.md)

Read more information about the different challenge here: [Skeletons](../create_an_exercise/Skeletons.md)


## The Challenge class

The Challenge class is initialized with a context object and once initialization is done the step function will be called continuously until the challenge is completed.


### The context object

The context object given in the Challenge constructor contains data for the current run and has some additional useful functionality.
This section goes through them in detail.


#### level: int

The current challenge level.

#### is_interact_mode: bool

Returns True when the canvas is running in interact mode

#### canvas

An object that is used to interact with the canvas.
It is used to add new graphical elements, more details on them are covered by another document.
It is best practice to only modify the canvas in dedicated setup and update methods (by default `_setup_canvas` and `_update_canvas`).

##### width: float
Returns the total width of the canvas in coordinate units.


##### height: float
Returns the total height of the canvas in coordinate units.

#### console

An object that is used to interact with the `console`.
It has two methods: `log` and `debug`.
The `log` method acts similar to print and prints the arguments to the `console` for all players.
The `debug` method acts identically except it only prints when inside the challenge editor.

Some example of how to use the console [can be found here](Implementation.md#logging-and-printing).


#### solutions: list

The list of solutions partaking in the challenge together for this level.
There are more details on the solution objects later in this document.
This list will always be the same length as the number of solutions.
However, they will all be set to None when generating the initial state of the canvas (this affects the Challenge constructor, not the step method).


#### parallel

An object that acts as a proxy solution for making parallel solution calls.
It has the candidate methods specified in the API specification, but not the solution attributes.
More details about calling solutions in parallel is available in a later section.


#### living_solutions

A method that generates index-solution pairs for each living solution (solutions that have not crashed or timed out).
This is the recommended way to iterate through solutions in the step method.


### The solution objects

The solution objects contain information about the solutions partaking in the challenge level and is a way to call candidate methods on the solutions.
The solutions objects have the candidate methods specified in the API specification which is used to call the solution and get a result.
Calling a solution may result in a `SolutionException` (imported from `lib.exceptions`) which must be caught (the solution is no longer alive at that point).
Note that the challenge must not call a candidate method on a crashed solution.
The solution objects also have the following attributes.


#### name: str

The name of the solution.
This will either be the player name or a placeholder name.


#### index: int

The index of the solution.
This should be used to identify a solution.


#### console

An individual console for the solution.
The interface is identical to the global console except that it limits the output to the console of the specific solution.


#### alive: bool

A flag indicating if the solution is still running or has crashed/timed out.


### What to do in the Challenge constructor

The challenge constructor should do the following things (preferably in this order):

- Store the context and select the appropriate level configuration.

- Initialise the required challenge attributes.

- Initialise any additional variables used for the run.

- Setup the canvas.

The constructor should not:

- Call any solution methods (neither individual, nor in sequence or in parallel).

Note that the step method is not guaranteed to be called before the solutions call the API methods on the challenge, so any preparations needed for the API methods must be done in the constructor.


### What to do in the step method

The step method should do the following things:

- Call any solution methods (either individual, in sequence, or in   parallel).

- Update any variables, etc.

- Update the scores.

- Update the canvas.

- If appropriate, mark the challenge as finished.

It is important that the step method catches any `SolutionExceptions` raised when calling the solutions. The examples include best practices for handling solution calls (especially note how the Code test example handles multiple calls on the same solution).


### Making parallel solution calls

Calling the solutions in parallel is preferable if the challenge supports it (i.e. the actions of one solution does not affect any other solution until all solutions have acted).
To call solutions in parallel the `context.parallel` object should be used instead of the solution objects themselves.

The results of a parallel call is a dictionary with solution indices as keys and the solution result as value.
There is no need to catch `SolutionException` when making parallel calls.
Only solutions that lived before making the call are included in the dictionary.
The solution result is either a the return value from the solution or a `SolutionException` if the solution crashed, therefore it is necessary to make an isinstance check on the result (unless the return value is not needed).

The differences between sequential and parallel solution calls are highlighted in the [Challenge](Implementation.md#challenge-example) and [Parallel challenge](Implementation.md#parallel-challenge-example) examples.


### The API methods

The challenge should have methods matching the API methods in the API specification.
These methods should take a solution object as its second argument.
This solution object corresponds to the solution that called this API method.

Sometimes an API method has additional restrictions on its arguments.
If this is the case then they should be thoroughly documented in the challenge/test description.
In the API method a specific check should be put in place where a `SolutionException` (imported from `lib.exceptions`) with an appropriate message should be raised.
This will terminate that solution, taking it out from the challenge from then on.


### The Challenge attributes

There are a couple of required attributes for a challenge, these must be set in the constructor as they are used continuously throughout the challenge run.
They are listed here, together with the required type and a short summary of how it is used.


#### finished: bool

Indicates whether the challenge run is completed or not, once it is set to `True` the step method won't be called any more and the run will end.
Note that the run will also end if all solutions have crashed.


#### level_name: str

The name of the level.
For a code test it should be descriptive and echo what is being tested.
For challenges it is not as important to be descriptive, but it doesn't hurt.
Usually taken from the configuration.


#### max_score: int

The total possible score for the level.
For a code test this should usually be 10.
For challenges this value might not make sense, but it is currently still required.
Usually taken from the configuration.


#### scores: \[int\]

A list of scores corresponding to each solution.
Should be updated in each call to the step method.


##### Score strategy

How to score a tournament: tournaments should use a 70/30/0 scoring system for three participants.
This is to avoid the issue that otherwise the ELO doesn't change if the scores are very close.


### The configurations

The challenge usually has a `_configuration` list containing a dictionary for each level.
This dictionary can be used to store the level configuration (constants, initial values, flags, etc) and usually contain the following items.


#### name (str)

The level name (used to set `level_name`).


#### show_canvas (bool)

A flag indicating if the canvas should be rendered or not.
The canvas should always be rendered for public levels, and preferably also for hidden levels.
The flag is often False for performance levels in code tests due to the large input size.


#### max_score (int)

The maximum score for the level (used to set `max_score`).


## Logging and printing

### Print from the solution

To print from the solution to the console just use the regular print statement used by your language.
In python3 it would be `print('hello world')`.


### Print from the implementation

To print from the implementation just use the regular print statement for Python3.


## Example implementations

The example implementations are functionally very simple and are only intended to demonstrate some different concepts that can be used when writing an implementation.


### Exercise example

In this implementation the candidate should compute the sum and difference of two numbers.
The candidate methods are `sum() -> int` and `difference() -> int`.
The API methods are `get_a() -> int` and `get_b() -> int`.

???+ example "Exercise example"
    ```
    """
    This is the module containing the challenge implementation.
    """
    from lib.exceptions import SolutionException

    class Challenge:
        """
        A new challenge object is created when a challenge level is run.

        :var finished: Whether the level run is finished or not.
        :vartype finished: bool
        :var level_name: The name of the current level.
        :vartype level_name: str
        :var max_score: The highest possible score for the level.
        :vartype max_score: int
        :var scores: The scores for all solutions.
        :vartype scores: [int]
        """

        def __init__(self, context):
            """
            :param contex: An object containing relevant information for this run.
                Read the provided challenge creation documentation for more details.
            """
            self._context = context

            # At the end of this template there is a list of level configurations.
            # Here we check the level value and look up the current configuration.
            if not 0 <= self._context.level <= len(self._configurations):
                raise Exception(f'There is no level {self._context.level}')
            self._config = self._configurations[self._context.level]

            # These attibutes are used by the challenge runner (do not remove).
            self.finished = False
            self.level_name = self._config['name']
            self.max_score = self._config['max_score']
            self.scores = [0] * len(self._context.solutions)
            
            # Setup initial values, should assume failure.
            self._sum_answer = None
            self._sum_correct = False
            self._diff_answer = None
            self._diff_correct = False

            # Setup the canvas.
            self._show_canvas = self._config['show_canvas']
            if self._show_canvas:
                self._setup_canvas()

        def step(self):
            """
            Executes a single step of the challenge.
            This includes calling solutions, updating the scores, and updating the canvas.

            This method is continously called until either "finished" is set to "True"
            or all solutions have crashed.
            """
            # Use this method to gracefully handle crashed solutions.
            for solution in self._context.living_solutions():
                try:
                    # Do not call the solution methods more than once.
                    self._sum_answer = solution.sum()
                    if self._sum_answer == self._config['sum']:
                        self._sum_correct = True
                        self.scores[solution.index] += 4
                    # Fully handle each task in turn, in case of crashes.
                    self._diff_answer = solution.difference()
                    if self._diff_answer == self._config['diff']:
                        self._diff_correct = True
                        self.scores[solution.index] += 6
                except SolutionException:
                    # A SolutionException is the only exception that should be caughthen
                    # calling a soluti and it must be caught when calling a solution.
                    pass

            # Update the canvas.
            if self._show_canvas:
                self._update_canvas()

            # A code test usually only has one step.
            self.finished = True

        def get_a(self, solution):
            """
            :param solution: The solution that called this api method.
            
            :rtype: int
            """
            return self._config['a']

        def get_b(self, solution):
            """
            :param solution: The solution that called this api method.
            
            :rtype: int
            """
            return self._config['b']

        def _setup_canvas(self):
            """
            Performs canvas setup before a solution is run.
            Read the provided documentation for details on how to use the canvas.
            """
            canvas = self._context.canvas
            # Show input data
            canvas.new_text(f'Input: {self._config["a"]} and {self._config["b"]}', 20, 'black',tx= y=20) 
            # Show task titles and expected answers.
            canvas.new_text('Task 1: sum', 20, 'black', x=20, y=60)
            canvas.new_text('Task 2: difference', 20, 'black', x=200, y=60)
            canvas.new_text(f'Expected: {self._config["sum"]}', 16, 'black', x=20, y=90)
            canvas.new_text(f'Expected: {self._config["diff"]}', 16, 'black', x=200, y=90)
            # Add sections for candidate answers, to later be updated.
            self._sum_label = canvas.new_text('Actual: ___', 16, 'black', x=20, y=120)
            self._diff_label = canvas.new_text('Actual: ___', 16, 'black', x=200, y=120)

        def _update_canvas(self):
            """
            Updates the canvas for each challenge step.
            Read the provided documentation for details on how to use the canvas.
            """
            canvas = self._context.canvas
            # Update the canvas with candidate answers.
            if self._sum_answer is None:
                self._sum_label.set_text('Actual: N/A')
            else:
                self._sum_label.set_text(f'Actual: {self._sum_answer}')
            if self._diff_answer is None:
                self._diff_label.set_text('Actual: N/A')
            else:
                self._diff_label.set_text(f'Actual: {self._diff_answer}')
            # Add correct/incorrect labels.
            if self._sum_correct:
                canvas.new_text('Correct', 16, 'green', x=20, y=150)
            else:
                canvas.new_text('Incorrect', 16, 'red', x=20, y=150)
            if self._diff_correct:
                canvas.new_text('Correct', 16, 'green', x=200, y=150)
            else:
                canvas.new_text('Incorrect', 16, 'red', x=200, y=150)

        _configurations = [
            {
                # Level 1 (Public)
                'name': 'Level 1',
                'show_canvas': True,
                'max_score': 10,
                'a': 7,
                'b': 3,
                'sum': 10,
                'diff': 4,
            },
            {
                # Level 2 (Public)
                'name': 'Level 2',
                'show_canvas': True,
                'max_score': 10,
                'a': 16,
                'b': 24,
                'sum': 40,
                'diff': 8,
            },
        ]
    ```


### Challenge example

In this challenge all players get to guess on a number.
The number should be just above the mean of all the guesses to get maximum points, if it is on or below they get no points.
This continues for a number of rounds.

The candidate methods are `guess_mean() -> int`.
The API methods are `last_mean() -> int`.

???+ example "Challenge example"
    ```
    """
    This is the module containing the challenge implementation.
    """
    from lib.exceptions import SolutionException
    from statistics import mean

    class Challenge:
        """
        A new challenge object is created when a challenge level is run.

        :var finished: Whether the level run is finished or not.
        :vartype finished: bool
        :var level_name: The name of the current level.
        :vartype level_name: str
        :var max_score: The highest possible score for the level.
        :vartype max_score: int
        :var scores: The scores for all solutions.
        :vartype scores: [int]
        """

        def __init__(self, context):
            """
            :param contex: An object containing relevant information for this run.
                Read the provided challenge creation documentation for more details.
            """
            self._context = context

            # At the end of this template there is a list of level configurations.
            # Here we check the level value and look up the current configuration.
            if not 0 <= self._context.level <= len(self._configurations):
                raise Exception(f'There is no level {self._context.level}')
            self._config = self._configurations[self._context.level]

            # These attibutes are used by the challenge runner (do not remove).
            self.finished = False
            self.level_name = self._config['name']
            self.max_score = self._config['max_score']
            self.scores = [0] * len(self._context.solutions)

            # Setup variable used for the game.
            self._guesses = {}
            self._gains = {}
            self._last_mean = None
            self._rounds = self._config['rounds']
            self._current_round = 0

            # Setup the canvas.
            self._show_canvas = self._config['show_canvas']
            if self._show_canvas:
                self._setup_canvas()

        def step(self):
            """
            Executes a single step of the challenge.
            This includes calling solutions, updating the scores, and updating the canvas.

            This method is continously called until either "finished" is set to "True"
            or all solutions have crashed.
            """
            # Setup the new round.
            self._guesses = {}
            self._gains = {}
            self._current_round += 1
            self._context.console.debug(f'Starting round {self._current_round}')
            # Go through all solutions that still are running.
            for solution in self._context.living_solutions():
                try:
                    # Collect the guesses from each solution.
                    self._guesses[solution.index] = solution.guess_mean()
                    self._context.console.debug(f'Solution {solution.index} guessed {self._gues[solution.index]}')`  
                except SolutionException:
                    # A SolutionException is the only exception that should be caught whe
                    # calling a solution, and it must be caught when calling a solution.
                    pass
            
            # Calculate new points (if any results where obtained).
            if self._guesses:
                # Calculate the mean.
                self._last_mean = round(mean(self._guesses.values()))
                self._context.console.debug(f'Mean is {self._last_mean}')
                # Award points.
                for index, guess in self._guesses.items():
                    if self._last_mean < guess < self._last_mean + 100:
                        self._gains[index] = 100 - (guess - self._last_mean)
                        self.scores[index] += self._gains[index]
                self._context.console.debug(f'New scores are {self.scores}')
            else:
                self._last_mean = None

            # Update the canvas.
            if self._show_canvas:
                self._update_canvas()

            # Limit the number of steps to the specified number of rounds.
            if self._current_round >= self._rounds:
                self.finished = True

        def last_mean(self, solution):
            """
            :param solution: The solution that called this api method.
            
            :rtype: int
            """
            solution.console.debug(f'Solution {solution.index} called "last_mean"')
            # Disallow calling 'last_mean' until after the first round.
            if self._last_mean is None:
                raise SolutionException('"last_mean" may only be called after the first round'
            return self._last_mean

        def _setup_canvas(self):
            """
            Performs canvas setup before a solution is run.
            Read the provided documentation for details on how to use the canvas.
            """
            canvas = self._context.canvas
            # Create the player table.
            canvas.new_text('Player', 14, 'black', x=10, y=10)
            canvas.new_text('Guess', 14, 'black', x=200, y=10)
            canvas.new_text('Gain', 14, 'black', x=275, y=10)
            canvas.new_text('Score', 14, 'black', x=350, y=10)
            self._guess_labels = []
            self._gain_labels = []
            self._score_labels = []
            for index, solution in enumerate(self._context.solutions):
                # Need to check if solution exists, otherwise use placeholder name.
                name = solution.name if solution else f'Player {index}'
                canvas.new_text(name, 12, 'black', x=10, y=20*index+30)
                label = canvas.new_text('N/A', 12, 'black', x=200, y=20*index+30)
                self._guess_labels.append(label)
                label = canvas.new_text(0, 12, 'black', x=275, y=20*index+30)
                self._gain_labels.append(label)
                label = canvas.new_text(0, 12, 'black', x=350, y=20*index+30)
                self._score_labels.append(label)
            # Create entry for the mean.
            canvas.new_text('Mean', 14, 'black', x=10, y=180)
            self._mean_label = canvas.new_text('N/A', 14, 'black', x=200, y=180)

        def _update_canvas(self):
            """
            Updates the canvas for each challenge step.
            Read the provided documentation for details on how to use the canvas.
            """
            canvas = self._context.canvas
            # Update player table.
            for index in range(len(self._context.solutions)):
                self._guess_labels[index].set_text(self._guesses.get(index, 'N/A'))
                self._gain_labels[index].set_text(self._gains.get(index, 0))
                self._score_labels[index].set_text(self.scores[index])
            # Update the mean.
            self._mean_label.set_text('N/A' if self._last_mean is None else self._last_mean)

        _configurations = [
            {
                # Level 1 (Public)
                'name': 'Level 1',
                'show_canvas': True,
                'max_score': 10,
                'rounds': 5,
            },
            {
                # Level 2 (Public)
                'name': 'Level 2',
                'show_canvas': True,
                'max_score': 10,
                'rounds': 10,
            },
        ]
    ```


### Parallel Challenge example

This is the same challenge as the previous example, but it calls the solutions in parallel.
???+ example "Parallel Challenge example"
    ```
    """
    This is the module containing the challenge implementation.
    """
    from lib.exceptions import SolutionException
    from statistics import mean

    class Challenge:
        """
        A new challenge object is created when a challenge level is run.

        :var finished: Whether the level run is finished or not.
        :vartype finished: bool
        :var level_name: The name of the current level.
        :vartype level_name: str
        :var max_score: The highest possible score for the level.
        :vartype max_score: int
        :var scores: The scores for all solutions.
        :vartype scores: [int]
        """

        def __init__(self, context):
            """
            :param contex: An object containing relevant information for this run.
                Read the provided challenge creation documentation for more details.
            """
            self._context = context

            # At the end of this template there is a list of level configurations.
            # Here we check the level value and look up the current configuration.
            if not 0 <= self._context.level <= len(self._configurations):
                raise Exception(f'There is no level {self._context.level}')
            self._config = self._configurations[self._context.level]

            # These attibutes are used by the challenge runner (do not remove).
            self.finished = False
            self.level_name = self._config['name']
            self.max_score = self._config['max_score']
            self.scores = [0] * len(self._context.solutions)

            # Setup variable used for the game.
            self._guesses = {}
            self._gains = {}
            self._last_mean = None
            self._rounds = self._config['rounds']
            self._current_round = 0

            # Setup the canvas.
            self._show_canvas = self._config['show_canvas']
            if self._show_canvas:
                self._setup_canvas()

        def step(self):
            """
            Executes a single step of the challenge.
            This includes calling solutions, updating the scores, and updating the canvas.

            This method is continously called until either "finished" is set to "True" 
            or all solutions have crashed.
            """
            # Setup the new round.
            self._guesses = {}
            self._gains = {}
            self._current_round += 1
            self._context.console.debug(f'Starting round {self._current_round}')
            # Go through all solutions that still are running, in parallel.
            for index, result in self._context.parallel.guess_mean().items():
                # Skip solutions that caused an exception.
                if isinstance(result, SolutionException):
                    continue
                # Collect the guess.
                self._guesses[index] = result
                self._context.console.debug(f'Solution {index} guessed {result}')
            
            # Calculate new points (if any results where obtained).
            if self._guesses:
                # Calculate the mean.
                self._last_mean = round(mean(self._guesses.values()))
                self._context.console.debug(f'Mean is {self._last_mean}')
                # Award points.
                for index, guess in self._guesses.items():
                    if self._last_mean < guess < self._last_mean + 100:
                        self._gains[index] = 100 - (guess - self._last_mean)
                        self.scores[index] += self._gains[index]
                self._context.console.debug(f'New scores are {self.scores}')
            else:
                self._last_mean = None

            # Update the canvas.
            if self._show_canvas:
                self._update_canvas()

            # Limit the number of steps to the specified number of rounds.
            if self._current_round >= self._rounds:
                self.finished = True

        def last_mean(self, solution):
            """
            :param solution: The solution that called this api method.
            
            :rtype: int
            """
            solution.console.debug(f'Solution {solution.index} called "last_mean"')
            # Disallow calling 'last_mean' until after the first round.
            if self._last_mean is None:
                raise SolutionException('"last_mean" may only be called after the first round')
            return self._last_mean

        def _setup_canvas(self):
            """
            Performs canvas setup before a solution is run.
            Read the provided documentation for details on how to use the canvas.
            """
            canvas = self._context.canvas
            # Create the player table.
            canvas.new_text('Player', 14, 'black', x=10, y=10)
            canvas.new_text('Guess', 14, 'black', x=200, y=10)
            canvas.new_text('Gain', 14, 'black', x=275, y=10)
            canvas.new_text('Score', 14, 'black', x=350, y=10)
            self._guess_labels = []
            self._gain_labels = []
            self._score_labels = []
            for index, solution in enumerate(self._context.solutions):
                # Need to check if solution exists, otherwise use placeholder name.
                name = solution.name if solution else f'Player {index}'
                canvas.new_text(name, 12, 'black', x=10, y=20*index+30)
                label = canvas.new_text('N/A', 12, 'black', x=200, y=20*index+30)
                self._guess_labels.append(label)
                label = canvas.new_text(0, 12, 'black', x=275, y=20*index+30)
                self._gain_labels.append(label)
                label = canvas.new_text(0, 12, 'black', x=350, y=20*index+30)
                self._score_labels.append(label)
            # Create entry for the mean.
            canvas.new_text('Mean', 14, 'black', x=10, y=180)
            self._mean_label = canvas.new_text('N/A', 14, 'black', x=200, y=180)

        def _update_canvas(self):
            """
            Updates the canvas for each challenge step.
            Read the provided documentation for details on how to use the canvas.
            """
            canvas = self._context.canvas
            # Update player table.
            for index in range(len(self._context.solutions)):
                self._guess_labels[index].set_text(self._guesses.get(index, 'N/A'))
                self._gain_labels[index].set_text(self._gains.get(index, 0))
                self._score_labels[index].set_text(self.scores[index])
            # Update the mean.
            self._mean_label.set_text('N/A' if self._last_mean is None else self._last_mean)

        _configurations = [
            {
                # Level 1 (Public)
                'name': 'Level 1',
                'show_canvas': True,
                'max_score': 10,
                'rounds': 5,
            },
            {
                # Level 2 (Public)
                'name': 'Level 2',
                'show_canvas': True,
                'max_score': 10,
                'rounds': 10,
            },
        ]
    ```
