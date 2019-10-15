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

The adventure "map" is provided in a few **data files**, that contain room names and description, and in particular, information about which rooms are connected to other rooms, and using which commands.

Though Crowther originally wrote his game in Fortran, an imperative programming language that has been around since the 1950s, we will be taking a more modern approach to its implementation, using object-oriented programming (OOP). OOP is particularly suited to Adventure, because its main idea is a series of rooms that are connected. Each room will be an object, and all of these objects will point to each other.


## Specification

Implement an object-oriented version of Crowther's Adventure game using the class structure provided below. It should have the following parts:

1. implement **loading** of the map:
	* handling command line arguments to open a given datafile
	* loading map data into a series of objects
2. implement user **interaction**:
	* prompting the user for commands and execute those
	* warn about non-existent commands
	* moving the player from room to room
3. implement game **logic**:
	* forced movements
	* finishing the game


### Distribution

	$ cd ~/module9
	$ wget https://prog2.mprog.nl/course/problems/adventure-less/adventure.zip
	$ unzip adventure.zip
	$ rm adventure.zip
	$ cd adventure
	$ ls
	adventure.py  data/  room.py


## Understanding

### `data/`

The `data` directory contains data files with which you can create two versions of adventure:

- `TinyRooms.txt` is the smallest adventure game, consisting of 4 rooms. It's quite useful for testing purposes. You can see below that the file contains multiple "rooms", with a room id and descriptions. Below the line are "connections" that each point to one of the other rooms. The game player should be able to enter the names of those connections as commands (like "WEST").

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

		4
		Victory
		You have found the hidden well of winning a tiny game. Congratulations!
		-----
		FORCED    0

- `SmallRooms.txt` is a bit larger and includes more advanced interactions.


### `room.py`

This file contains the `Room` class definition. Each room contains basic descriptions, as well as links to the other rooms it is connected to in the game map. These are saved in a **dictionary**. The `connections` dictionary for a room might look like this:

	connections = {
		"WEST": <room.Room object at 0x7f325cbc4d68>,
		"EAST": <room.Room object at 0x7f325cbc4fd0>
	}

This means that the dictionary maps a **direction** (string) to a `Room` object.

For example, if we load the **Tiny** game map, the result should be that we have 4 objects in memory, all pointing to each other:

![](tiny.png)

1. The `__init__` method initializes a room with an id, name and description. For example, you should be able to create a `Room` object using the following syntax:

		room = Room(3, 'Inside building', 'You are inside a building, a well house for a large spring.')

2. The `add_connection` method allows us to connect the room to a new one. Given the `room` variable that we created in the previous step, we should be able to use it like this:

		some_other_room_object = Room(5, 'Outside', 'You are outside.')
		room.add_connection("WEST", some_other_room_object)

3. The `has_connection` method does nothing more than checking whether a connection is available in a particular direction. For example:

		print(room.has_connection("WEST"))  # should print True, given step 2 above

4. The `get_connection` method assumes the connection is there and returns the room object that may be found in that direction.

		new_room = room.get_connection("WEST")  # we should now have access to the "other" room

> A hard constraint in this program is that the `Room` class may not access (use) other classes. Its methods may only manipulate `self` and any access only objects that are passed to it as arguments to method calls.

### `adventure.py`

Take a look at `adventure.py`. The file has three main components.

1. The `import` statement. Instead of working from a single file, we've split our two classes into separate files, in order to keep our files relatively short and tidy. To be able to access the Room class from the `adventure.py` file, we use `import`.

2. The biggest part of the file is the `Adventure` class, which contain all methods that make the game work.

	- The `__init__` method ensures that all is set for playing an adventure game. In particular, it uses the other methods to load game data, build a structure of rooms, and point `current_room` to the first room in the map.

	- The `parse_rooms` method opens and parses the data files, and the `load_rooms` method creates `Room` objects with that data. We'll explain more in step 1, below.

	- The `game_over` method will eventually decide (calculate) if the game has been won or lost by the player.

	- Moving around in the game is handled by the `move` method, by setting the "current" room to a different one.

3. The `if __name__ == "__main__"` part, which contains the main "game loop" of the program. After introducing the game, it repeatedly asks for a command from the user, and tries to perform that command. In the `while` statement, you see that the game will stop as soon as the `game_over()` method returns `True`.

> A hard constraint in this program is that the `Adventure` class may not `print` anything, except in the `move` method. All other printing should be done in the `__main__` part. And in return, the `__main__` part may, aside from printing things, only call methods in the `Adventure` class. It may not access methods and/or attributes from the `Room class`.

## Step 0: Implement the Room class

Above, you have seen how the `Room` class is supposed to be *used* in code. We should be able to create rooms, and then connect different rooms to each other. The `__init__` method has already been implemented.

For you `TODO` are three other methods that manage connections. The main idea is to always use these methods to change or find connections, and never to access the `connections` dictionary from outside the `Room` class.

Implement the `add_connection`, `has_connection` and `get_connection` methods, recalling that the `connections` dictionary is a map between directions like "EAST" and room objects.

	connections = {
		"WEST": <room.Room object at 0x7f325cbc4d68>,
		"EAST": <room.Room object at 0x7f325cbc4fd0>
	}

1. In `add_connection` you need to add an item to the `connections` dictionary. How do you add items to a dictionary?

2. In `has_connection` you need to check whether there is an item in the dictionary with a particular key, like "EAST". How do you check if a dictionary contains some key?

3. In `get_connection` you need to retrieve an item from the dictionary, given a particular direction. How do you retrieve items in a dictionary?

After implementing, you might test the class by starting Python and creating `Room` objects:

	$ python -i room.py
	>>> r1 = Room(1, "Room 1", "Description 1")
	>>> r2 = Room(2, "Room 2", "Description 2")
	>>> r2.add_connection("WEST", r1)
	>>> r2.has_connection("EAST")
	False
	>>> r2.has_connection("WEST")
	True
	>>> r2.get_connection("WEST")
	<__main__.Room object at 0x7f36f65076a0>

In that last line, you find the Python description of a `Room` object, along with its assigned memory address. Seems to work! (The address on your computer will most likely be different.)

Be sure to test manually like above!


## Step 1: Reading data files and the code

Take another look at the data in `TinyRooms.txt`. Each room has 4 parts:

	<id>
	<name>
	<description>
	-----
	<route1>
	<route2>
	...

Any `<routeX>` line contains both a `<direction>` and an `<id>`. For example, room 2 is connected to room 1 by directions `EAST` and `DOWN`. **Parsing** is the task of loading this information into memory in a useful way. 


### Reading the data files

In `adventure.py`, you're provided with the `parse_rooms` method, wich loads all lines from a data file, and puts it into a list of data:

	[[room_data], ..., [room_data]]

And each [room_data] list might look like this:

	[id, name, description, connection, ..., connection]

> We have provided this method in full because the format of the adventure data files is not ideally suited for Python. In modern applications, you might have data files in CSV or JSON format. You'll learn more about those in future assignments.


### Creating rooms

The "list of lists" from the previous step is a temporary structure, which is passed on to the `load_rooms` method. This method is going to create the "map" of rooms in memory.

The first phase is creating each room. For each list of room data we know that the zeroth element is the `room_id`, the first element is its `name` and the second element is a `description`. We use those variables to make a `Room`, which is immediately placed into the `self.rooms` dictionary. We use a dictionary so we can make a mapping between a room number and the `Room` object, which in turn allows us to look up a room object by its ID.

This means that we should now have a fully initialized `self.rooms` dictionary:

	self.rooms = {
		1: <room.Room object at 0x7f325cbc4d68>,
		2: <room.Room object at 0x7f325cbc4fd0>
	}


### Making connections

The second phase of `load_rooms` is making connections. Because all rooms have now been created, we can loop through the data again and make each connection (can you think of the reason that we have to do this in two phases?).

Take a look at the code for phase 2. The connections are again retrieved from the `room_data`. The first three elements were room descriptions that we have already used, the next element is a separator line, and the remaining elements are indeed connections.

So it's these elements that we extract from the list using a slice operator (`[4:]`), then loop through each element, **split** the text into a direction and a room number (of the room that the connection will be made to).

And that's where you'll find your next `TODO`. At that point in your program, you have enough information to make a connection. Use the `add_connection` method from `Room` to create them. The are a couple of fine points that you have to figure out when doing this, but it shouldn't be more than a couple of lines of code!

- One particular fine point is that the final "winning" room is indicated by a connection to a room 0 (which does not exist). So for now, if you encounter a connection to 0, skip over it.


## Step 2: Moving around

### Implement `move`

Now that we have a couple of rooms, we can start implementing the game itself. The most basic function of this game is moving around between rooms. Remember that the `Adventure` class has a variable that keeps track of the "current room" for the game. It also has a still-empty `move` method that's supposed to set the current room to a new one.

The `move` method has one parameter, `direction`, which should let you lookup (via the `current_room`) which room we're going to move on to. Just set `current_room` to that room and you're done.

- `move` should return True or False depending on whether the move was possible. The main program can use this result to notify the user if the move could not be performed.


### Prompting for commands

Now check out `if __name__ == '__main__'` at the bottom of `adventure.py`. Currently, if you run `adventure.py`, you will be shown the description of the first room, you can even enter commands, but nothing will happen as you enter them.

We're going to support a few different commands, but first of all, let's allow your use to move around in the game using directions like "IN" or "WEST".

- Start by passing the entered `command` to the adventure class's `move` method.

- Modify the program to display the room description after each command, so it feels like moving around in the map. (Currently a description is only printed once, at the start of the game.)

- Following the description we'll again prompt the player for a command. The '>' will mark this prompt. It should look like this:

		You are standing at the end of a road before a small brick
		building.  A small stream flows out of the building and
		down a gully to the south.  A road runs up a small hill
		to the west.
		>

- Not all users read the docs! Be sure to allow for both UPPER and lower case commands.

- If the player attempts a command that cannot be executed tell them they attempted an "Invalid command." and prompt for another command using the '>'.

		> OUT
		Invalid command.
		>

You should now be able to pass an extra test in `check50`.


## Step 3: Short and long descriptions?

If a player enters a room they've already seen, only give them the short description. How should we keep track of that?

- First, add a new attribute to `Room.__init__`: self.visited. It should probably be False when the room is first initialized.

- Then, add a `set_visited()` method to `Room`, which marks it as visited. Also, add a `get_visited()` method, which returns False or True depending on the current state of the room.

- Having done that, you can change `Adventure`'s `move` method to set a room to visited **right before moving to another room**. Use the new `set_visited` method to do that.

- And finally, you can now use `get_visited` in `Adventure.get_description` to return either the room `name` or the room `description`, depending on whether it was visited before.


## Step 4: Additional commands

As a final step for making the basic game work, we'll add a few commands that make it easier to use: `QUIT`, `HELP` and `LOOK`. Implement these in the following way:

-   `HELP` prints instructions to remind the player of their commands and how to use them. It should behave as follows:

		> HELP
		You can move by typing directions such as EAST/WEST/IN/OUT
		QUIT quits the game.
		HELP prints instructions for the game.
		LOOK lists the complete description of the room and its contents.

-   `QUIT` lets the player stop the game. Print `Thanks for playing!` and terminate the program cleanly.

		> QUIT
		Thanks for playing!

-   `LOOK` prints a full description of the room the player is currently in, even if the room was visited earlier.

		Inside building
		> LOOK
		You are inside a building, a well house for a large spring.


## Step 5: Try `SmallRooms`

Before continuing, make sure your program still works if you transition from the **Tiny** map to the **Small** map! You'll need it for the next part.


## Step 6: Forced movement

Sometimes a player will attempt a movement they cannot make. For example, in the Small adventure, when going WEST from the "Outside grate" room (6), one finds oneself at the edge of an "unpassable stream". The only way is going back the "Outside grate" room.

The adventure game has a special feature called `FORCED` movements. If a player enters a room that has a direction named `FORCED`, the full room description will be printed, but then the user will be immediately moved back to the connected room.

- You'll most likely want to do a check each time you move to a new room. If there's a `FORCED` connection in the new room, take a good look around and follow the forced route.

- As you're going to have to print the description, handle this in the main game loop and not in the `move` method!


## Step 7: The winner takes all

Now that you have implemented all the features of Adventure, your game should be fully playable. What's left is to make the game winnable. As you might recall from earlier, a "winning" room is indicated by having a `FORCED` connection to room 0 (which does not exist).

To implement winning, you'll have to:

- Change the `Room` class to add an attribute that indicates it's a "winning" room. Also add methods to set this attribute (called `set_winning`) and to request it (called (`is_winning`)).

- Change the phase 2 algorithm in `load_rooms` to set a room to "winning" as soon as it encounters a `FORCED` connection to room 0.

- Change the main game loop to make use of this new information. It should congratulate the user and gracefully terminate the game.


## Step 8: Check your work

Have a good look at the constraints noted earlier:

- A hard constraint in this program is that the `Room` class may not access (use) other classes. Its methods may only manipulate `self` and any access only objects that are passed to it as arguments to method calls.

- A hard constraint in this program is that the `Adventure` class may not `print` anything, except in the `move` method. All other printing should be done in the `__main__` part. And in return, the `__main__` part may, aside from printing things, only call methods in the `Adventure` class. It may not access methods and/or attributes from the `Room class`.

- Watch out! You should hand in your program with the "Small" adventure loaded (atop the main). The checks depend on this particular version of Crowther's Adventure.


## Step 9: Synonyms

Step 9 doesn't exist. This was Adventure!


### `check50`

	check50 minprog/cs50x/2019/adventure/less


### `style50`

	style50 adventure.py
	style50 room.py
