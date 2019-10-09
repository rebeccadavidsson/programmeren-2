# Hangman

## tl;dr

Implement a program that allows someone to play Evil Hangman against the computer.

	$ python hangman.py
	WELCOME TO EVIL HANGMAN >=)
	I have a word in my mind of 8 letters.
	Guess a letter: a
	That's not in the word >=)
	Guess a letter: n
	It's in the word :(
	_____N__
	Guess a letter: ...

## Background

It's hard to write computer programs to play games. When we as humans sit down to play a game, we can draw on past experience, adapt to our opponents' strategies, and learn from our mistakes. Computers, on the other hand, blindly follow a preset algorithm that (hopefully) causes them to act somewhat intelligently. Though computers have bested their human masters in some games, most notably chess and go, the programs that do so often draw on hundreds of years of human game experience and use extraordinarily complex algorithms and optimizations to out-calculate their opponents.

While there are many viable strategies for building competitive computer game players, there is one approach that has been fairly neglected in modern research --- cheating. Why spend all the effort trying to teach a computer the nuances of strategy when you can simply write a program to play dirty and win handily all the time? In this assignment, you will build a mischievous program that bends the rules of Hangman to trounce its human opponent time and time again. In doing so, you'll cement your skills with abstract data types and iterators, and will hone your general programming savvy. Plus, you'll end up with a piece of software which will be highly entertaining. At least, from your perspective.

In case you aren't familiar with the game Hangman, the rules are as follows:

1. One player chooses a secret word, then writes out a number of dashes equal to the word length.
2. The other player begins guessing letters. Whenever she guesses a letter contained in the hidden word, the first player reveals each instance of that letter in the word. Otherwise, the guess is wrong.
3. The game ends either when all the letters in the word have been revealed or when the guesser has run out of guesses.

Fundamental to the game is the fact the first player accurately represents the word she has chosen. That way, when the other players guess letters, she can reveal whether that letter is in the word. But what happens if the player doesn't do this? This gives the player who chooses the hidden word an enormous advantage. For example, suppose that you're the player trying to guess the word, and at some point you end up revealing letters until you arrive at this point with only one guess remaining:

	DO–BLE

There are only two words in the English language that match this pattern: "doable" and "double." If the player who chose the hidden word is playing fairly, then you have a fifty-fifty chance of winning this game if you guess 'A' or 'U' as the missing letter. However, if your opponent is cheating and hasn't actually committed to either word, then there is no possible way you can win this game. No matter what letter you guess, your opponent can claim that she had picked the other word, and you will lose the game. That is, if you guess that the word is "doable," she can pretend that she committed to "double" the whole time, and vice-versa.

Let's illustrate this technique with an example. Suppose that you are playing Hangman and it's your turn to choose a word, which we'll assume is of length four. Rather than committing to a secret word, you instead compile a list of every four-letter word in the English language. For simplicity, let's assume that English only has a few four-letter words, all of which are reprinted here:

	ALLY  BETA  COOL  DEAL  ELSE  FLEW  GOOD  HOPE  IBEX

Now, suppose that your opponent guesses the letter 'E.' You now need to tell your opponent which letters in the word you've "picked" are E's. Of course, you haven't picked a word, and so you have multiple options about where you reveal the E's. If you'll notice, every word in your word list falls into one of five "word families:"

- `----`, which contains the word `ALLY`, `COOL`, and `GOOD`.
- `-E--`, containing `BETA` and `DEAL`.
- `--E-`, containing `FLEW` and `IBEX`.
- `E--E`, containing `ELSE`.
- `---E`, containing `HOPE`.

Since the letters you reveal have to correspond to some word in your word list, you can choose to reveal any one of the above five patterns. There are many ways to pick which pattern to reveal – perhaps you want to steer your opponent toward a smaller family with more obscure words, or toward a larger family in the hopes of keeping your options open. In this assignment, in the interests of simplicity, we'll adopt the latter approach and always choose the largest of the remaining word families. In this case, it means that you should pick the pattern `----`. This reduces your word list down to

	ALLY  COOL  GOOD

and since you didn't reveal any letters, you would tell your opponent that her guess was wrong.

Let's see a few more examples of this strategy. Given this three-word word list, if your opponent guesses the letter `O`, then you would break your word list down into two families:

- `-OO-`, containing COOL and GOOD.
- `----`, containing ALLY.

The first of these families is larger than the second, and so you choose the pattern `-OO-`, revealing two O's in the word and reducing your list down to

	COOL GOOD

But what happens if your opponent guesses a letter that doesn't appear anywhere in your word list? For example, what happens if your opponent now guesses 'T'? This isn't a problem. If you try splitting these words apart into word families, you'll find that there's only one family --- the one with the pattern `----` in which T appears nowhere and which contains both COOL and GOOD. Since there is only one word family here, it's trivially the largest family, and by picking it you'd maintain the word list you already had.

There are two possible outcomes of this game. First, your opponent might be smart enough to pare the word list down to one word and then guess what that word is. In this case, you should congratulate her --- that's an impressive feat considering the scheming you were up to! Second, and by far the most common case, your opponent will be completely stumped and will run out of guesses. When this happens, you can pick any word you'd like from your list and say it's the word that you had chosen all along. The beauty of this setup is that your opponent will have no way of knowing that you were dodging guesses the whole time --- it looks like you simply picked an unusual word and stuck with it the whole way.

## Specification

Your assignment is to write a computer program which plays a game of Hangman using this "Evil Hangman" algorithm. In particular, your program should do the following:

1. Read the file `dictionary.txt`, which contains the full contents of the Official Scrabble Player's Dictionary, Second Edition. This word list has over 120,000 words, which should be more than enough for our purposes.

2. Prompt the user for a word length, reprompting as necessary until she enters a number such that there's at least one word that's exactly that long. That is, if the user wants to play with words of length -42 or 137, since no English words are that long, you should reprompt her.

3. Prompt the user for a number of guesses, which must be an integer greater than zero. Don't worry about unusually large numbers of guesses – after all, having more than 26 guesses is clearly not going to help your opponent!

4. Prompt the user for whether she wants to have a running total of the number of words remaining in the word list. This completely ruins the illusion of a fair game that you'll be cultivating, but it's quite useful for testing (and grading!)

5. Play a game of Hangman using the Evil Hangman algorithm, as described below:

	1. Construct a list of all words in the English language whose length matches the input length.

	2. Print out how many guesses the user has remaining, along with any letters the player has guessed and the current blanked-out version of the word. If the user chose earlier to see the number of words remaining, print that out too.

	3. Prompt the user for a single letter guess, reprompting until the user enters a letter that she hasn't guessed yet. Make sure that the input is exactly one character long and that it's a letter of the alphabet.

	4. Partition the words in the dictionary into groups by word family.

	5. Find the most common "word family" in the remaining words, remove all words from the word list that aren't in that family, and report the position of the letters (if any) to the user. If the word family doesn't contain any copies of the letter, subtract a remaining guess from the user.
	
	6. If the player has run out of guesses, pick a word from the word list and display it as the word that the computer initially "chose."

	7. If the player correctly guesses the word, congratulate her.

	   Ask if the user wants to play again and loop or exit accordingly.

Your program will consist of three major parts.

1. The Lexicon class: Lexicon objects are used to retrieve words for the game from a dictionary.

2. The Hangman class: a *Hangman* object will include all of the logic needed to play the Evil Hangman game. It will keep track of the current status of the game, and it will be able to update the status of the game when a letter is guessed. However, a Hangman object will not directly interact with the user (the person playing the game).

3. The user interface: this is a piece of code that interacts with the user. It displays messages to the user about the game, and prompts her for new guesses. This piece of code will use the Hangman class to keep track of the game itself.

For the Lexicon and Hangman classes, we will prescribe how they should work, like in previous assignments. They can be checked with `check50`. For the user interface, you have some freedom, but be careful to stick to the specification.

Finally, here are some general tips and tricks that might be useful:

- Letter position matters just as much as letter frequency. When computing word families, it's not enough to count the number of times a particular letter appears in a word; you also have to consider their positions. For example, "BEER" and "HERE" are in two different families even though they both have two E's in them. Consequently, representing word families as numbers representing the frequency of the letter in the word will get you into trouble.

- Watch out for gaps in the dictionary. When the user specifies a word length, you will need to check that there are indeed words of that length in the dictionary. You might initially assume that if the requested word length is less than the length of the longest word in the dictionary, there must be some word of that length. Unfortunately, the dictionary contains a few "gaps." The longest word in the dictionary has length 29, but there are no words of length 27 or 26. Be sure to take this into account when checking if a word length is valid.

## Steps

### 0. Before you get started

Testing for this assignment will again work automatically. Because you are going to write classes, it is now possible to test each class's functionality separately from the other parts of the program.

### 1. The `Lexicon` class

The first thing to implement is a class called `Lexicon`, which has the responsibility of managing the full word list and extracting words of a given length. It can be loaded once and asked for words whenever a new game is started.

Download the lexicons via:

	cd ~/module8
	wget https://prog2.mprog.nl/course/problems/hangman/dictionaries.zip
	unzip dictionary.zip

Create a file called `hangman.py` and add a `Lexicon` class. This class should have two methods: `__init__()` to initialize, and `get_words()` to extract a list of words with a  specific length to play Hangman:

    import random
    class Lexicon:

        def __init__(self):
            self.words = []
            # Load the dictionary of words.
            # TODO

        def get_words(self, length):
            # Return a list of all words from the dictionary of the given length.
            # TODO

        def get_word(self, length):
            # Return a single random word of given length. Uses `get_words` above.
            return random.choice(self.get_words(length))

In our code, we will use a **list** to store the master word list. That's why we have `self.words = []` at the top of the initializer method.

Now, implement those two methods.

> Note that the loading of words was demonstrated in last week's [Python lecture](/lectures/python)! It uses a **set** to store words, but you can modify it to use a list instead. Recall how to add items to a list?

### 2. Testing the `Lexicon`

Before we move on to the next step, we want to test if the class is working correctly. For example, try to get words of length 8 and see if the result seems reasonable. Start Python *interactively* using:

	python -i hangman.py

If you followed step 0 correctly, it should just load the Lexicon class and do nothing else. Then you could try some of the following.

	lex = Lexicon()
	lex.get_word(8)
	lex.get_word(8)
	lex.get_word(8)

Check if everything is in order. Are the words reasonable, i.e., of the right length? Do you get a different word each time? If something is still wrong, consider testing `get_words()` specifically, instead of `get_word()`. For example, if you `print(lex.get_words(2))`, are all printed words of length 2?

> You should not put testing code like the above in `hangman.py` like you might have done in earlier assignments. This is because `check50` should be able to load your program and perform its own tests. Your tests would interfere with the checks.

You can now test using `check50` for the first time!

### 3. The `Hangman` class

So now we have a class to manage the word list. We can also create a class that manages playing a game of Hangman. Let's think about what is needed to "play" a game.

-   First of all, a game is played based on a particular word length. Also, we decide upfront how many guesses will be allowed. These two are the only pieces of information that a Hangman game object needs to get started. This means that we know how it may be eventually initialized:

        game = Hangman(length=8, num_guesses=5)

-   Second, we need a method to enter a guess. Maybe we'd like to try the letter A:

		game.guess("a")

So that specifies how the class should work when testing. Here's a minimal skeleton, which you should copy-paste below your `Lexicon` class.

    class Hangman:
        def __init__(self, length, num_guesses):
            # Initialize game by choosing a word and creating an empty pattern.
            self.length = length
            self.num_guesses = num_guesses
            TODO

        def guess(self, letter):
            # Update the game for a guess of letter. Return True if the letter
            # is added to the pattern, return False if it is not.
            TODO

        def __str__(self):
            # Return a nice version of the pattern, for printing.
            TODO

To implement it, consider the following:

-  For the initializer:

    1.  you need to create a variable to contain the random word. You should instantiate a Lexicon object and ask it for that word.

    2.  you need a variable to hold the "pattern" of guessed letters. Initially, this should simply be an array of N underscores, where N is the chosen word length. For example, if the length is 4, the empty pattern should be the array ["_", "_", "_", "_"].

    3.  you need a variable to hold the letters that have been guessed. For the same reason as above, you should use an array, adding guessed letters as the game progresses.

        > We choose an array here instead of a string because the array is **mutable**. We would like to add guessed letters to the pattern as the game progresses, so we need to mutate (change) the array.

- For the `guess` method:

    1.  you need to add the letter to the list of "guessed" letters, so we keep track of all earlier guesses.

    1.  you need to see if the chosen letter occurs somewhere in the random word. The letter may occur more than once! Wherever it occurs, that letter should replace the `_` in the pattern.

    2.  you need to make sure that you `return` either True or False depending on whether the letter was indeed found in the word (at least once).

- For the `__str__` method:

    1.  you need to create a string version of the pattern, because it's an ugly list. When printing, it should be nicely readable. In this case, feel free to use an internet search for [list to string python](https://duckduckgo.com/?q=list+to+string+python).


### 4. Testing the `Hangman` game

Let's test our game logic. We should be able to start a new game, and repeatedly guess letters. Again, run

    python -i hangman.py

and enter the following commands, or a variation thereof:

	game = Hangman(8, 6)
	game.guess("e")
	print(game)
    game.guess("a")
	print(game)
    print(game.finished())
    print(game.consistent_word())
    print(game.pattern())
    game.guess("o")
    game.guess("i")
    game.guess("u")
    print(game.pattern())
    print(game)


### 5. Winning

A user should be able to win or lose the game, and our computer version should be able to notice if a game has been won or lost. Let's add a few methods to the `Hangman` class:

    def won(self):
        # Return True if the game is finished and the player has won, 
        # otherwise False.
        pass

When has the game been won? Think about it. You should be able to program this method without introducing new `self` variables, but instead, calculating if the game has been won by checking out the letters in `self.guessed`.

    def lost(self):
        # Return True if the game is finished and the player has lost, 
        # otherwise False.
        pass

And for this method the same applies: you should be able to calculate if the game has been lost by checking the number of previously guessed letters, and comparing with `self.num_guesses`.

    def finished(self):
        # Return True if the game is finished, otherwise False.
        pass

Finally, `finished` is a neutral function. It may be used to check if any more input is allowed. If the game has either been lost or won, it is also finished. So feel free to use the `lost()` and `won()` methods to decide!


### 6. Testing the `Hangman` game again

Let's test our game logic. We should be able to start a new game, and repeatedly guess letters. Again, run

    python -i hangman.py

and enter the following commands, or a variation thereof:

	game = Hangman(8, 6)
	game.guess("e")
	print(game)
    game.guess("a")
	print(game)
    game.guess("o")
    game.guess("i")
    game.guess("u")
    print(game)


### Intermezzo: handling exceptions

What happens when you want to create a Hangman game that does not follow the specifications? For example, what should happen if someone uses your class like the following:

	game = Hangman(-5, 6)

Try it yourself! Most likely, your code will indeed try to create a hangman game with a word of length -5. But that is not going not work, ever.

Because the `Hangman` object does not interface directly with the user, it makes no sense for it to reprompt the user for new input. However, we still need to be able to signal when something is wrong.

When you write code that does not interface with a person, you should still be able to deal with faulty input. To do so, Python (and many other languages) uses *exceptions*. Exceptions are like a warning signal: "Something went wrong! I'm stopping here!"

Try running the following Python code to see an exception being created:

    my_list = [1, 2, 3, 4]
    my_value = my_list[5]

There is no element in the list at index 5, so something goes wrong. You should get a message about an `IndexError`, which is indeed an "exception". It makes no sense to continue computation, because we don't know what the value of `my_value` should be. So the program halts.

In order for your Hangman class to be easy to work with, it should deal with wrong inputs using exceptions, instead of just going on. This means that running `game = Hangman(-5, 6)` should cause some kind of exception. For example:

    class Hangman:
        def __init__(self, length, num_guesses):
            if length < 1:
                raise Exception("Hangman word needs to have positive length.")
            if num_guesses < 1:
                raise Exception("You need at least one guess to play Hangman.")
            # ... and here follows other code.

Note that `check50` for this problem expects that such checks, with accompanying exceptions, are present in your code!


### 5. Implementing user interaction

While the `Hangman` class has all you need to play Hangman, someone who does not know your program won't understand that you have to write things like `game = Hangman(8, 6)` to start a game and `game.guess("e")` to guess a letter. So, let's make a **user interface**.

Your user interface should at least:

1. Prompt the user for how many letters the Hangman word should have. If the input is not a positive integer, or there is no word with that many letters, repeat the prompt until you get correct input.

2. Prompt the user for how many guesses she should get until she loses. This should be a positive integer.

3. Prompt the user for whether she wants to see detailed statistics of the game while playing (the statistics you put into the `__str__` method).

4. Play the game: repeatedly do the following

    1. Prompt the user for a guess. The guess should be a single letter that
       has not yet been guessed.

    2. Show an updated pattern, and the number of guesses remaining.

    3. Show detailed game statistics, if she asked for those.

    4. If the game has finished, either congratulate the player (on a win), or
       tell the player the Hangman word (any word that is consistent with the
       current pattern). Then ask the player if she wants to play again.

Like in "Game of Cards", the game code should be added inside an `if __name__ == '__main__':` condition. This ensures that that code will not run when checking, but will run when you test the program yourself using `python hangman.py`.

> Note that the program shown in the introduction at the top of the assignment is not a valid solution; it is just an illustration.

## Testing

	check50 minprog/cs50x/2019/hangman

## Extensions

The algorithm outlined in this handout is by no means optimal, and there are several cases in which it will make bad decisions. For example, suppose that a player has exactly one guess remaining and that computer has the following word list:

	DEAL   TEAR   MONK

If the player guesses the letter 'E' here, the computer will notice that the word family for the pattern `-E--` has two elements and the word family for the pattern `----` has just one. Consequently, it will pick the family containing DEAL and TEAR, revealing an E and giving the player another chance to guess. However, since the player has only one guess left, a much better decision would be to pick the family `----` containing MONK, causing the player to lose the game.

There are several other places in which the algorithm does not function ideally. For example, suppose that after the player guesses a letter, you find that there are two word families, the family `--E-` containing 10,000 words and the family `----` containing 9,000 words. Which family should the computer pick? If the computer picks the first family, it will end up with more words, but because it revealed a letter the player will have more chances to guess the words that are left. On the other hand, if the computer picks the family `----`, the computer will have fewer words left but the player will have fewer guesses as well. More generally, picking the largest word family is not necessarily the best way to cause the player to lose. Often, picking a smaller family will be better.

After you implement this assignment, take some time to think over possible improvements to the algorithm. You might weight the word families using some metric other than size. You might consider
having the computer "look ahead" a step or two by considering what actions it might take in the future.

If you implement something interesting, make sure to document your partition method well by describing your changes in detail!

## Submitting

To submit this assignment, you need to submit all of your source files. If you have created your own evil algorithm (see the section Extensions), then include a short description in a text file of what you've written.
