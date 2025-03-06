---
title: Gameboard
contributors:
  - KimSimmons
search:
  exclude: true
---

!!! note
    This has mostly been replaced with [GameChallenge](../create_an_exercise/Skeletons.md#gamechallenge).

## \[DRAFT\]

The `GameBoard` library

A reusable and configurable system for game-like challenges.

**TODO**

* Board quick setup
* Creating and placing a gamepiece
* Pathfinding
* Styling concepts
* Custom board setup


## `GameBoard` quick setup

Here's an example to quickly get up and running: 
???+ example "Basic `GameBoard`"
    ```
    from gameboard import GameBoard
    from gameboard_helpers import CommonSetups, Style

    class Cell:
      def __init__(self, color, line_color):
        self.color = color
        self.line_color = line_color

    grass = Cell('green', 'darkgreen')
    mud = Cell('brown', 'darkgrey')

    class Challenge:
      fn __init__(self, context):
        ...
        board = GameBoard()
        CommonSetups.square(board)
        Style.topdown(board, context.canvas, tiles)
        board.fill(10, 10, grass)
        board.add_cell(2, 3, mud)
        fit_board_into(board, 5, 5, 10, 10)
    ```

The `CommonSetups` class configures the board to use a square tiled grid with four directions. The `Styles` class then configures how the board is displayed: the look of the tiles and how they are laid out.

\[TODO\]
* add link to documentation for `CommonSetups` and `Styles`.

You can fit the board into view with the function `fit_board_into(board, x, y, w, h)`, provided in the `gameboard_helpers` module.


## The `GameBoard` library

### `GameBoard`

A `GameBoard` can be used to represent a playfield with evenly spaced cells (like square or hexagonal).
Using the two matrices `base` and `projection` you're able to configure a multitude of orthographic styles.
The property `neighbor_deltas` gives a list of how cells are connected and is mainly used for pathfinding.


#### Cells

A location on the board is called a `cell`, consisting of a coordinate and an object describing the attributes.
You may add cells in any configuration, so non-square and irregular boards are allowed.
Use `add_cell(x, y, cell)` to add cells. The `cell` object may hold any set of attributes you may need.
Some specific attributes may be required by the predefined tile factories, described in the [Visuals](Gameboard.md#visuals) section.
The `cell` is examined by duck typing and is not expected to be derived of a certain class.

The method `fill(width, height, cell, x=0, y=0)` fills a square area; useful for quickly creating a square gameboard or rooms within one.


#### Layout

There are two important concepts for the layout of the board: the spatial transformation of coordinates and the neighbor deltas.
This section isn't strictly necessary to understand in depth, but it's useful if you're trying to customize a board beyond the `CommonSetups` and `Style` helpers.


##### Matrices

Some linear algebra is used to transform a cell coordinate into canvas coordinates, but no deeper understanding of it is required to configure the game board.
There are three `coordinate spaces`: cell, board and local canvas.
The `GameBoard` has two matrices called `basis` and `projection` which are used to transform points between the coordinate spaces.
\[TODO: Link to and explain math2d lib.\]

The cell coordinate space is what the game logic refers to as you add cells and place game pieces on the board.
This is most often an integer pair.

Multiplying the cell coordinate with the `basis` matrix gives you a point in `board space`.
Its primary use is to give a "flatter" space that is useful when dealing with hexagonal cells.
The cell and board spaces will likely look exactly the same for square grids, using an identity matrix.

The `projection` matrix makes things fancy.
It can take the flat board space and rotate and skew it into any `orthographic` view.
It only applies to the position of game pieces and will not skew the game piece graphics.


#### Visuals

The `GameBoard` constructs the tile visuals when adding a cell using the `tile_factory` function.
The `Style` class assigns this for you, but if the prebuild styles doesn't cover your needs you may wish to make your own `tile_factory` function and assign it to the `GameBoard`.

The `tile_factory` function should have the arguments `x`, `y` and `cell`, and return a new `GraphicalObject` that is the visual representation of this tile.
You'll likely need access to the `canvas` and other information about the board when creating a tile; To achieve this you may either wrap the configuration of the `tile_factory` in a closure or create a callable class using a `__call__(self, x, y,cell)` method.

???+ example "Using the tile factory"
    ```
    class Cell:
      def __init__(self, color):
        self.color = color

    grass = Cell('green')

    # The outer closure takes the additional parameters and returns the actual tile_factory functiothat has access to the closures arguments.`  
    def configure_tile_factory(canvas):
      def tile_factory(x, y, cell):
        color = getattr(cell, 'color', "#505050")
        return canvas.new_circle(1.0, color)
      return tile_factory

    class Challenge:
      def __init__(self, context):
        board = GameBoard()
        board.tile_factory = configure_tile_factory(context.canvas)
        board.add_cell(0, 0, grass)
    ```

The returned `GraphicalObject` will be managed by the `GameBoard` and placed automatically.


#### Helper tile_factory functions

The `gameboard_helper` contains a set of predefined `tile_factory` functions, used by the `Style` class but available on their own if needed.

`configure_square_tile_factory` creates square tiles that can be used with any projection, like topdown, isometric or any other orthographic layout.
The graphics are simple flat polygons, useful for quick prototyping without the need for more detailed graphics.
The `cell` object is expected to contain `color` and `line_color`.

`configure_hex_tile_factory` works like the square counterpart, except with hexagonal tiles.


### `GamePiece`

The `GamePiece` class should be used for anything that can be placed on the `GameBoard`.

???+ example "Adding a `GamePiece`"
    ```
    from gameboard import GameBoard, GamePiece

    class Challenge:
      def __init__(self, context):
        board = GameBoard()
        gfx = context.canvas.new_circle(1.0, '#FF00FF')
        actor = GamePiece(gfx)
        board.place_game_piece(actor, 1, 1)
    ```

You can use it as-is for many cases or create your own derived class for more flexibility.


### Pathfinding

The library comes with some general pathfinding tools that are easily configured with functions in the `gameboard_helper` module.
The `Pathfinding` class, located in the `gameboard` module lets you construct a graph and traverse it with an implementation of the AStar algorithm.
It's a very generalized implementation, though nodes are denoted with integer coordinates `(x,y)`.


#### Quick setup
???+ example "Setting up pathfinding"
    ```
    from gameboard import GameBoard
    from gameboard_helpers import generate_board_pathfinding

    class Challenge:
      def __init__(self, context):
        board = GameBoard()
        ...
        pathfinding = generate_board_pathfinding(board)
    ```


#### `GamePiece` pathfinding

You may pass the resulting pathfinding instance to one or more `GamePieces`.
Doing this enables the `step_towards` method that makes it very simple to traverse the board.

???+ example "Passing pathfinding to a `GamePiece`"
    ```
    from gameboard import GameBoard, GamePiece
    from gameboard_helpers import generate_board_pathfinding

    class Challenge:
      def __init__(self, context):
        board = GameBoard()
        ...
        pathfinding = generate_board_pathfinding(board)
        self.piece = GamePiece(gfx, pathfinding)
        board.place_game_piece(self.piece, 0, 0)

      def step(self):
        self.piece.step_towards(4, 5)
    ```

Each time `step_towards` is called the game piece is moved one step towards the target by referencing the pathfinding graph provided to the game piece's constructor.
The method returns `True` if a step was taken, `False` otherwise.
You may have to refer to the game piece's coordinate to see if it arrived at the goal or if it was unable to move.

`get_possible_moves` returns a list of coordinates a game piece may move to, based on the pathfinding graph.


#### Helper for generation

The `generate_board_pathfinding` will likely cover most uses.
By default it takes the `neighbor_delta` from the board itself to generate connections between cells.
If movement differs between characters you may provide a move set as the second argument.
The generator will consider walls as blocking if using a `WalledGameBoard` and setting the third argument `wall_check` to `True` (default).

It will also set node weights if the cell has a `weight` attribute (defaults to `1.0` otherwise), resulting in paths avoiding higher weight tiles without blocking them completely.

\[TODO\] Explain collision groups.


#### Building a graph

In case the provided `generate_board_pathfinding` isn't sufficient you may construct your own `Pathfinding` graph.
The `Pathfinding` class provides methods for adding nodes and connecting them.
It's not coupled with the gameboard in anyway and keeps track of its own graph of nodes.

\[TODO\] Explain the dynamic check function.
