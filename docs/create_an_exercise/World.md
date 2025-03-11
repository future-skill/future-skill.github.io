---
title: World
contributors:
  - Henrik Rostedt
  - Henrik
  - Ludvig
---

This article goes through how to use the World library to create a topdown or isometric world.


## Overview

The World library is split into two main parts:

- Core functionality, focused on flexibility and reuse
- Prefabricated content, focused on easy of use

This article will focus on the prefab content, but include enough core details to be able to take full advantage of the library.

The worlds created using this library are divided into descrete spaces, usually a square grid.
So positions will be based on integer coordinates.

The prefabricated content supports both topdown and isometric perspectives, and it is very easy to switch between the two as needed.
Additional perspectives are possible, but not supported by the prefab content.

All classes, types, and constants mentioned in this article can be imported from `lib.world`.

Easiest way to get started is to pick up one of the [templates](World.md#templates) at the end of the article, and then jump to relevant sections as questions come up.


## Setup

To use the prefab content you will have to use the `WorldSetup` class.
The following sections will assume that you have an instance of this class called `setup`.

To start, just create the setup object like this `setup = WorldSetup()`.

The class has the following attributes:

`directions` - `list[tuple[int, int]]`  
: The allowed movement directions, see [directions](World.md#directions), defaults to `CARDINAL_DIRECTIONS`

`basic` - `bool`  
: If `True` will draw colored geometric shapes rather than the image-based tiles and walls

`perspective` - `"topdown" | "isometric"`  
: The perspective to use for the prefab content, defautls to `"topdown"`

`resolution` - `int`  
: The "unit size", the size of a tile in topdown, will be used for all content

`block_corners` - `bool`  
: If `True` will block diagonal movement next to a wall, should generally be enabled when using `SQUARE_DIRECTIONS`

`debug_mode` - `bool`  
: If `True` adds additional debugging features to the world element, see [debugging](World.md#debugging)

`manager` - `PrefabManager`  
: The object responsible for managing prefabricated assets, generally no need to override the default value

???+ example "Creating the setup object"
    ```
    setup = WorldSetup(
        directions=SQUARE_DIRECTIONS,
        perspective="isometric",
        block_corners=True,
    )
    ```


### Directions

You can specify which directions exist in the world, which will affect how actors can move.

By default only the cardinal directions exist:

- `Cardinal.LEFT` = (-1, 0)
- `Cardinal.RIGHT` = (1, 0)
- `Cardinal.UP` = (0, -1)
- `Cardinal.DOWN` = (0, 1)

But these can be extended with diagonals:

- `Ordinal.LEFT_UP` = (-1, -1)
- `Ordinal.RIGHT_UP` = (1, -1)
- `Ordinal.LEFT_DOWN` = (-1, 1)
- `Ordinal.RIGHT_DOWN` = (1, 1)

For convenience, the following direction list constants are available:

- `CARDINAL_DIRECTIONS`
- `ORDINAL_DIRECTIONS`
- `SQUARE_DIRECTIONS`

You can specify any other set of directions you want, but it will seldom make sense to do so.


## World

To create the world itself you can use `world = setup.create_world()`.

This section will not go into detail of how the `World` object works, that will be covered throughout this article.


### Rendering

The world library can be used without rendering, but generally you do want to show the results.
When using the library you will initialize and make changes to the world in the `setup_state` and `update_state` methods.
Then you will want to render and re-render the world in the `setup_canvas`/`setup_view` and `update_canvas`/`update_view` methods.

When doing the initial render you can choose how the rendered element will behave:

- Use `world.render_resizable()` to get an element which dynamically scales the world to fit within the size of the element.
Recommended for smaller worlds.
- Use `world.render_scrollable()` to get a scroll area containing the world.
The size of the world is based on the `resolution`.
Recommended for larger worlds.

No matter which way you create the initial element, you need to call `world.render_changes()` when you want the world to be re-rendered.
No need to do anything else.

See the [templates section](World.md#templates) for examples of how to do this in practice.


## Tiles

Worlds consist mainly of tiles, which are used to both define the walkable area of the world and the visual tiles the world consist of.

The library comes with [prefabricated graphical tiles](World.md#prefab-tiles) and has built-in support for plain colored tiles, but it is also possible to make custom tiles.

There are a few ways to add tiles to the world, which to use is partly up to need and partly up to preference.

First way is to include a `tiles` argument to `setup.create_world()`.
This can take the form of a dictionary with coordinates as keys and tiles as values.

???+ example "Add tiles using a dictionary"
    ```
    world = setup.create_world(tiles={
        (0, 0): "grass",
        (1, 0): "grass",
        (0, 1): "gravel",
    })
    ```

Alternatively the `tiles` argument can be a string as described in [tile map](World.md#tile-map)

???+ example "Add tiles using a string"
      ```
      world = setup.create_world(tiles="gg\nr")
      ```

Second way is to use the `World` methods:

`world.add_tile(coordinate, data)`  
: Where `data` depends on what types of tiles you are using
:   ???+ example "Add single tile"
        ```
        world.add_tile((1, 1), {"name": "grass"})
        ```
: Or if using basic tiles:
:   ???+ example "Add single basic tile"
        ```
        world.add_tile((1, 1), {"color": "black"})
        ```

`world.fill(top_left, bottom_right, data)`  
: Which will add the same tile to the area spanned by `top_left` and `bottom_right`
:   ???+ example "Fill area with tiles"
        ```
        world.fill((0, 0), (1, 1), {"name": "mud", "color": "tan"})
        ```

It is possible to get tile data for a coordinate using `world.get_tile_data(coordinate)`, and it is possible to get all tile coordinates using `world.get_tile_coordinates()`.

You can remove a tile using `world.remove_tile(coordinate)`.


### Prefab tiles

| Constant           | Name          | Code | Notes    | Top down | Isometric |
|--------------------|---------------|------|----------|----------|-----------|
| Tile.BLOCKED       | blocked       | b    | blocking | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_tile_blocked.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_tile_blocked.webp){loading=lazy} |
| Tile.CARPET        | carpet        | c    |          | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_tile_beige_carpet.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_tile_beige_carpet.webp){loading=lazy} |
| Tile.CARPET_BLUE   | carpet_blue   | k    |          | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_tile_dark_blue_carpet.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_tile_dark_blue_carpet.webp){loading=lazy} |
| Tile.CARPET_BLUE_2 | carpet_blue_2 | K    |          | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_tile_light_blue_carpet.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_tile_light_blue_carpet.webp){loading=lazy} |
| Tile.CARPET_GREY   | carpet_grey   | C    |          | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_tile_grey_carpet.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_tile_grey_carpet.webp){loading=lazy} |
| Tile.GRASS         | grass         | g    |          | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_tile_grass.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_tile_grass.webp){loading=lazy} |
| Tile.GRASS_2       | grass_2       | G    |          | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_tile_grass_2.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_tile_grass_2.webp){loading=lazy} |
| Tile.GRAVEL        | gravel        | r    |          | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_tile_gravel.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_tile_gravel.webp){loading=lazy} |
| Tile.MUD           | mud           | m    |          | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_tile_ground.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_tile_ground.webp){loading=lazy} |
| Tile.PARQUET       | parquet       | p    |          | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_tile_dark_parquet.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_tile_dark_parquet.webp){loading=lazy} |
| Tile.PARQUET_2     | parquet_2     | P    |          | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_tile_light_parquet.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_tile_light_parquet.webp){loading=lazy} |
| Tile.TILES         | tiles         | t    |          | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_tile_stone_tiles.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_tile_stone_tiles.webp){loading=lazy} |
| Tile.TILES_2       | tiles_2       | T    |          | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_tile_bathroom_tiles.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_tile_bathroom_tiles.webp){loading=lazy} |
| Tile.WATER         | water         | w    | blocking | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_tile_calm_water.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_tile_calm_water.webp){loading=lazy} |
| Tile.WATER_2       | water_2       | W    | blocking | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_tile_water.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_tile_water.webp){loading=lazy} |
| Tile.SPACE_BLUE    | space_blue    | s    |          | N/A | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_tile_space_blue.webp){loading=lazy} |
| Tile.SPACE_GREEN   | space_green   | S    |          | N/A | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_tile_space_green.webp){loading=lazy} |


### Tile map

The tile map string is a convenient way of specifying what tiles to use.
Each tile is specified using a code (see table in previous section), use `.` to mark gaps.
To be able to add line breaks you will have to use triple `"` for the strings.
Or you can use the special sequence `\n` instead of adding a line break.
Leading and trailing whitespace is ignored, which allows indenting the tile maps.

???+ example "Tile string from main section"
    ```  
    ex1 = "gg\nr"
    # Equivalent to ex1
    ex2 = """
        gg
        r
    """
    ```

???+ example "Tile string using gaps"
    ```
    ex3 = """
        .wr.
        mrrg
        mrgg
        .rg.
    """
    # Equivalent to ex3, but hard to read
    ex4 = """.wr
    mrrg
        mrgg\n.rg"""
    ```


### Interactable tiles

Tiles can be interacted with using an on click listener.
The callback function to this listener is given to the `WorldSetup.create_world` function.

???+ example "Implementation of `on_tile_click` callback"
    ```  
    from lib.world import Tile 

    def on_tile_click(info: TileInfo) -> None:
        coord: tuple[int, int] = info.coordinate
        name: str | Tile = info["name"]
        print(f"Tile clicked: {name} at {coord}")

    world = self.setup.create_world(
        tiles=self.tiles,
        on_tile_click=on_tile_click,
    )
    ```

!!! note
    Depending on how the tile was instantiated `info["name"]` can either be a string or a `Tile(StrEnum)`.
    If the tiles are generated using a `TileMap` the name in `TileInfo` will be a `Tile(StrEnum)`, but if instantiated using `World.add_tile({"name": "tile_name"})` the name in `TileInfo` till be the string `"tile_name"`.


## Walls

It is possible to place walls between tiles in the world, these will block movement between the tiles.

While tiles are identified using a single coordinate, walls are identified using a pair of coordinates.
The pair is the coordinates of the tiles the wall is placed between.
E.g. if the wall should be placed between tiles `(0, 1)` and `(1, 1)` then the coordinate pair is `((0, 1), (1, 1))`.

Like tiles, the library comes with [prefabricated graphical walls](World.md#prefab-walls) and support for plain colored walls.
And it is possible to make custom walls.

Walls are added to the world in a similar way as tiles, except there is no equivalent to tile maps.

First way is to include a `walls` argument to `setup.create_world()`.
This takes the form of a dictionary with coordinate pairs as keys and walls as values:

???+ example "Add walls using a dictionary"
    ```
    world = setup.create_world(tiles={
        ((0, 0), (1, 0)): "hedge",
        ((0, 0), (0, 1)): "stone",
    })
    ```

Second way is to use the `World` method:

`world.add_wall(coordinate_pair, data)`  
: Where `data` depends on what types of walls you are using
: ???+ example "Add a single prefab wall"
        ```
        world.add_wall(((0, 1), (1, 1)), {"name": "hedge_2"})
        ```
: Or if using basic walls:
: ???+ example "Add a single basic wall"
        ```
        world.add_wall(((1, 0), (1, 1)), {"color": "orange", "height": 0.3})
        ```

It is possible to get wall data for a pair of coordinates using `world.get_wall_data(coordinate_pair)`.

You can remove a wall using `world.remove_wall(coordinate_pair)`.


### Prefab walls

| Constant          | Name         | Top down | Isometric |
|-------------------|--------------|----------|-----------|
| Wall.BRICK        | brick        | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_wall_brick_face.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_wall_brick_left.webp){loading=lazy} |
| Wall.BRICK_FENCE  | brick_fence  | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_wall_brick_fence_face.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_wall_brick_fence_left.webp){loading=lazy} |
| Wall.HEDGE        | hedge        | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_wall_hedge_face.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_wall_hedge_left.webp){loading=lazy} |
| Wall.HEDGE_2      | hedge_2      | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_wall_hedge_2_face.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_wall_hedge_2_left.webp){loading=lazy} |
| Wall.PICKET_FENCE | picket_fence | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_wall_picket_fence_face.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_wall_picket_fence_left.webp){loading=lazy} |
| Wall.PLAIN        | plain        | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_wall_white_face.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_wall_white_left.webp){loading=lazy} |
| Wall.STONE        | stone        | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_wall_stone_face.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_wall_stone_left.webp){loading=lazy} |


### Block corners

Due to the geometry of a square grid, you will generally only want to have visual walls between tiles in the cardinal directions.
However, when allowing diagonal movement (e.g. using `directions=SQUARE_DIRECTIONS`) the characters can move diagonally across the corners.
This will look incorrect and not make intuitive sense for the users.

To solve this you can set `block_corners=True` in the `WorldSetup` or `World` constructors.
This will automatically add invisible walls at the corners next to walls so that movement is blocked as expected.

!!! note
    These walls must be manually removed if needed.


## Actors

Everything that is not a tile or a wall is an `Actor`.
This includes characters, objects, and even props like flowers.

The easiest way to create actors is to use the `WorldSetup` methods (see following sections), but you can also create them manually if you need more flexibility.

The `World` class has the following `Actor`-related methods:

`world.add_actor(coordinate, actor)`  
: Places the actor in the world at the given coordinate.
: The actor will be configured to follow the movement rules of the world.
: Multiple actors can be placed on the same coordinate.

`world.remove_actor(actor)`  
: Removes the actor from the world.

`world.find_actors()`  
: Allows looking up actors based on various criteria, returns a list.
: If given no arguments it returns all actors in the world.
: The following arguments are supported:
: `at` - `tuple[int, int]`  
    : Coordinate (tile) to search at.
    : ???+ example "Find actor at tile `(0, 0)`"
            ```
            world.find_actors(at=(0, 0))
            ```
: `tags` - `set[str]`  
    : Tags the actor must have.
    : ???+ example "Find actors with tag `"my_tag"`"
            ```
            world.find_actors(tags={"my_tag"})
            ```
: `tags_match_any` - `bool`  
    : Only requires that one tag in `tags` matches.
    : ???+ example "Find actors with either the `"player"` or `"enemy"` tag"
            ```
            world.find_actors(tags={"player", "enemy"}, tags_match_any=True)
            ```


### Attributes

All actors have a number of attributes that usually can be set when creating the actor:

`actor.coordinate` - `tuple[int, int] | None`  
: The current coordinate of the actor, or `None` if the actor is not in a world.

`actor.offset` - `tuple[float, float] | None`  
: A visual offset on the current tile, mainly used to add small props (such as rocks and flowers).

`actor.tags` - `set[str]`  
: A set of tags for the actor, which are used in `world.find_actors` or other features.

`actor.block_tags` - `set[str]`  
: A set of tags this actor is blocked by. If a space has an actor with any of these tags, then this actor can not move there.


### Deletion

All actors have the `actor.delete()` method with will ask for the actor to be removed from existence, such as the world or a container.
If the actor was rendered manually, this will have no effect.


### Movement

This library has a pathing system which enables actors to find the quickest path from point a to point b.

In order for this system to work, you must call `world.update_world_graph()`.
This should be done after finishing setting up the world, and when you have modified any tiles or walls.

Actors have a set of methods related to movement:

`actor.can_move(direction)`  
: Returns `True` if the actor currently can move in the given direction.

`actor.move(direction)`  
: Moves the actor one step in the given direction.
: Does not require movement to be properly set up.
: ???+ example "Move `actor` one tile to the left"
        ```
        actor.move(Cardinal.LEFT)
        ```

`actor.move(direction, check=True)`  
: Same as above, but includes additional checks (such as blocked check).
: Returns `False` if the movement failed.
: Requires movement to be properly set up.
: ???+ example "Attempt to move `actor` one tile to the left"
        ```
        actor.move(Ordinal.LEFT_DOWN, check=True)
        ```

`actor.possible_moves()`  
: Returns the possible directions this actor can move (currently).

`actor.next_move_towards(coordinate)`  
: Returns the current direction the actor should move to get one step closer to the given coordinate along a shortest path.
: Returns `None` if there is no path to to coordinate or if the actor is already there.

`actor.move_towards(coordinate)`  
: Moves the actor one step towards the given coordinate along a shortest path.
: Returns `False` if the movement failed or the actor is already there.
: ???+ example "Move `actor` one step towards tile `(3, 4)`"
        ```
        actor.move_towards((3, 4))
        ```

You can have a callback trigger whenever the actor performs a successful movement, by supplying the following argument when creating an actor:

`on_move` - `(ActorMoveEventData) -> None`  
: Callback to call when the actor successfully moves (one step)
: The `ActorMoveEventData` object has the following attributes:
: `source` - `Actor`  
    : The actor that moved
: `direction` - `tuple[int, int]`  
    : The direction the actor moved
: `origin` - `tuple[int, int]`  
    : The coordinate the actor came from


### Keyboard Input

This library has a keyboard input system which enables users to use the keyboard to move actors or perform other actions.

When creating an actor you can supply the following arguments:

`key_map` - `dict[str, tuple[int, int] | (ActorKeyboardEventData) -> None] | None`  
: Maps keys to movements and/or callbacks.

`on_key_up` - `(ActorKeyboardEventData) -> None`  
: Callback to call if the key is not found in the key map.
: The `ActorKeyboardEventData` object has the following attributes:
: `source` - `Actor`  
    : The actor that received the key event
: `key` - `str`  
    : The key value

We supply a default keymap for cardinal movement called `MOVE_KEY_MAP` which supports arrow keys and ++w++ ++a++ ++s++ ++d++ movement.

???+ example "Using default movement for a main character"
    ```
    world.add_actor((0, 0), setup.create_character("skilly", "blue", key_map=MOVE_KEY_MAP))
    ```

???+ example "Adding an additional action when pressing ++p++"
    ```
    world.add_actor((0, 0), setup.create_character("skilly", "orange", key_map=MOVE_KEY_MAP | {"p": lambda e: e.source.pick_up()}))
    ```

???+ example "Printing the value when a key is pressed"
    ```
    world.add_actor((0, 0), setup.create_character("skilly", "orange", on_key_up=lambda e: print(e.key)))
    ```


### Interaction

This library has an interaction system with allows responding to mouse clicks or can be used with keyboard input or just through code.

Actors have a set of methods related to interaction:

`actor.interact()`  
: Invokes the actors `on_interact` callback.

`actor.interact(agent=agent)`  
: Invokes the actors `on_interact` callback, specifying the interaction agent (must be an `Actor`).

When creating an actor you can supply the following arguments:

`on_interact` - `(ActorInteractEventData) -> None`  
: Callback to call when this actor is interacted with.

`interact_on_click` - `bool`  
: If set to `True`, `on_interact` will be called when the user clicks on this actor.

The `ActorInteractEventData` object has the following attributes:

`source/target` - `Actor`  
: The actor that received the key event

`agent` - `Actor | None`  
: The actor that caused the event, set to `None` if not caused by another actor (such as clicked by user).

Some prefab actors have predefined `on_interact` callbacks, which can be overridden and are detailed under their respective section.


## Characters

Character actors have a few extra features and, most importantly, are animated.
The characters will automatically get appropriate walking animations based on movement, and there are a couple of additional actions with dedicated animations.

When you call `world.render_changes()` the appropriate animation will be chosen.
If you want to perform multiple movements or actions, you have to use `canvas.split_step()` and call `world.render_changes()` multiple times.

Like with props, there is a dedicated method to create a character `setup.create_character(name, color)`. 
Available characters and colors are listed in the following tables:

| Constant          | Name    | Description | Top down | Isometric |
|-------------------|---------|-------------|----------|-----------|
| Character.ADA     | ada     | a girl      | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_ada_black_idling.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_ada_black_idling.webp){loading=lazy} |
| Character.DOUGLAS | douglas | a dog       | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_douglas_black_idling.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_douglas_black_idling.webp){loading=lazy} |
| Character.KATNISS | katniss | a cat       | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_katniss_black_idling.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_katniss_black_idling.webp){loading=lazy} |
| Character.RUST    | rust    | a boy       | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_rust_black_idling.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_rust_black_idling.webp){loading=lazy} |
| Character.SKILLY  | skilly  | a robot     | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_skilly_black_idling.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_skilly_black_idling.webp){loading=lazy} |

| Constant              | Name   |
|-----------------------|--------|
| CharacterColor.BLACK  | black  |
| CharacterColor.BLUE   | blue   |
| CharacterColor.ORANGE | orange |
| CharacterColor.RED    | red    |
| CharacterColor.WHITE  | white  |
| CharacterColor.YELLOW | yellow |

Character actors have an additional set of methods with animations:

`actor.pick_up()`  
: [Interacts](World.md#interaction) with any other actors on the current tile, and plays a picking up animation.

`actor.gesture(direction)`  
: Plays a gesturing animation in the given direction.
: ???+ example "Make `actor` gesture facing down"
        ```
        actor.gesture(Cardinal.DOWN)
        ```

`actor.say(text)`  
: Plays a speaking animation and shows a speech bubble with the given text.
: ???+ example "Make `actor` say "Hello World!""
        ```
        actor.say("Hello World!")
        ```

Additional optional parameters:

`font_size`
: Adjust the font size for the bubble.

`theme`
: See [speech bubble](UI.md#speechbubble) documentation.

`actor.interact_with(dir=dir)`  
: [Interacts](World.md#interaction) with any actors on the tile in the specified direction, and plays a gesturing animation.


## Props

Prop actors are simply prefabricated actors that can be placed in the world.
It is up to you to decide how to use them.

Props are created using `setup.create_prop(name)` which also accepts various `Actor` and `Image` attributes.
The available prefab props are listed in the following table:

| Constant              | Name        | Notes     | Top down | Isometric |
|-----------------------|-------------|-----------|----------|-----------|
| Container.BARREL      | barrel      | container | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_barrel1.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_barrel1.webp){loading=lazy} |
| Container.BARREL_SIDE | barrel_side | container | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_prop_barrel2.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_barrel2.webp){loading=lazy} |
| Container.CHEST       | chest       | container | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_prop_chest1.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_chest1.webp){loading=lazy} |
| Container.CHEST_BAND  | chest_band  | container | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_prop_chest2.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_chest2.webp){loading=lazy} |
| Container.CRATE       | crate       | container | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_prop_crate1.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_crate1.webp){loading=lazy} |
| Container.CRATE_TALL  | crate_tall  | container | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/td_prop_crate2.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_crate2.webp){loading=lazy} |
| Item.COIN             | coin        | item      | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_coin.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_coin.webp){loading=lazy} |
| Item.HEART            | heart       | item      | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_heart.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_heart.webp){loading=lazy} |
| Item.KEY              | key         | item      | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_key.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_key.webp){loading=lazy} |
| Item.STAR             | star        | item      | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_star.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_star.webp){loading=lazy} |
| Prop.BUSH             | bush        |           | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_bush1.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_bush1.webp){loading=lazy} |
| Prop.FLOWER           | flower      |           | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_flower1.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_flower1.webp){loading=lazy} |
| Prop.FLOWER_2         | flower_2    |           | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_flower2.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_flower2.webp){loading=lazy} |
| Prop.STONE            | stone       |           | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_stone.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_stone.webp){loading=lazy} |
| Prop.STONE_2          | stone_2     |           | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_stone_2.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_stone_2.webp){loading=lazy} |
| Prop.TREE             | tree        |           | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_tree1.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_tree1.webp){loading=lazy} |
| Prop.TREE_STONES      | tree_stones |           | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_tree2.webp){loading=lazy} | ![](https://sws-uploads.s3.eu-central-1.amazonaws.com/lazy/iso_prop_tree2.webp){loading=lazy} |

Smaller props such as flowers and stones can preferably be added as cosmetic scenary to world, with an offset from the center.
Here is an example where a couple of stones and flowers are added:

???+ example "Add a couple of stones anf flowers to the world"
    ```
    scenary = [
        ("stone", (0, 0), (-0.3, 0)),
        ("flower", (0, 0), (0.3, 0.4)),
        ("stone", (1, 0), (0.2, -0.4)),
    ]
    for name, coord, offset in scenary:
        world.add_actor(coord, setup.create_prop(name, offset=offset))
    ```

Bigger props should generally be used without an offset to avoid intersecting walls and characters.
You would generally also want to give the bigger props a tag that can be used to block movement.

???+ example "Add a prop with a tag to the world"
    ```
    world.add_actor((0, 2), setup.create_prop("bush", tags={"obstacle"}))
    ```

The props marked as `container` or `item` can be used as normal props, but if you want to have them be opened/broken, then you should take a look at the [containers section](World.md#containers) or [items section](World.md#items).


## Containers

Container actors are created using the method `setup.create_container(name)` with one of the names marked as `Container` in [the props table](World.md#props).

The containers feature a very lightweight inventory system where `ItemActor` objects can be "stored" inside the container.
When the container is opened, the first of these items will be displayed on/inside the container.

By default, when interacting with the container it will (1) be opened if closed, or (2) the first contained item will be interacted with (usually leading to it being removed, depending on the item itself).

Containers have the following extra attributes:

`actor.items` - `list[ItemActor]`  
: List of items contained within the container, readonly

`actor.opened` - `bool`  
: Whether the container currently is opened/broken or not, readonly

And the following extra methods:

`actor.add_item(item)`  
: Adds the given item to the container (at the end)

`actor.remove_item(item)`  
: Removes the given item from the container

`actor.open()`  
: Opens/breaks the container

`actor.close()`  
: Closes/un-breaks the container

The containers have animations that play when they open/break, and then they will keep the open/broken appearance.
They can then be closed, but there currently are no animations playing for this action.


## Items

Item actors are created using the method `setup.create_item(name)` with one of the names marked as `Item` in [the props table](World.md#props).

By default, when interacting with an item any `on_pick_up` callbacks will be called and then the item will be [deleted](World.md#deletion).
If the `on_pick_up` callback returns `True`, then the item will not be deleted.
This default behaviour can be overridden by setting `on_interact` manually, however, it is usually better to just set `on_pick_up`.

Items have the following extra attributes:

`actor.hidden` - `bool`  
: Whether the item currently is hidden or not, readonly

And the following extra methods:

`actor.reveal()`  
: Reveals the item

`actor.hide()`  
: Hides the item

`actor.pick_up()`  
: Manually invoke the items `on_pick_up` callback, with the same behaviour as default interaction

The items have animations that play when they are revealed.
They can then be hidden, but there currently are no animations playing for this action.


## Debugging

The world element has a subtle menu on the top left which can be used to activate various visualizations:

`(x, y)` - coordinates  
: Shows coordinates for all tiles, which can greatly help when placing actors or writing solutions.
: Always available.

`nav` - navigation  
: Visualization of the pathfinding graph, does not take into account blocking tags.
: Requires debug mode.


## Custom Prefabs

It is possible to extend the existing list of prefabs with your own additions.
This is not trivial and only recommended if you are well versed in the Freecode platform and have some image editing experience (you will have to provide your own images in a particular format).

All classes or variables mentioned in the following section needs to be imported from `lib.world.prefab`.

Although you can create your own prefab manager, you would generally just register them with the `DEFAULT_PREFAB_MANAGER` which is used by default.
Just keep in mind that it is better to add new entries rather than overriding existing ones.
There is no way to add new values to the enums, so you will have to only use the strings directly (or define your own string enum, or use constants).

For each prefab type, the manager has a corresponding `.register_`<type>`s(entry_1, entry_2, ...)` method (e.g. `.register_items(...)`).
It can be used to register one or more prefab entries.
The entries are instances of the corresponding <Type>`Entry` classes for the prefab types (e.g. `ItemEntry`).

!!! note
    If you register any prop variant (such as container or item) a corresponding prop entry will automatically be created for you.

The entry instance allows you to tweak a number of parameters which will be used to determine the image names, positioning, and more.
The following subsections detail what parameters exist and what they do, as well as what image names are expected.

!!! note
    Images must be provided in WebP, which should be supported by most modern image editors.


### TileEntry

`name` - `str`  
: The name to be used to refer to this prefab.
Required and must be unique within the prefab category (will otherwise override the others).

`image` - `str | None` (default `None`)  
: An alternative name to use for determining the image name.

`animated` - `bool` (default `False`)  
: Whether the prefab is animated or not.
Animations requires an additional sprite sheet json file.

`code` - `str | None` (default `None`)  
: A code to be used in tile maps. Must be exactly one character.

`color` - `str` (default `"magenta"`)  
: The color to use when `basic` mode is enabled.

`blocked` - `bool` (default `False`)  
: Whether the tile should be considered blocked for movement.

Tile images are named `<perspective>_tile_<name>.webp`.
???+ example "The name of a top down grass tile"
    ```
    td_tile_grass.webp
    ```


### WallEntry

`name` - `str`  
: The name to be used to refer to this prefab.
Required and must be unique within the prefab category (will otherwise override the others).

`image` - `str | None` (default `None`)  
: An alternative name to use for determining the image name.

`animated` - `bool` (default `False`)  
: Whether the prefab is animated or not.
Animations requires an additional sprite sheet json file.

`color` - `str` (default `"magenta"`)  
: The color to use when `basic` mode is enabled.

`height` - `float` (default `0.5`)  
: The height of the wall when `basic` mode is enabled.

Wall images are named `<perspective>_wall_<name>_<direction>.webp`.
Directions for topdown are `face` and `side`, and directions for isometric are `left` and `right`.
???+ example "The name of an isometric stone wall with the left direction"
    ```
    iso_wall_stone_left.webp
    ```


### CharacterEntry

`name` - `str`  
: The name to be used to refer to this prefab.
Required and must be unique within the prefab category (will otherwise override the others).

`image` - `str | None` (default `None`)  
: An alternative name to use for determining the image name.

`colors` - `set[str]` (default `set(CharacterColor)`)  
: The set of supported colors for the character.

`size` - `tuple[float, float]` (default `(1, 1)`)  
: The size of the prefab, in world units.

`center_offset` - `tuple[float, float]` (default `(0, 0)`)  
: Offset from the center of the image to the center of the prefab footprint.

`face_offset` - `tuple[float, float]` (default `(0, 0)`)  
: Offset from the center of the image to the center of the character face.

There are too many images used for characters to list out the names here.


### PropEntry

`name` - `str`  
: The name to be used to refer to this prefab.
Required and must be unique within the prefab category (will otherwise override the others).

`image` - `str | None` (default `None`)  
: An alternative name to use for determining the image name.

`animated` - `bool` (default `False`)  
: Whether the prefab is animated or not.
Animations requires an additional sprite sheet json file.

`size` - `tuple[float, float] | dict[str, tuple[float, float]]` (default `(1, 1)`)  
: The size of the prefab, in world units.
: Can give a dictionary to have different values based on perspective.

`center_offset` - `tuple[float, float] | dict[str, tuple[float, float]]` (default `(0, 0)`)  
: Offset from the center of the image to the center of the prefab footprint.
: Can give a dictionary to have different values based on perspective.

`prespective` - `str | None` (default `None`)  
: If set, override with the given perspective when determining the image name.

Prop images are named `<perspective>_prop_<name>.webp`.
???+ example "The name of an isometric bush prop"
    ```
    iso_prop_bush.webp
    ```
!!! note
    The perspective must match if a particular perspective was forced.


### ContainerEntry

Same fields as `PropEntry`, with the following additions:

`item_offset` - `tuple[float, float] | dict[str, tuple[float, float]]` (default `(0, 0)`)  
: Offset from the center of the footprint of the container to the center of the footprint of the showcased item.
: Can give a dictionary to have different values based on perspective.

Containers follow the same naming as props, except that the base image should be the open animation.
An image suffixed with `_closed` will be used for the default state and an image suffixed with `_opened` will be used for the opened state.


### ItemEntry

Same fields as `PropEntry`.

Items follow the same naming as props. An image suffixed with `_spawn` will be used for the reveal animation.


## Templates

The following template is a bit rough around the edges and might not always showcase best practices.

This template showcases how the world library can be used together with the [`GameChallenge` skeleton](Skeletons.md#gamechallenge):

```
"""
This is the module containing the challenge implementation.
"""
from lib.exceptions import SolutionException
from lib.skeletons import LevelInfo, GameChallenge, GameInfo, PlayerInfo, StatisticInfo
from lib.world import MOVE_KEY_MAP, WorldSetup


# NOTE: only needed due to a bug
class HumanAdapter:
    pass


class Challenge(GameChallenge):
    """
    This is a challenge implementation using the game skeleton and the world library.
    """
    
    def setup_game(self):
        """
        This method returns required game information for setting up the challenge.
        """
        # The info prompt shows up in the bottom left corner in interact mode
        self.info_prompt = "Move using WASD or arrow keys"
        # We don't use the turn system, this change is just cosmetic
        self.auto_finish_turn = True
        return GameInfo(
            title="My game",
            summary="Follow the path",
            description="My game description.

Which can use <em>html formatting</em>!",
        )
    
    def setup_players(self):
        """
        This method returns a list with required information for each player.
        """
        return [PlayerInfo(
            role="Player",
            name="Your solution",
            image="TODO",
        )]

    # Keep track of points using a statistic
    def setup_statistics(self):
        return [StatisticInfo("points", suffix="points", type=int, default=10)]

    # Disable the auto-finish turn setting
    def setup_settings(self):
        return []

    def setup_level(self, level):
        """
        This method returns required level information for the current level.
        The "level" parameter is the level index, starting at 0.
        """
        match level:
            case 0:
                name = "Right down the path"
                self.tiles = """
                    rrrrr
                    GGGGr
                    GGGGr
                    GGGGr
                    GGGGr
                """
                self.walls = {}
                self.path_length = 8
            case 1:
                name = "Not straight-farward"
                self.tiles = """
                    rrrGb
                    mmrGb
                    mrrGb
                    mrGGG
                    mrrrr
                """
                self.walls = {}
                self.path_length = 10
            case 2:
                name = "Walls as well"
                self.tiles = """
                    rrmmm
                    grrrm
                    grrrm
                    grrrm
                    wwwrr
                """
                self.walls = {
                    ((1, 1), (2, 1)): "stone",
                    ((1, 2), (2, 2)): "stone",
                    ((2, 2), (3, 2)): "hedge",
                    ((2, 3), (3, 3)): "hedge",
                    ((4, 3), (4, 4)): "picket_fence",
                }
                self.path_length = 12
            case 3:
                name = "Not so square"
                self.tiles = """
                    rGGGb
                    rrr.g
                    .mrrr
                    ..mrr
                    ...rr
                """
                self.walls = {
                    ((3, 2), (3, 3)): "hedge_2",
                    ((4, 3), (4, 4)): "brick_fence",
                }
                self.path_length = 10
            case _:
                raise Exception(f"level {level} not supported")
        return LevelInfo(name=name, max_score=10)

    def setup_state(self):
        """
        This method is where you should do all initial setup, except for graphics.
        """
        # We first need to create the world setup, in this case we just use the default
        self.setup = WorldSetup()
        # Create the world using the tiles and walls assigned for the level
        self.world = self.setup.create_world(tiles=self.tiles, walls=self.walls)
        # Needed to enable movement
        self.world.update_world_graph()
        # Create the player character with standard movement controls and a movement callback
        self.player_character = self.setup.create_character("rust", "red", key_map=MOVE_KEY_MAP, on_move=self.move_callback)
        # Add the player to the world at the starting position
        self.world.add_actor((0, 0), self.player_character)
        # Keep track of how many steps the player has taken
        self.step_count = 0

    def setup_view(self):
        """
        This method returns the main view for the challenge, which can be any graphics element.
        """
        # We want the world to be resized to fit the designated area
        # For a bigger world we would want to use "render_scrollable"
        return self.world.render_resizable()

    def update_state(self):
        """
        This method is where you should call solutions and update the current state.
        It is called continuously until `self.finished` is set to "True".
        """
        # When calling a solution you need to handle any "SolutionException"
        try:
            solution = self.context.solutions[0]
            # TODO: call a solution method
        except SolutionException:
            # Code put here will run if the solution crashed
            pass

    def update_view(self):
        """
        This method is where you should update the view based on the current state.
        """
        # Need to trigger rendering of any changes
        self.world.render_changes()

    # We add this method to be called whenever the player character moves
    def move_callback(self, event):
        # If the player takes too many steps they will lose points
        self.step_count += 1
        if self.step_count > self.path_length:
            self.players[0].points -= 1
        # If the player moves off the path they will lose points
        tile_data = self.world.get_tile_data(self.player_character.coordinate)
        match tile_data["name"]:
            # On the path, do not deduct points
            case "gravel":
                pass
            # Walked on mud, deduct double points
            case "mud":
                self.players[0].points -= 2
            # Matches any other tile
            case _:
                self.players[0].points -= 1
        # If the player has reached the end, we stop the run
        if self.player_character.coordinate == (4, 4):
            self.finished = True
            self.info_prompt = "You made it to the end of the path!"
        # We can update the scores here
        self.scores[0] = self.players[0].points)
```
