+++
title = "Things I Learned During Advent of Code 2017"
description = ""
tags = [
    "programming",
    "python",
    "golang",
    "2017"
]
date = "2018-01-06"
categories = [
    "Programming",
]
menu = "main"
draft = false
+++

Over the month of December I participated in an advent calendar game called the [Advent of Code](http://adventofcode.com/). Each December from the 1st to the 25th, a new programming puzzle is dished out at 12:00AM EST for you to solve. There's a bit of a metagame involved where competitive programmers try to race to get to solutions as fast as possible, but it's still totally fun and a worthwhile set of programming exercises even if you don't have the stamina to _start_ programming at midnight. I've catalogued a few things I learned over the month in the post below.
<!--more-->

### Input Parsing Is Very Easy In Python

AoC is structed such that although every participant's puzzles are the same, his or her inputs will vary. Depending on the puzzle, these inputs can be something as simple as an integer or string of characters. In more complicated puzzles though, they are often used to used to represent things like [graphs in day 12](http://adventofcode.com/2017/day/12):

    24 <-> 244, 624
    25 <-> 25
    26 <-> 18, 490, 545, 1123, 1958

Positions, velocities, and accelerations like in [day 20](http://adventofcode.com/2017/day/20):

    p=<5528,2008,1661>, v=<-99,-78,-62>, a=<-17,-2,-2>
    p=<3088,2748,-1039>, v=<-103,-136,94>, a=<-5,0,-4>
    p=<628,-1052,161>, v=<20,33,-92>, a=<-5,2,8>

Or maps such as in [day 21](http://adventofcode.com/2017/day/21):

    ../.. => .#./###/##.
    #./.. => ..#/.#./#.#

Python makes reading these sorts of inputs and tossing them into a data structure trivially easy. Although you'll likely need a regular expression for something like the positioning information above, the ability to carelessly shove your data into a dictionary or list without having to give a lot of thought to types or structs makes blasting through the boilerplate an absolute breeze. For the day 12 example above, creating the data structure representing our map is as simple as splitting our comma-separated value string into a list and inserting it into the dict:

    nodes = dict()
    with open("input1.txt") as f:
        for row in f:
            from_ = row.split()[0]
            to = ''.join(row.split()[2:]).split(',')
            nodes[from_] = to

For some puzzles, like [day 5](http://adventofcode.com/2017/day/5), parsing your input can be as easy as writing a simple list comprehension:

    with open("input1.txt") as f:
        maze = [int(x) for x in f]

None of these are ground-breaking techniques, but the ability to skip past the busy work and get to the meat of the puzzle was something that I really appreciated for the few puzzles I worked on in Go and Rust.


### Puzzles To "Scale"

During the latter half of AoC, many of the puzzles started playing with the idea of running millions or billions of iterations. Part 2 of [day 16](http://adventofcode.com/2017/day/16) requests that you iterate the first part of your code a billion times, part 2 of [day 17](http://adventofcode.com/2017/day/17) requests 50 million, [day 22](http://adventofcode.com/2017/day/17) wants 10 million, and [day 13](http://adventofcode.com/2017/day/13), rather secretly, will take you in the neighborhood of 4 million iterations before you find a result.

Before I caught on to this pattern, it was convenient excuse to just assume that Python was a relatively slow language and that I needed to re-write my solution in Go. This unfortunately was almost always a waste of time. Although Python is definitely slower, recognizing sooner that the puzzles are specifically asked in a way such that you _need_ to re-think your approach would have saved me a few hours in total. For an example, I was pretty proud of my solution to [day 17](http://adventofcode.com/2017/day/17). The first part wants you to manage a list over ~2000 iterations like so:

    spinlock = list([0])
    for x in range(2017):
        index = ((index + step) % len(spinlock)) + 1
        spinlock.insert(index, x+1)

The second part bumps this up to 50 million iterations, but notes that it really only cares about the item in the first index. Recognizing that this fundamentally changes the puzzle means that you can toss out what you just wrote and write a second, totally-different loop:

    for x in range(50000000):
        index = (index + step) % (x+1)
        if index == 0:
            to_return = x+1
        index += 1

We're not even using a data structure anymore! Next year I might just take a mandatory break when I long numbers in the second part of question; tossing our your expectations and approaching the problem from a fresh perspective is surprisingly powerful.


### Generators In Python Are Pretty Slick

[Day 13](http://adventofcode.com/2017/day/13)'s firewall problem was the first one to jump out at me as being a natural fit for creating a generator. I recognized the pattern as being pretty damn close to the out-of-the-box itertools [cycle](https://docs.python.org/3/library/itertools.html#itertools.cycle), but unfortunately itertools doesn't have any built-in support for an oscillating pattern. I took a stab at building my own oscillating iterator by mashing together a forwards cycle and a backwards cycle which was messy but did the job:

    def oscillate(iterable):
        # cycle('ABCD') --> A B C D C B A B C D ...
        reverse = False
        first = iterable[0]
        last = iterable[len(iterable)-1]
        iterator = cycle(iterable)
        reverse_iterator = cycle(iterable[::-1])
        while True:
            if reverse:
                to_return = next(reverse_iterator)
                if to_return == first:
                    reverse = False
                    next(iterator)
                yield to_return
                continue
            if not reverse:
                to_return = next(iterator)
                if to_return == last:
                    reverse = True
                    next(reverse_iterator)
                yield to_return
                continue

Although not a groundbreaking discovery, my efforts to get familiar with generators paid off handsomely a couple days later when [day 15](http://adventofcode.com/2017/day/15) screamed as loud as possible for participants to write generators to solve it. This was probably one of the easiest days of the event since I had just wrote a generator that was far more complicated. I was even able to solve the second part of the question simply by adding a conditional.

    def picky_generator(value, factor, multiple):
        while True:
            value = value * factor
            value = value % 2147483647
            if value % multiple == 0:
                yield value


### I Do Not Like Python For Recursive And Structural Questions

Python is a blast to write when you're able to ignore scope and can use the built-in data structures, but becomes more error-prone when you want to do more "formal" things like specifically pass values or references. I've also had a few bad experiences with `copy` vs `copy.deepcopy`, and the fact that mutable objects are accessible globally is also potentially dangerous and a bit spooky to write.

(As an aside, [this blog post](https://jeffknupp.com/blog/2012/11/13/is-python-callbyvalue-or-callbyreference-neither/) is a pretty good rundown of some of the trickiness inherent in how Python binds variable names. The comments arguing that the author is wrong for opposite reasons are wonderful.) 

I reworked a few of AoC's more complicated questions in Go a handful of times, but only [day 24](http://adventofcode.com/2017/day/24) jumped out to me as one that I knew I wanted a staticly-typed language from the start. I created a super simple struct to represent our bridge parts:

    type Component struct {
	    PinA int
        PinB int
    }

and then made a clean, organized, recursive function with `maxStrength` passed by reference:

    func buildBridge(componentList []Component, pin int, size int, maxStrength *int) {
        // get list of possible components
        var options []Component
        for _, element := range componentList {
            if element.PinA == pin || element.PinB == pin {
                options = append(options, element)
            }
        }

        if len(options) == 0 {
            if size > *maxStrength {
                *maxStrength = size
            }
            return
        }
        // for each of those, continue building
        for _, element := range options {
            newSize := size + element.PinA + element.PinB
            remainingComponents := make([]Component, len(componentList))
            copy(remainingComponents, componentList)
            remainingComponents = findAndRemove(remainingComponents, element)
            var newPin int
            if element.PinA == pin {
                newPin = element.PinB
            } else {
                newPin = element.PinA
            }
            buildBridge(remainingComponents, newPin, newSize, maxStrength)
        }
    }

In retrospect, it would likely have been very possible to use a Python `tuple` to represent my bridge pieces, but specifically being able to identify each side as `pinA` or `pinB` seems cleaner. I could also have set `maxStrength` as a `global` instead of passing a reference around. Lastly, although Python can certainly copy lists just like any other language, I've learned to trust Go's `copy` far, far more that Python's implementations.

### Lastly, One Neat Trick For Representing Multidimensional Arrays (Puzzlers Hate It!)

Grid-based puzzles are almost always more arduous than they need to be. Working in multiple dimensions is generally a pain in the ass, and is particularly awful in a scenario where your grid needs to be resized. Early on in AoC, I solved these sorts of problems in the standard way:
    
    while(True):
        spiral[x,y] = spiral[x-1,y-1]+spiral[x,y-1]+spiral[x+1,y-1]+spiral[x-1,y]+spiral[x+1,y]+spiral[x-1,y+1]+spiral[x,y+1]+spiral[x+1,y+1]
        if direction == "right":
            if spiral[x,y-1] == 0:
                y = y-1
                direction = "up"
            else:
                x = x+1
            continue
        ...

In the case of [day 3](http://adventofcode.com/2017/day/3) where this snippet is from, I just made a too-big grid so I wouldn't have to bother with re-sizing. Numpy can also fill in a matrix with `0` values, which fit the requirements here well.

However, I had a revelation while working on [day 19](http://adventofcode.com/2017/day/19), which asks you to navigate a piece of fixed-width ASCII art. Rather than attempting to store our grid as a list of lists, why not store it in a dict keyed by `(x, y)` pairs?

    tubes = dict()
    with open("input1.txt") as f:
    for y, row in enumerate(f):
        row_list = list()
        for x, c in enumerate(row):
            tubes[(x, y)] = c
            if c is "|" and y == 0:
                position = (x, y)

This allows you to totally ignore the traditional "bounds" of a list-based data structure, and lets you use the `(x, y)` pairs that you learned in school instead of the `(y, x)` pairs that reading in a file would generally produce. In addition, "moving" in a tuple-keyed dictionary is supremely easy. If you store your directions as tuples as well, you can simply add your tuples together:

    east = (1, 0)
    position = (5, 5)
    new_position = add(position, east)

This data structure ended up being the absolute perfect structure for [day 22](http://adventofcode.com/2017/day/22), which required turning, movement, and an infinite grid:

    def create_if_not_exists(pos):
        if pos not in grid:
            grid[pos] = '.'

    for x in range(10000):
    if grid[position] is '.':
        direction = turn_left(direction)
        grid[position] = '#'
        infections += 1
    elif grid[position] is "#":
        direction = turn_right(direction)
        grid[position] = '.'
    position = add(position, direction)
    create_if_not_exists(position)

One thing that this paradigm will not do? Make a puzzle like [day 21](http://adventofcode.com/2017/day/21) remotely tolerable. If there's any way to chop up and partition a matrix of values without tedious and manual array manipulation, I'm all ears!

Thanks so much for reading- I had a great time working on AoC 2017 and learned a whole lot doing it. All of [my solutions are available](https://github.com/mkolas/advent2017) on GitHub if you'd like to see full examples or get a sense of how I solved a puzzle. I'd highly recommend jumping in next year if you're even a small bit interested- it's a lot of fun and a great series of exercises. 