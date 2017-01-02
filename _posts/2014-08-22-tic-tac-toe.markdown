---
author: nhuhu
comments: true
date: 2014-08-22 06:29:07+00:00
layout: post
link: https://artificialcognition.wordpress.com/2014/08/22/tic-tac-toe/
slug: tic-tac-toe
title: 'Project: Making an AI that Plays Tic Tac Toe'
wordpress_id: 25
categories:
- Experiments
---

I've been reading about [Blondie24: Playing at the Edge of AI](http://www.amazon.com/Blondie24-Playing-Kaufmann-Artificial-Intelligence/dp/1558607838), which is about a neural network that is trained to play checkers, by playing against itself and using an genetic algorithm. I haven't finished the book yet, but so far it's pretty good. I think I'd like to do something like that, but I'm gonna start out small. Thus, **Tic Tac Toe**. It's pretty easy, and I spent a little while today just making the framework first.

Here's the code which basically checks for who wins and prints the board, I'll do the rest over the next few days maybe:
{% highlight python %}
grid = [[" "]*3]*3

def check(): #checks to see if someone has won
             # if won, returns player, if not, returns None, if tie, returns "tie"

    for row in grid:  # checks row
        x,y,z = row
        if x == y == z != " ":
            return x

    for b in range(3):  # checks columns
        first = grid[0][b]
        second = grid[1][b]
        third = grid[2][b]
        if first == second == third != " ":
            return first

    if grid[0][0] == grid[1][1] == grid[2][2] != " ":  # checks diagonal
        return grid[0][0]

    if grid[0][2] == grid[1][1] == grid[2][0] != " ": 
        return grid[0][2]
    
    count = 0              # checks for a tie
    for c in grid:
        for d in c:
            if d == " ":
                count += 1

    if count == 0:
        return "tie"
    else:
        return None   # when none of the checks work, return a none

def printgrid(): # prints out the grid
    print " _________________"
    for x in grid:
        print "|     |     |     |"
        for y in x:
            print "|  " + y + " ",
        print "|\n|_____|_____|_____|"
{% endhighlight %}

