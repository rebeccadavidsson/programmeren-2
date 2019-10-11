# Adventure

## tl;dr

- Implement Crowther's Adventure game using OOP in Python.
- Play your game!

## Background

Back in the days, before dedicated graphics cards were a thing, text-based adventure games were incredibly popular. This type of game consists entirely out of text, and is traversed by commands much like the ones you would enter in the terminal. One such game is Colossal Cave Adventure, created by [William Crowther](https://en.wikipedia.org/wiki/William_Crowther_(programmer)) in 1975, that served as the inspiration for the text adventure game genre.

In Adventure you have to navigate between "Rooms" through commands such as "WEST" and "EAST", but also "IN" or "OUT":


    You are standing at the end of a road before a small brick
    building. A small stream flows out of the building and
    down a gully to the south. A road runs up a small hill
    to the west.
    > WEST
    You are at the end of a road at the top of a small hill.
    You can see a small building in the valley to the east.
    > EAST
    Outside building.
    >

That first part of the adventure "map" may look like this:

![](map.png){:.w300}

You can find the full map, including a spoiler-free version, [at this website](http://www.spitenet.com/cave/), but note that you will be implementing a portion of the full map!

But there is more than just navigating, at all times you can ask for `HELP` for an explanation of the game, or `LOOK` to get a detailed description of the room you are in.
From the previous example you could see that the second time a room is entered a shorter description was shown. If we were to enter the `LOOK` command we would again see the following:

    > LOOK
    You are standing at the end of a road before a small brick
    building. A small stream flows out of the building and
    down a gully to the south. A road runs up a small hill
    to the west.
    >

The adventure "map" is provided in a few **data files**, that contain room names and description, and in particular, which rooms are connected to other rooms, and using which commands.

Though Crowther originally wrote his game in Fortran, an imperative programming language that has been around since the 1950s, we will be taking a more modern approach to its implementation, using object-oriented programming (OOP). OOP is particularly suited to Adventure, because its main idea is a series of rooms that are connected. Each room will be an object, and all of these objects will point to each other.


## Specification

Implement an object-oriented version of Crowther's Adventure game using the class structure provided below. It should:

1. implement **loading** of the map:
	* handling command line arguments to open a given datafile
	* loading map data into a series of objects
2. implement user **interaction**:
	* prompting the user for commands and execute those
	* warn about non-existent commands
	* moving the player from room to room
3. implement game **logic**:
	* forced movements
	* solving the game


### Distribution

	$ cd ~/module9
	$ wget https://prog2.mprog.nl/course/problems/adventure/adventure.zip
	$ unzip adventure.zip
	$ rm adventure.zip
	$ cd adventure
	$ ls
	adventure.py  data/  room.py


## Understanding

### `data/`

The `data` contains four datafiles with which you can create two versions of adventure:

- `TinyRooms.txt` and `TinyItems.txt` together are the smallest adventure game, consisting of 3 rooms and 2 items. Great for starting out with!

- `SmallRooms.txt` and `SmallItems.txt` are a bit larger and include more advanced interactions.


### `adventure.py`

Take a look at `adventure.py`. The file has three main components.

1. The `import` statement. Instead of working from a single file, we've split our two classes into separate files, in order to keep our files short and tidy. To be able to access the class from one file in the other, we have to import them:

		from room import Room

	This line ensures that the `Room` class from `room.py` is available to use within `adventure.py`, so we can create `Room` objects for use in the adventure game. For example, the following call would create a `Room` object for you to use in Adventure:

		Room(3, 'Inside building', 'You are inside a building, a well house for a large spring.')

	Note that we did not include an import statement atop `room.py`. This is because `Room` does not need to know anything about the adventure game, only the other way around.

2. The `Adventure` class, which contain all methods that make the game work.

	- The `load_rooms` method is used to parse the data files and creates `Room` objects with that data.

	- The `game_over` method will eventually decide if the game has been won or lost by the player.

	- Moving around in the game is handled by the `move` method. Here you'll make use of the room objects you've created earlier.

	- The `play` method contains the main loop that makes your game playable. One important part of this is translating commands given by your player into method calls that handle the actions. We've already given you a headstart here. See how we check if a command is a "direction" with which to move?

3. The `if __name__ == "__main__"` part

	This part is very short. We only include a line to create an `Adventure` object, which will start the game. Note, however, that this is the only place where you can change the name of the map that is loaded.


### `room.py`

This is your `Room` class, where al code related to rooms will be collected.
The `\\__init__()` method initializes each room with a given id, name and description.

So the call from earlier;

	Room(3, 'Inside building', 'You are inside a building, a well house for a large spring.')

creates a room with `id` = 3, `name` = 'Inside building' and `description` = 'You are inside a building, a well house for a large spring.'.

It also contains two methods, one for adding connections and one for checking connections, but you still have to implement them!


## Step 0: Reading data files and the code

### Parsing data files

`TinyRooms.txt`, the smallest version of the game, contains the following data:

	1
	Outside building
	You are standing at the end of a road before a small brick
	building.  A small stream flows out of the building and
	down a gully to the south.  A road runs up a small hill
	to the west.
	-----
	WEST     2
	UP       2
	NORTH    3
	IN       3

	2
	End of road
	You are at the end of a road at the top of a small hill.
	You can see a small building in the valley to the east.
	-----
	EAST     1
	DOWN     1

	3
	Inside building
	You are inside a building, a well house for a large spring.
	-----
	SOUTH     1
	OUT       1


These are the details for all rooms in the game and how to navigate between them. Each room consists of 4 parts:

	<id>
	<name>
	<description>
	---
	<routes>


A `<routes>` line contains both a `<direction>` and an `<id>`. For example, `Room 2 - End of road` is connected to room `1 - Outside building` by directions 'EAST' and 'DOWN'.

You'll find that the name is actually a short description of the room, whereas the 'description' is fully descriptive (Keep this in mind for later parts of the game!).
First we'll have to parse this data into our program. Since a room is a bit more complicated than a simple string or integer we have set you up with a class named Room in room.py.

In adventure.py you'll find the `load_rooms` method. We'll discuss how it is implemented shortly, but first let's see where it is used.

We call the `load_rooms` method in the `__init__()` of the Adventure class. We use it there because every time we create a game, we want it to at least have a list of rooms. Any data an Adventure class needs to function properly is initialized here.

Now let's look at `load_rooms` a little closer. We can divide the function into three parts.

#### Part 1

The first task to parsing the file is dividing the rooms into a list of data about each individual room. Looking at the datafiles we see that rooms are divided by a single newline. This means we can read lines from the file and save them to a list, until we reach that newline. That creates the follow list:


	[id, name, description, connection, ..., connection]

Then, when a newline is read, we save those lines we just read to another list. This creates a list of lists with each set of data about a room. That list looks like this:

	[[room_data], ..., [room_data]]


#### Part 2

Now that all rooms are loaded into memory and partitioned into separate lists, it is time to initialize each room into an object of our Room class.

For each set of room data we know that the first element is the `room_id`, the second element is its `name` and the third element is a `description`. We use those variables to make a `Room`, after wich we add that room to a dictionary of rooms using the room_id as the `key` for the `key`:`value` pair.

#### Part 3

To finalize the Room objects we'll have to update each room with its corresponding connections from the data file. Once again we iterate over each set of room_data, but this time we'll use the connections.

Since we don't know how many connections each room has, we'll have to slice till the end of each list. The start of each slice will be the fifth element of room data, that is the first element after the '`-----`' line.

The connections ar then cleaned up a by splitting them into two strings, one for the direction and another for the connected room_id. Finally we use the room_id from the room_data to find the correct room in the dictionary we created in step 2.

Here you'll find your first TODO.


## Step 1: Loading connections and moving around

First start with opening the correct datafile. Your program should accept a command line argument that is either "Tiny", "Small", or "Crowther" and no others.
Use that argument for the `game` variable used in the `if \\__name__ == '\\__main__'` statement. Make sure to return an exit code of 1 if the argument is incorrect!

Now it's up to you to add the connections to each individual room object.
To do so, you'll first have to head over to room.py and implement the add_route method for the `Room` class. You might need to add additional attributes to the `\\__init__()` for a proper implementation.

Then you can use that method to update each room with their respective connections.

. Parse a command line argument for the datafile filename.
. Implement the add_route method for the `Room` class.
. Add the correct routes to each respective room object.

### Moving around

Now that we have a couple of rooms, we can almost start playing the game.

First, implement the is_connected method in the `Room` class that can be used to see if the given command is a valid move. Use it to check if a given string has a connection for the room.

Next up is the `move()` method in `Adventure`. We've already initialized the `current_room` attribute in the `\\__init__()` and set it to the room with id 1.
Make sure to use and update that attribute in `move()` so you can keep track of where the player is.

Hint: If an attempted move is not a valid connection, let the player know they tried an "Invalid command".

You can test moving around by adding the following code to `if \\__name__ == '\\__main__'`:


	adventure.move("WEST") # should move to the 'room 2' object
	print(adventure.current_room) # should print room 2: "End of road"
	adventure.move("DOWN") # should move to the 'room 1' object
	adventure.move("IN") # should move to the 'room 3' object
	print(adventure.current_room) # should print room 3: "Inside building"

Make sure this works before going on!

- Implement `is_connected` for the `Room` class.
- Implement `move` in `Adventure`.
- Implement the `\\__str__` method for your `Room` class.

### Prompt for commands

Time for your first steps into making this a game; have players give commands.

Remove the code used to test moving around from the `if \\__name__ == '\\__main__'` and instead add `adventure.play()`.
Now when you run the script you should be met with a welcome message and be prompted for a command. But alas, not much happens when actually entering such a command!

Start with letting the player know where they are in the game.

Each time a player enters a room for the first time, we'll provide them with a full description of the room.
Following the description we'll prompt the player for a command. The '>' will mark this prompt. It should look like this:

	You are standing at the end of a road before a small brick
	building.  A small stream flows out of the building and
	down a gully to the south.  A road runs up a small hill
	to the west.
	>


Now allow the player to actually move around wit valid commands. Each time they press enter you have to parse their input and check whether they can move in the indicated direction.
Remeber not all users read the documentation. So be sure to allow for both UPPER and lower case commands.

If the player attempts a command that cannot be executed tell them they attempted an "Invalid command." and prompt for another command using the '>'.
Like so:

	> OUT
	Invalid command.
	>


If a player enters a room they've already seen, only give them the short description. How should we keep track of that?

Hint: Starting the game counts as 'entering' the first room! Make sure to provide a description of the starting location.

- Print the full description of a room each time a player enters a new room.
- Print only the name of a room when a player enters a room they've entered before.
- Allow a player to `move` using UPPER/lower cased commands from the terminal.
- Properly handle invalid movements.

### Additional commands

As a final step for making the basic game work, we'll add a few commands that make it easier to use: `QUIT`, `HELP` and `LOOK`.
Implement htem in the following way:

`HELP` prints instructions to remind the player of their commands and how to use them.
Have it behave as follows:

	> HELP
	You can move by typing directions such as EAST/WEST/IN/OUT
	QUIT quits the game.
	HELP prints instructions for the game.
	INVENTORY lists the item in your inventory.
	LOOK lists the complete description of the room and its contents.
	TAKE <item> take item from the room.
	DROP <item> drop item from your inventory.


`QUIT` lets the player stop the game. Print `Thanks for playing!` and terminate the program cleanly.


	> QUIT
	Thanks for playing!


`LOOK` prints a full description of the room the player is currently in, even if the room was visited earlier.


	Inside building
	> LOOK
	You are inside a building, a well house for a large spring.


- Implement the additional commands: `HELP`, `QUIT`, `LOOK`


## Step 2: forced movement

Sometimes a player will attempt a movement they cannot make yet, because they are missing the required item. Passing the steel grate in room 6 for example requires keys.
Instead of printing a custom message, we'll have the player move into a special sort of room. This room displays a description for what happened, and then forcefully moves the player to where the forced movement points.
This move happpens automatically and immediately after printing the full description.

Another example of forced movements can be found in the Crowther rooms 70 through 75. These rooms even have a conditional `FORCED` movement. These are the final few rooms, if all required items are owned the player will win the game and go to room 77. If not, the player continues to room 76 and has to try and find the remaining "`treasures`". The interesting part is that rooms 70 through 75 are devoid of a description! This makes it possible to have conditional movement based on 6 items, even though 5 extra rooms are required to do so.

This leaves you to implement this `FORCED` movement to the game. Luckily a room with `FORCED` movement can only contain that movement and no others. So you won't have to take anything else into account when a `FORCED` move is encountered!

Being `FORCED` moved looks like this:

	You are in a 25-foot depression floored with bare dirt.
	Set into the dirt is a strong steel grate mounted in
	concrete.  A dry streambed leads into the depression from
	the north.
	> DOWN
	The grate is locked and you don't have any keys.
	Outside grate
	> DOWN
	The grate is locked and you don't have any keys.
	Outside grate
	>


Remember to always print the full description when a room `FORCED` is entered. There's no room to `LOOK` around, since the player is immediately moved by the game.

Hint: You'll most likely want to do a check each time you move to a new room. And if there's a forced movement in the new room, take a good look around and follow the forced route.

- Update either `move` or the `play` loop in `Adventure` to 'forcefully' push players to the next room.
- Don't forget to print a description when a player enters and leaves a 'FORCED' `Room`.


## Step 3: The winner takes all

Now that you have implemented all the features of Adventures, Crowther game is finally playable.
But let's also make it winnable. For example, in the `CrowtherRooms.txt` file you can see that room 77 corresponds to victory.

Implement the win condition into your game and gracefully terminate the game after attaining victory.


## Optional: Abbreviations

Between the datafiles you can also find a `SmallSynomyms.txt` file. Within that file is a list of commands and single-letter abbreviated versions.

If you want to reduce time spent typing commands; parse the file and implement a way to also move or execute commands using their synonyms.


### Testing

Initially, check your progress by navigating through rooms manually. Just play the game!

Later, use `check50` to check for specific corner cases that you may not have found.

And finally, you may use the commands in this link:/course/problems/adventure-less/win.txt[text file] to single out spots where the checks may not give you enough information to proceed. These commands only work TODO.


### `check50`

	check50 minprog/cs50x/2019/adventure/less


### `style50`

	style50 adventure.py
	style50 room.py
