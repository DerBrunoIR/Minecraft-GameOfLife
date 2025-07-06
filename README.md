<https://github.com/user-attachments/assets/3c1f660e-a730-45d5-8e19-34b5d589c328>

# Minecraft-GameOfLife

After experimenting with recursion using command blocks in Minecraft, I challenged myself to implement Conway's Game of life.

In the end, I was able to create a smoothly running simulation on a 20x20 grid.
A main feature is that the screen can be expanded only with a small constant amount of effort per pixel.

At the top is a video showing the `Pentadecathlon` configuration.

# How to install

- unzip `TheGameOfLife.zip` and put `TheGameOfLife` directory into your Minecraft Worlds folder `saves`.
- Join the World with Minecraft version 1.21.1

# How it works

Glowstone and blackstone are representing cells that are alive and dead.
Those blocks are placed in a plane inside the Minecraft world, forming the world grid.



We can check if the cell at position (x, y, z) is alive by using the following command:

`execute if block x y z glowstone run say cell is alive`.

The `say` command is only executed if the block at (x, y, z) is glowstone.

For counting dead and alive neighbors, we need some kind of variable. 
The easiest way to store numbers are the scoreboards of the game. 
`execute` can also be used for storing the number of living neighbors of a cell at position (x, y, z) into all player scores.
After replacing all `[formula]` with their specific value, the following commands would be valid:

```
scoreboard players set @a count 0
execute if block [x-1] [y]   [z] minecraft:glowstone run scoreboard players add @a count 1
execute if block [x-1] [y+1] [z] minecraft:glowstone run scoreboard players add @a count 1
execute if block [x]   [y+1] [z] minecraft:glowstone run scoreboard players add @a count 1
execute if block [x+1] [y+1] [z] minecraft:glowstone run scoreboard players add @a count 1
execute if block [x-1] [y]   [z] minecraft:glowstone run scoreboard players add @a count 1
execute if block [x-1] [y-1] [z] minecraft:glowstone run scoreboard players add @a count 1
execute if block [x]   [y-1] [z] minecraft:glowstone run scoreboard players add @a count 1
execute if block [x+1] [y-1] [z] minecraft:glowstone run scoreboard players add @a count 1
```

`@a` is a selector for all players, and `count` is a previously created scoreboard.
The first command initializes the count score for all players to `0`.
The following commands are checking all surrounding cells; if the checked cell is alive, the score is increased by one.

In the end, the score is equal to the count of living neighbors of the given cell.

Command blocks can execute those commands.
However, for larger screens, it would be insane to hardcode all those coordinates by hand.
Relative coordinates save the day.

With `~dx ~dy ~dz` we can address blocks relatively from the current player position or command block position.

By placing the neighbor counting commands blocks at specific relative offsets (dx,dy,dz) to the cell, we can use the following commands:

```
scoreboard players set @a count
execute if block ~[dx+1] ~[dy]   ~[dz] minecraft:glowstone run scoreboard players add @a count 1
execute if block ~[dx-1] ~[dy+1] ~[dz] minecraft:glowstone run scoreboard players add @a count 1
execute if block ~[dx]   ~[dy+1] ~[dz] minecraft:glowstone run scoreboard players add @a count 1
execute if block ~[dx+1] ~[dy+1] ~[dz] minecraft:glowstone run scoreboard players add @a count 1
execute if block ~[dx-1] ~[dy]   ~[dz] minecraft:glowstone run scoreboard players add @a count 1
execute if block ~[dx-1] ~[dy-1] ~[dz] minecraft:glowstone run scoreboard players add @a count 1
execute if block ~[dx]   ~[dy-1] ~[dz] minecraft:glowstone run scoreboard players add @a count 1
execute if block ~[dx+1] ~[dy-1] ~[zd] minecraft:glowstone run scoreboard players add @a count 1
```

This looks more complicated, however, it allows us to just copy those commands into multiple commands at different locations.
We can therefore use the same commands for counting neighbors for different cells.

Next we discuss how to change the state of cells.

The classic game of life obeys to the following rules:

1. Any live cell with fewer than two live neighbors dies, as if by underpopulation.
2. Any live cell with two or three live neighbors lives on to the next generation.
3. Any live cell with more than three live neighbors dies, as if by overpopulation.
4. Any dead cell with exactly three live neighbors becomes a live cell, as if by reproduction

By simplifying the rules, we get:

1. Any live cell with more than three neighbors dies.
2. Any live cell with less than two neighbors dies.
3. Any dead cell with exactly three neighbors rises from the dead.

Conveniently, we can leverage the `execute` command again.
This time, however, we have to compare to values.
Specifically, the values `2` and `3`.
Therefore, I created two entities holding scoreboard values `2` and `3`.
`@e[name=Two,limit=1]` can addresses a bat holding the value `2` in its score.

![two_three](https://github.com/user-attachments/assets/c02f664d-c180-4aa7-882e-38841f6dc241)

Now for a cell placed at offset (dx, dy, dz) we can implement the following rules:

```
execute if block ~[dx] ~[dy] ~[dz] glowstone if score @a count > @e[name=Three,limit=1] count run setblock ~[dx] ~[dy] ~[dz] minecraft:blackstone
execute if block ~[dx] ~[dy] ~[dz] glowstone if score @a count < @e[name=Two,limit=1] count run setblock ~[dx] ~[dy] ~[dz] minecraft:blackstone
execute if block ~[dx] ~[dy] ~[dz] blackstone if score @a count = @e[name=Three,limit=1] count run setblock ~[dx] ~[dy] ~[dz] minecraft:glowstone
```

Those conditions are mutual exclusive.
This allows us to run those commands in sequence without any branching.

1. init scoreboard
2. count live neighbors
3. update cell

![command blocks](https://github.com/user-attachments/assets/4972ce82-be88-4a60-ad73-dd13b212173f)

Now we can simulate a single cell.
If we want to simulate multiple cells at once, we have to store the neighbor count individually at different places.
Until now, the counts from different cells are all stored in the scores of all players.
The easiest way is to introduce for each cell a new entity, like a `ItemFrame`.
This entity can't move by default, can be placed like a block and doesn't despawn.
The selector `@e[type=minecraft:item_frame,limit=1,sort=nearest]` selects the nearest `ItemFrame` from the position of execution.
Relative to the command block position, we can now also address different values inside the scoreboard.

![Item frames](https://github.com/user-attachments/assets/1940732c-f6f0-44e9-9486-f0efcdda5b0c)

Now extending the grid by one cell is easy.

1. clone the command blocks
2. init the cell state
3. place the `ItemFrame`

For the 20x20 screen, automation seemed worthwhile.

![build_sytem](https://github.com/user-attachments/assets/7eb0baf7-caca-437e-b1c5-0093e53f5417)

One issue is still not covered.
Due to the parallel nature of cell computation, reading and writing from the same grid leads to race conditions.
A cell counting it neighbors would also count already updated neighbors.
Using two grids can solve this concurrency issue.
Therefore, I copied the latest grid state to another location before cells can safely read it.

![Frames](https://github.com/user-attachments/assets/508d8e13-f942-42e6-822d-23950db3f063)

# Conclusion

The final simulation runs fluently on the 20x20 screen.
In the future, it would be interesting to explore larger screen sizes.

By sharing the world, you can try it by your self.

Thanks for reading!
