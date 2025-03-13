---
title: Skeletons
contributors:
  - Henrik Rostedt
  - Henrik
---

This article goes through how to use the skeleton classes to write implementations.
This is the preferred way of creating new exercises, challenges, etc.

The article contains both a broad overview of the material as well as deep dives, it is recommended to start reading the broad sections and come back to the sub-sections as questions arise.


## Why and how to use

The skeletons are base classes that handles things that most implementations will need or things that are required.
The specialized skeletons also provides a bunch of functionality that otherwise would take a lot of work to implement.

The provided skeletons form a hierarchy where the children inherit functionality from the parent skeletons:

- `BasicChallenge` - Provides the most common functionality
    - `StageChallenge` - Provides a GUI structure and a couple other features
        - `GameChallenge` - Provides a bunch of features realated to making a turn-based game

To use a skeleton the `Challenge` class should inherit from the skeleton class.
Each skeleton has a number of required methods that need to be implemented.
We provide a basic template for each skeleton under respective heading, which are good starting points for creating a challange.


## `BasicChallenge`

This is the base of all the skeletons and provide the essential functionality, which will be used by all other skeletons as well.
However, keep in mind that some skeletons do implement some of the `BasicChallenge` methods for you.

??? example "Template for `BasicChallange`"
    ```
    """
    This is the module containing the challenge implementation.
    """
    from lib.exceptions import SolutionException
    from lib.skeletons import BasicChallenge, LevelInfo
    from lib.ui import Rectangle, Text


    class Challenge(BasicChallenge):
        """
        This is a challenge implementation using the basic skeleton.
        """

        def setup_level(self, level):
            """
            This method returns required level information for the current level.
            The "level" parameter is the level index, starting at 0.
            """
            return LevelInfo(name=f"Level {level + 1}", max_score=10)

        def setup_state(self):
            """
            This method is where you should do all initial setup, except for graphics.
            """
            # For example, setting up a variable which can be used later on
            self.my_variable = 5
            # Or writing a message in the console
            self.console.log("Hi console!")

        def setup_canvas(self):
            """
            This method is where you should create the initial graphics.
            """
            # For example, creating a square
            self.my_square = Rectangle(w=20, h=20, x=20, y=20, color="green", parent=self.canvas)
            # And putting text inside it
            self.my_text = Text("N", font_size=10, color="white", parent=self.my_square)

        def update_state(self):
            """
            This method is where you should call solutions and update the current state.
            It is called continuously until "self.finished" is set to "True".
            """
            # When calling a solution you need to handle any "SolutionException"
            try:
                solution = self.context.solutions[0]
                # TODO: call a solution method
            except SolutionException:
                # Code put here will run if the solution crashed
                pass

            # This is a good place to update the scores
            self.scores[0] = self.my_variable

            # You will want to set this conditionally if your challenge involves multiple step
            self.finished = True

        def update_canvas(self):
            """
            This method is where you should update the graphics based on the current state.
            """
            # For example, changing the text inside the square
            self.my_text.text = str(self.my_variable)
    ```


### Required methods

`setup_level(self, level)`  
: Must return a `LevelInfo` object which includes the information needed for the framework to handle the level.
: The `level` parameter is the index of the current level, starting at 0.
: The `LevelInfo` object has the following attributes:
: - `name` - The name of the level
: - `max_score` - The maximum possible score for a solution
: - `show_canvas` - Whether the level uses the canvas, defaults to `True`

`setup_state(self)`  
: Setup the challenge state here.
: Will be called once at startup, after `setup_level`.

`setup_canvas(self)`  
: Setup the canvas here.
: Will be called once at startup, after `setup_state`, unless `show_canvas` is `False`.
: !!! note 
        This method is implemented by most of the other skeletons, so do not add your own when using those.

`update_state(self)`  
: Call solutions and update the challenge state here.
: When calling solutions you must catch any `SolutionException`s that are raised.
: Will continuously be called until `self.finished` is set to `True`, so remember to do that.
: !!! note 
        Once `self.finished` is set to `True` no other method is called, so remember to also update the scores.

`update_canvas(self)`  
: Make any changes to the canvas here.
: Will be called immediately after `update_state`, unless `show_canvas` is `False`.
: !!! note
        This method is implemented by most of the other skeletons, so do not add your own when using those.

??? info "Flowchart for `BasicChallenge`"
    ``` mermaid
    ---
    config:
        look: handDrawn
    ---
    graph TB
        START([START]) --> A[setup_level];
        A --> B[setup_state];
        B --> IF_1{show_canvas};
        IF_1 --> |True| C[setup_canvas];
        C --> D[update_state];
        IF_1 --> |False| D;
        D --> IF_2{self.finished};
        IF_2 ---> |True| END([END]);
        IF_2 --> |False| IF_3{show_canvas};
        IF_3 --> |True| E[update_canvas];
        E --> D;
        IF_3 --> |False| D;
    ```


### Features

The main feature of this skeleton is reduced boilerplate and some minor conveniences.
Here is a list of class members and what they are used for:

`finished`  
: A boolean that should be set to `True` when the challenge has ended.

`scores`  
: A list of scores for each solution, indices match the solution indices.
: Scores start at 0 and must be updated manually.

`canvas`  
: Represents the canvas itself and is the top of the graphical hierarchy.

`console`  
: Represents the console, and can be used to log messages.

`context`  
: Used to access various features:
: `level`
    : The current level
: `canvas`
    : An object that is used to interact with the canvas
    It is used to add new graphical elements, more details on them are covered in [UI](UI.md).
    It is best practice to only modify the `canvas` in dedicated setup and update methods, by default `setup/update_canvas/view`
: `console`
    : An object that can be used to interact with the console.
    It is no longer needed, as the standard `print` statement can be used instead.
: `solutions`
    : A list of the `solution` objects corresponding to the coders partaking in the challenge/exercise/freecode together for this level.
    This list will always be the same length as the number of solutions, but all values will be set to `None` during `setup_canvas/view`, the proper values will be found in the `update_` methods.
    The `solution` objects contain the following:
    : `name`
        : The name of the solution, this will either be the player name or a placeholder string.
    : `index`
        : The index of the solution.
    : `console`
        : An individual console for the solution.
        Use the `log` and `debug` methods to print messages in the corresponding player's console.
    : `alive`
        : A flag indicating if the solution is still running or has crashed/timed out.
: `parallel`
    : An object that acts as a proxy `solution` for making parallel `solution` calls.
    `parallel` will be explained further in [Making parallel `solution` calls](Skeletons.md#making-parallel-solution-calls).

`session`  
: Used for interact mode, see [Interact mode](Skeletons.md#interact-mode).

`settings`  
: Used for interact mode, see [Interact mode](Skeletons.md#interact-mode).


#### Making parallel `solution` calls

To call solutions in parallel the `context.parallel` object should be used instead of the `solution` objects.

The results of a parallel call is a dictionary with `solution` indices as kets and the `solution` results as value.
There is no need to catch `SolutionException` when making parallel calls.
Only `solutions` that lived before making the call are included in the dictionary.
The solution result is either the return value from the player's code, or a `SolutionException` if the player's code crashed, therefore it is necessary to make an `isinstance` check on the result if the return value is used.


#### Interact mode

Interact mode is an advanced feature that is used to make challenges with interactive canvases.
This skeleton has a couple of conveniences related to interact mode.

The `session` class member is used to store the state of the current interact session, which can be used to restart the session in case it timed out or was paused.
It acts as a dictionary and the values must consist of only basic python types.
The exact data representation is not guaranteed to be retained, and notably any integer dictionary keys will be converted into strings.

The `settings` class member act just like `session` but is persisted over interact sessions.
So you can use it to store user settings or whether to show a help popup.

Both of these support callbacks on changes using `.register_callback(id, key, callback)` where `id` should be unique, `key` is the settings key, and `callback` should be a function that takes one argument (the new value).
A callback can be removed using `.deregister_callback(id, key)`.

!!! note
    Some of the other skeletons might use these features in order to persist settings or state.
    Therefore you should expect the possibility of unknown keys and values.
    However, those keys will always be prefixed with two "_" to avoid clashes.

These features use the `interact_session_state` and `interact_setting_state` to store the values, so make sure that you are not using those directly as well.


## `StageChallenge`

This skeleton adds a GUI setup called the stage on top of the basic skeleton, and some related functionality.
It acts as the base for other GUI focused skeletons, so it is likely you will want to use one of them instead.

??? example "Template for `StageChallenge`"
    ```
    """
    This is the module containing the challenge implementation.
    """
    from lib.exceptions import SolutionException
    from lib.skeletons import LevelInfo, StageChallenge
    from lib.ui import Rectangle, Text


    class Challenge(StageChallenge):
        """
        This is a challenge implementation using the stage skeleton.
        """

        def setup_level(self, level):
            """
            This method returns required level information for the current level.
            The "level" parameter is the level index, starting at 0.
            """
            return LevelInfo(name=f"Level {level + 1}", max_score=10)

        def setup_state(self):
            """
            This method is where you should do all initial setup, except for graphics.
            """
            # For example, setting up a variable which can be used later on
            self.my_variable = 5
            # Or writing a message in the console
            self.console.log("Hi console!")
            # Or writing a messege in the stage log
            self.add_log_entry("Hi log!")

        def setup_view(self):
            """
            This method returns the main view for the challenge, which can be any graphics element.
            """
            # The view will automatically be attached, resized and positioned by the framework
            self.my_view = Rectangle(color="green")
            # Any elements put inside the view will be centered inside they view
            self.my_text = Text("N", font_size=50, color="white", parent=self.my_view)
            return self.my_view

        def update_state(self):
            """
            This method is where you should call solutions and update the current state.
            It is called continuously until "self.finished" is set to "True".
            """
            # When calling a solution you need to handle any "SolutionException"
            try:
                solution = self.context.solutions[0]
                # TODO: call a solution method
            except SolutionException:
                # Code put here will run if the solution crashed
                pass

            # This is a good place to update the scores
            self.scores[0] = self.my_variable

            # You will want to set this conditionally if your challenge involves multiple steps
            self.finished = True

        def update_view(self):
            """
            This method is where you should update the view based on the current state.
            """
            # For example, changing the text inside the view
            self.my_text.text = str(self.my_variable)
    ```


### Required methods { #stagechallenge-required-methods }

`setup_level(self, level)`  
: See [`BasicChallenge` - Required methods](Skeletons.md#required-methods).

`setup_state(self)`  
: See [`BasicChallenge` - Required methods](Skeletons.md#required-methods).

`setup_view(self)`  
: Must return a graphics element which will comprise the main view of the challenge.
: It is automatically given a parent, size and position on the canvas.
: Will be called once at startup, after `setup_state`, unless `show_canvas` is `False`.

`update_state(self)`  
: See [`BasicChallenge` - Required methods](Skeletons.md#required-methods).

`update_view(self)`  
: Make any changes to the view here.
: Will be called immediately after `update_state`, unless `show_canvas` is `False`.


### Optional methods

`setup_info_panel(self)`  
: Similar to `setup_view` but should return an element to put on the left-hand side of the main view.
: There is no equivalent to `update_view`, so any needed updates should be put there.

`setup_description(self)`  
: Should return a text string that will be put in a "Description" tab.
This text can contain html formatting, including images.

`setup_settings(self)`  
: Used for interact mode, see [Interact mode - Settings](Skeletons.md#settings).

`setup_center_buttons(self)`  
: Used for interact mode, see [Interact mode - Toolbar](Skeletons.md#toolbar).

`setup_right_buttons(self)`  
: Used for interact mode, see [Interact mode - Toolbar](Skeletons.md#toolbar).

??? info "Flowchart for `StageChallenge`"
    ``` mermaid
    ---
    config:
        look: handDrawn
    ---
    graph TB
        START([START]) --> A[setup_level];
        A --> B[setup_state];
        B --> IF_1{show_canvas};
        IF_1 --> |True| C[setup_view];
        C --> SA[setup_info_panel];
        subgraph NEW
        SA --> SB[setup_description];
        SB --> SC[setup_center_buttons];
        SC --> SD[setup_right_buttons];
        end
        SD --> D[update_state];
        IF_1 --> |False| D;
        D --> IF_2{self.finished};
        IF_2 ---> |True| END([END]);
        IF_2 --> |False| IF_3{show_canvas};
        IF_3 --> |True| E[update_view];
        E --> D;
        IF_3 --> |False| D;
    ```


### Features

The `StageChallenge` class has the same features as the `BasicChallenge` in addition to some new features.

The main additional feature of this skeleton is the GUI setup with a main view and various optional features.
Many features are related to interact mode, and are covered in their own subsection.

Additional features include:

- Info panel on the left-hand side of the main view, see [Optional methods](Skeletons.md#optional-methods).
- A GUI log which can be written to using `self.add_log_entry`.
- A description tab with support for html formatting, see [Optional methods](Skeletons.md#optional-methods).
- A GUI settings system, see [Interact mode - Settings](Skeletons.md#settings).
- A toolbar at the bottom of the canvas, see [Interact mode - Toolbar](Skeletons.md#toolbar).


#### Interact mode

This skeleton provides several useful features for interact mode, some that work on top of the basic skeleton features.


##### Settings

This skeleton extends the settings system from the basic skeleton with GUI checkboxes in a new "Settings" tab to the right.

To add a setting here you need to add a `SettingInfo` object for it in the list returned by `setup_settings`.
The `SettingInfo` object has the following attributes:

- `key` - The key for the setting (same as used in basic skeleton)
- `label` - Text next to the checkbox
- `category` - Settings category, used for grouping settings
- `default` - The default value (defaults to `False`)

??? example "Example of setting up and using settings"
    ```
    def setup_settings(self):
        return [SettingInfo(key="my_setting", label="My Setting", category="Example")]

    def update_view(self):
        if self.settings.get("my_setting"):
            self.my_text.text = str(self.my_variable)
        else:
            self.my_text.text = "?"
    ```


##### Toolbar

When in interact mode a toolbar will appear at the bottom of the stage.
This toolbar comes with an info prompt as well as the possibility to add buttons in the center and to the right.

The info prompt can be updated by setting the `self.info_prompt` attribute (which defaults to empty).

Buttons can be added by returning a list of buttons in either `setup_center_buttons` or `setup_right_buttons`. 
These buttons will be sized and positioned appropriately, but you will have to attach behaviour yourself.

??? example "Example of setting up buttons and using the info prompt"
    ```
    def setup_center_buttons(self):
        return [Button("Center", on_click=lambda _: setattr(self, "info_prompt", "Center clicked"))]

    def setup_right_buttons(self):
        return [Button("Right", on_click=lambda _: setattr(self, "info_prompt", "Right clicked"))]

    def setup_state(self):
        self.info_prompt = "Click a button"
    ```


## `GameChallenge`

This skeleton adds a bunch of nice features useful when creating games (or other challenges where solutions take turns).
It makes extensive use of the stage skeleton features, so only implement methods also mentioned here.
Like the stage skeleton, there are plenty of features specifically for interact mode.

??? example "Template for `GameChallenge`"
    ```
    """
    This is the module containing the challenge implementation.
    """
    from lib.exceptions import SolutionException
    from lib.skeletons import LevelInfo, GameChallenge, GameInfo, PlayerInfo
    from lib.ui import Rectangle, Text


    class Challenge(GameChallenge):
        """
        This is a challenge implementation using the game skeleton.
        """
        
        def setup_game(self):
            """
            This method returns required game information for setting up the challenge.
            """
            return GameInfo(
                title="My game",
                summary="My game summary",
                description="My game description. Which can use <em>html formatting</em>!",
            )
        
        def setup_players(self):
            """
            This method returns a list with required information for each player.
            """
            return [PlayerInfo(
                role="Player",
                name="Your solution",
                image="ui_general_null.webp",
            )]

        def setup_level(self, level):
            """
            This method returns required level information for the current level.
            The "level" parameter is the level index, starting at 0.
            """
            return LevelInfo(name=f"Level {level + 1}", max_score=10)

        def setup_state(self):
            """
            This method is where you should do all initial setup, except for graphics.
            """
            # For example, setting up a variable which can be used later on
            self.my_variable = 5
            # Or writing a message in the console
            self.console.log("Hi console!")
            # Or writing a messege in the stage log
            self.add_log_entry("Hi log!")
            # Or writing a log message associated with a player (use player index starting at 0)
            self.add_player_log_entry(0, "Hi player!")

        def setup_view(self):
            """
            This method returns the main view for the challenge, which can be any graphics element.
            """
            # The view will automatically be attached, resized and positioned by the framework
            self.my_view = Rectangle(color="green")
            # Any elements put inside the view will be centered inside they view
            self.my_text = Text("N", font_size=50, color="white", parent=self.my_view)
            return self.my_view

        def update_state(self):
            """
            This method is where you should call solutions and update the current state.
            It is called continuously until "self.finished" is set to "True".
            """
            # When calling a solution you need to handle any "SolutionException"
            try:
                solution = self.context.solutions[0]
                # TODO: call a solution method
            except SolutionException:
                # Code put here will run if the solution crashed
                pass

            # This is a good place to update the scores
            self.scores[0] = self.my_variable

            # You will want to set this conditionally if your challenge involves multiple steps
            self.finished = True

        def update_view(self):
            """
            This method is where you should update the view based on the current state.
            """
            # For example, changing the text inside the view
            self.my_text.text = str(self.my_variable)
    ```


### Required methods

`setup_game(self)`  
: Must return a `GameInfo` object which includes the information needed for the framework to handle the game.
: It has the following attributes (all optional):
: - `title` - The title of the game
: - `summary` - A short summary of the game
: - `description` - A longer description of the game, which can include html formatting including images
: - `step_mode` - See dedicated subsection of the features section.

`setup_players(self)`  
: Must return a list of `PlayerInfo` objects including the information needed for the framework to handle players.
: Typically the player list will correspond with the solution list, but there might be reasons to have more or fewer players in a challenge.
: The `PlayerInfo` object has the following attributes:
: - `role` - The role of the player, will be displayed above the name
: - `name` - The name of the player
: - `image` - The name of the image to use for the player, will be shown in the player list and the log

`setup_level(self, level)`  
: See [`BasicChallenge` - Required methods](Skeletons.md#required-methods).

`setup_state(self)`  
: See [`BasicChallenge` - Required methods](Skeletons.md#required-methods).

`setup_view(self)`  
: See [`StageChallenge` - Required methods](Skeletons.md#stagechallenge-required-methods).

`update_state(self)`  
: See [`BasicChallenge` - Required methods](Skeletons.md#required-methods).

`update_view(self)`  
: See [`StageChallenge` - Required methods](Skeletons.md#stagechallenge-required-methods).


### Optional methods

`setup_statistics(self)`  
: See [Statistics](Skeletons.md#statistics).

`setup_settings(self)`  
: See [`StageChallenge` - Optional methods](Skeletons.md#optional-methods).
: !!! note
        You need to include the settings from `super().setup_settings()`.

`setup_center_buttons(self)`  
: See [`StageChallenge` - Optional methods](Skeletons.md#optional-methods).

`step_mode_check(self)`  
: See [Interact mode - Turn management](Skeletons.md#turn-management).

`resign(self)`  
: Used for interact mode, see [Interact mode - Toolbar](Skeletons.md#gamechallenge-toolbar).

??? info "Flowchart for `GameChallenge`"
    ``` mermaid
    ---
    config:
        look: handDrawn
    ---
    graph TB
        START([START]) --> GA[setup_game];
        subgraph NEW
        GA --> GB[setup_players];
        GB --> GC[setup_statistics];
        end
        GC --> A[setup_level];
        A --> B[setup_state];
        B --> IF_1{show_canvas};
        IF_1 --> |True| C[setup_view];
        C --> SA[setup_info_panel];
        SA --> SB[setup_description];
        SB --> SC[setup_center_buttons];
        SC --> SD[setup_right_buttons];
        SD --> D[update_state];
        IF_1 --> |False| D;
        D --> IF_2{self.finished};
        IF_2 ---> |True| END([END]);
        IF_2 --> |False| IF_3{show_canvas};
        IF_3 --> |True| E[update_view];
        E --> D;
        IF_3 --> |False| D;
    ```


### Features

This skeleton adds a number of GUI features on top of the stage UI from the `StageChallenge` skeleton, as well as some core features for creating games.
Most features are detailed in their own subsection.


#### Players

The main addition of this skeleton is the concept of players, which usually correspond to the solutions.
The information provided in `setup_players` will be displayed in the new left-hand side information panel, together with a game summary.

There might be reasons to have a different number of players than solutions, but normally only in interact mode.


#### Statistics

The skeleton includes a system for tracking statistics for each player during a game.
These statistics will be shown in the player panel under the players name.

To setup a statistic you will include a `StatisticInfo` object in `setup_statistics` with the following attributes:

- `name` - The name of the statistic (required), should be a valid python identifier
- `suffix` - A suffix to append in the player panel (defaults to empty)
- `type` - The type of the statistic (defaults to `float`), use `int` for integers or `str` for strings
- `default` - The initial value for the statistic (defaults to 0)

To get or set a statistic for a player you can use `self.players[i].name` where `i` is the player index and `name` is the statistic.
For example, setting the "points" statistic for the first player: `self.players[0].points = 42`.


#### Interact mode

This skeleton adds a complete turn-management system which greatly simplifies interaction for games.


##### Toolbar {:#gamechallenge-toolbar}

This skeleton utilizes the toolbar introduced with the stage skeleton by adding buttons on the right-hand side.
You should add any custom buttons to the center of the toolbar.

The added button are "Resign" (with a confirmation popup) and "Finish turn" (see following sections).
The `resign(self)` method can be overridden to implement specific behaviour for resignation.


##### Turn management

A main feature of this skeleton is the turn management system which makes it easier to handle user input and turn logic.
This consists of a few attributes, methods and GUI elements.

When using this system there are two things to remember in a user input callback:

1.  Make sure that `self.turn_complete` has the correct value (should be `True` if the turn can be finished)
2.  Make sure to end the callback by calling `self.finish_input()`.

This ensures that the "Finish turn" button is enabled when it should be and that the auto-complete turn feature will work correctly.

This skeleton comes with a pre-made setting called "Auto-finish turn" which will automatically finish the turn once it is complete, otherwise the user needs to press the "Finish turn" button manually.

When the turn is finished (manually or automatically) the system will perform a number of "steps" according to the `step_mode` (see `GameInfo`).
Each step consists of a call to `update_state` and `update_view` (if `show_canvas` is `True`).
The following step modes exists:

`StepMode.ROUND`  
: Will step exactly once.
: Use if your update method always goes through all solutions.

`StepMode.PLAYER`  
: Will step once for each player (the default).
: Use if your update method handles just one solution each time.
: !!! note 
        `StepMode.PLAYER` matches number of players and not solutions, in case the count is not the same.

`StepMode.CUSTOM`  
: Will use the method `step_mode_check(self)` to check how many times to step.
: After each step the method is called, and if `True` is returned another step is performed.
: Use if the number of steps may vary (e.g. being able to miss a turn).
: !!! note
        Each time at least one step is performed.


### Examples

For a complete example of a Tournament Freecode which uses the GameChallenge, check out one of these (you have to be logged in to view the implementations):

- [Reversi](https://futureskill.com/freecode-creator/60ace4a1b705932a147b0686)
- [Widescreen](https://futureskill.com/freecode-creator/6668377f8e7fd9a591996e33) - how to remove parts of a GameChallenge
