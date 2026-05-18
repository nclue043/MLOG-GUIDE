# MLog Guide

## Table of contents
1. [Table of contents (you're here!)](#table-of-contents)
2. [Introduction](#introduction)
3. [MLog](#mlog)
    - [Execution](#execution)
    - [Variables](#variables)
    - [Conditionals](#conditionals)
    - [Linking buildings](#linking-buildings)
        - [Sensor](#sensor)
        - [Control](#control)
    - [Binding units](#binding-units)
        - [Flagging units](#flagging-units)
    - [Solutions](#solutions)
4. [Other](#other)

##  Introduction
If you're reading this guide, you probably already know what Mindustry logic is, but just in case you don't, Mindustry logic (often just mlog for brevity) is the language which the various processor buildings in Mindustry use. If you've never programmed before, don't worry. This guide assumes no prior programming experience.

Throughout the guide I'll be showing mlog code snippets. These aren't valid (you can't load them into the game) for readability. I decided on this as the actual text representation is cluttered with parameters that the command doesn't even use in that configuration (e.g. `@copper` in `Unit locate` despite being in `building` and not `ore` mode). Instead, the code snippets will be written as they appear in-game (as of build 157.4). Values will be represented with `()`, dropdowns with `[]`, and variable names with `{}`.

As you read through, I encourage you to follow along if you can. Practice makes perfect, as cliche as the saying is. Just make sure to actually experiment with and understand the concepts discussed, you aren't going to learn anything by blindly copying things. I won't be covering every instruction, so you should go look through them yourself after this guide.

## MLog

### Execution
MLog is typically run from top to bottom (see [Conditionals](#conditionals)) at some amount of instructions per second and loops back to the top when it reaches the bottom, continuing to loop indefinitely. You can also prematurely loop back to the top using `End`. Though seldom used, there's also `Stop` which permanently halts execution of the processor.

### Variables
Variables are one of the most important things in programming, and it's where we'll begin. A variable is just something that holds a value, say a number or a piece of text. Variables in mlog are quite simple. You can set them with `Set: {A} = (B)`, where A is the variable's name and B is the value it will be set to, and other instructions that have an output. You can do simple math with them using `Operation: {result} = (a) [+] (b)` and you can use them in place of any normal value.

```
0 Set: {a} = (5)
1 Set: {b} = (3)
2 Operation: {result_add} = (a) [+] (b)
3 Operation: {result_sub} = (a) [-] (b)
```

Variables can hold one of many different data types. Here they are for reference, though you aren't expected to remember all of them:
| Type     | Description                               | Example           |
| -------- | ----------------------------------------- | ----------------- |
| number   | a simple number                           | `12.34`           |
| string   | text that's "wrapped in double quotes"    | `"Hello, world!"` |
| bool     | true or false                             | `true`            |
| null     | nothing. returned by functions on failure | `null`            |
| building | an object reference to a building         | `shard1`          |
| unit     | an object reference to a unit             | `@unit`           |
| content  | represents a type of something            | `@copper`         |

MLog also provides many built-in variables, which start with the symbol @, like the aforementioned `@unit`, and they can be found in the built-in variables tab in a processor. Most are read-only, and can represent many things such as: an aforementioned Content type, like `@copper`, a mathematical constant like `@pi`, properties of the processor itself, like `@thisx` and more. 

### Conditionals
Of course, simple math isn't all you can do. Think of any task you do in real life, and inevitably there will be a choice you make, or a thing you need to repeat, not a linear path. MLog lets you do this with essentially its sole control flow operation, `Jump`. It has many modes, but defaults to `Jump: (result) [not] (false)`, where not means "not equal to". If the whole condition evaluates to true, it jumps to the specified part of code (here I represent it as `Jump -> address:`), and if it's false then it just continues on normally. This is the only control flow you'll get, no if statements, no loops. Jumps are very common, so the built-in editor automatically shifts line numbers when new lines are inserted.

```
0 Set: {number} = (15)
1 Jump -> 5: (number) [>=] (10) ──────┐
2 Print: ("number is less than 10!")  │
3 Print Flush: to (message1)          │
4 Stop                                │
5 Print: ("number is 10 or above!") <─┘
6 Print Flush: to (message1)
7 Stop
```

### Linking buildings
Before, we were basically only moving and manipulating numbers. Now it's time to enter the actual Mindustry world. You can link buildings to a processor by selecting a processor, then pressing another building. Above that building you can see its automatically assigned name, which is accessible in the processor as a read-only variable.

### Sensor
It's no use just having that variable though, first you must use one of the instructions that takes a building as input. One of the most important of these is `Sensor`, which lets you sense properties of buildings (or units! see [Binding units](#binding-units)). There's a comprehensive dropdown list of all the built-in properties you can sense, though in v8 you can also sense another processor's variables directly.

```
0 Sensor: {x} = (@x) in (container1)
1 Sensor: {y} = (@y) in (container1)
2 Sensor: {items} = (@copper) in (container1)
```

### Control
Finally, we will get to start actually affecting the world. `Control` lets you, well, control a building. The most important things you can do with it are enabling/disabling buildings (`Control: set [enabled] of (building) to (0)`) and making buildings, specifically turrets, target a position or unit. By now, you've learned enough to make useful scripts! Try writing a script that disables a thorium reactor when it doesn't have cryofluid, a common and practical piece of logic many have saved in their schematic lists. The solution will be [here](#solutions).

### Binding Units
While controlling buildings is a good start, most logic uses units as well. Binding units often confuses beginners, but it's pretty simple. Calling `Unit Bind: type (utype)` binds the next unit of the given type, in order of unit creation (oldest to youngest), and stores it in `@unit`.

This means it can still be ordered by players, as it hasn't been controlled yet. The unit is only controlled when you call, well, `Unit Control`. Units continue following their last given order (even after binding a new unit), but eventually stop unless given more. If you call `Unit Control: [unbind]`, then that will cancel orders. To bind another unit without cancelling, simply call `Unit Bind` again.

Try creating a script that makes every flare move 5 tiles the processor's left (`@thisx` and `@thisy`). [Solutions](#solutions).

### Flagging units
Sometimes you want to bind every unit of a given type, but often you just want to bind a few, if not only one. MLog provides a simple system for "claiming" units and stopping other processors from using them. Units can store a single number, called its flag, defaulting to 0. When binding a unit, you check its flag first and bind again if the flag isn't 0. After finding a free unit, you can then flag it with `Unit Control: [flag] value (thisf)` where `thisf` is a unique number. Common practice is to tie it to the processor's position, as such:

```
0 Operation: {temp} = (@thisy) [*] (@mapw)
1 Operation: {thisf} = (@thisx) [+] (temp)
```

Now try safely binding a free flare, flag it, then move it to the processor's position directly until it dies. [Solutions](#solutions).

### Solutions
There can be many different ways of writing the same code, so don't be discouraged if your solution is different from those provided here.

#### Reactor safety:
```
0 Sensor: {cryo} = (@cryofluid) in (reactor1)
1 Jump -> 4: (cryo) [==] (0) ──────────────────┐
2 Control: set [enabled] of (reactor1) to (1)  │
3 End                                          │
4 Control: set [enabled] of (reactor1) to (0) <┘
```
Take note of that `End` there. Remember, top to bottom execution. After a jump, that's still true!

#### Flare control:
```
0 Unit Bind: type (@flare)
1 Operation: {leftx} = (@thisx) [-] (5)
2 Unit Control: [move] x (leftx) y (@thisy)
```
Remember, units are bound in a cycle, and execution loops after reaching the end!

#### Flagging:
```
0 Operation: {temp} = (@thisy) [*] (@mapw)
1 Operation: {thisf} = (@thisx) [+] (temp)
2 Wait: (0.05) seconds <──────────────────────┐
3 Unit Bind: type (@flare)                    │
4 Sensor: {unitf} = (@flag) in (@unit)        │
5 Jump -> 2: (unitf) [not] (0) ───────────────┘
6 Unit Control: [flag] value (thisf)
7 Unit Control: [move] x (@thisx) y (@thisy) <┐
8 Sensor: {dead} = (@dead) in (@unit)         │
9 Jump -> 7: (dead) [not] (true) ─────────────┘
```
Be careful here, most of the time it's irrelevant, but code like this often has race conditions, specifically TOC/TOU. Basically, if lines 3-5 run, the processor finishes for this tick, and another processor takes that same unit, then the next tick, our processor will just continue with line 6 anyways. This is what that wait is for; The details aren't important but it accumulates enough instruction burst to guarantee that lines 3-6 will run in the same tick. The equation for this is `i/ips` where `i` is the number of instructions and `ips` is the instructions per second of your processor. This burst is maxed out at `5*ipt` insructions, where ipt is `ips/60`.

## Other
And... that's it! This is just the start, there's so much more you can do. Go out there and use your hopefully newfound knowledge! Reading through a guide can only get you so far after all.

Like I said, this guide doesn't cover every instruction, but the in-game tooltips should be enough to let you make most common logic scripts, such as unit item hauling.

If you want a more normal programming experience and don't want to be writing basically assembly, there are multiple programming languages that compile to mlog, people have made C compilers, python compilers, and [mindcode](https://github.com/cardillan/mindcode) which was made with targeting mlog in mind.
