# RPGiabMakerToolkit
A toolkit of scripts to help speed up development using RPG-in-a-box.

# Installing

This requires [RPG-in-a-box](https://rpginabox.com). This is a full project that can be opened and
used directly. Assets can be copy/pasted into your own RPGiab projects for use. Suggested that
all scripts prefixed with "rmt" be copied over if doing so.

# Setup

To make this tookit work on a general level there are some key steps to setup per below:

1. Add a global property called "start" and supply the name of the map to load at startup
2. Add a global property called "defaultZOffset" and set it to the default map increment you'll be using (currently project uses 8)
3. When running a project that uses this code, ensure rmtStart is set as the start scripts
4. For any maps using this toolkit, ensure "rmtLoadMap" is set as the preload script
5. For any maps using this toolkit, find a tile and name it "start" to designate the start point

# Features

Below are some of the current features provided by the toolkit and how they are Suggested

## Auto-Pathing

Maps in this toolkit do not require navigation paths to be added. Instead, there is a system of scripts
that automatically wire paths for the player based on certain criteria. Auto-pathing is triggered by two scripts:

- rmtPathTrigger - primary trigger that kicks off the path connector script as well as triggers some other functionality (moveable blocks / animations)
- rmtPathConnector - main script responsible for wiring paths based on a set of criteria

Auto-pathing works by checking the tiles in four cardinal directions from the player, and determining if a path can be added for moment. This system depends on some level of consistency in tile placement, so for now requires a consistent increment to be used on the Z axis (vertical). The vertical increment is recommended to be the same as the x/y size of the map tiles. However, this can be adjusted using the global $defaultZOffset setting, or via specific tile zOffset property, or a map-specific configuration (more on these later).

### Evaluation Method

The path-checking starts by using the player coordinates and looking at the surrounding tiles on the same z-plane as the player. Below are some of the basic rules followed along with special tile-type interactions that change the behavior of the navigation paths.

The table below handles all the cases if there's no tile on top of the floor tile. Behaviors on navigation change when there's a tiles
that is position at +zOffset on top of the floor tile.

| Passable Property == false? | Has Impassable Tag? | Navigation Created |
|-----------------------------|---------------------|--------------------|
| No                          | No                  | Walk And Interact  |
| Yes                         | No                  | NONE               |
| Yes                         | Yes                 | NONE               |
| No                          | Yes                 | NONE               |
| No                          | Yes                 | NONE               |

When the floor tile has another tile on top that's at the +zOffset position, another set of evaluation happen to determine how to handle
the navigation wiring.

#### Stairs

*Model Tag*: stair

Stairs placed on top of the floor have a special property. Currently this is hard-coded in the pathing, but a stair assumes
there is only one entry point allowed to create a navigation to go "up". The current stair setup has the stair entrance at the "Front" of the stair, which is the cardinal direction SOUTH.

When the system evaluates stairs, it looks at the current direction of the player, and ensures the stair direction connects with the base tile appropriately.

Note: current a similar check is not done for stairs going "down", meaning it's possible to place a stair the wrong way and still get a path created on it.

#### Bumps

*Model Tags*: passable, bump

A bump is a special kind of tile that requires navigation to be drawn to the "topTile" but does not provide a stair-like function. This is currently used for the "button" tile. It allows the player to step up on the top tile, but then future navigation paths are drawn based on the tile _below_ the bump.

This is useful so the player can step up and down on the bump without the bump becoming a stair to another level.

#### Doors

*Model Tag*: door
*Animation States*: open, close
*Properties*:
  - locked: true / false :: if true, prevents the door from opening automatically when the player is next to it
  - isOpen: true / false :: primarily used by the script that manages the door. Retains the open / closed state of the Doors

The current implementation of doors is pretty simple. The door needs to have the proper tag, and, if it should be animated, it needs an "open" and "close" animation. By default doors do not open automatically. They can be made to open automatically by setting the "locked" property to false (note: if making your own door models you can set default custom properties to get whatever default behavior you want);

#### Moveable Blocks

Moveable blocks are tiles that can be pushed around by the player. This functionality is still experimental, and there is some hard-coding in the rmtPathTrigger script that makes this only work with the moverableBlock8 model.

Moveable blocks can only be pushed on flat ground. They will not go up stairs, through doors, or over bumps.

#### Passable
*Model Tag*: passable

Any tile model can have a "passable" tag added to it. If this tag exists, a navigation path will be drawn on it so the player can walk over properly. Note that

#### Impassable

*Model Tag*: impassable

A tag used to prevent the player from walking on a tile. Good for things like water / lava / gaps / etc.

## Event Triggers

Because the auto-pathing uses a special script to trigger navigation, additional triggers tied to "EnterTile" need to be handled a little differently. To make an "EnterTile" script execute, you must add a custom property sting "OnEnterTile" and set the value to the string name of the script to trigger. 

You can see an example of this with the prototype button.
