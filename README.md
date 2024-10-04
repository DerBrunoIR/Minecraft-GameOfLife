# Minecraft-GameOfLife

After playing with recursion and command blocks for a while I challanged myself for implementing Connways Game of life inside a Minecraft (version 1.21.1) world by limiting myself to command blocks and redstone. 

The simulation should be running in real time and It should be possible to increase the screen resolution with reasonable effort.

# How to install
- unzip `World.zip` and put `TheGameOfLife` directory into your minecraft worlds folder `saves`. 

# How it works 

Glowstone and blackstone are representing cells that are alive and dead.
Those blocks are placed in a plane inside the minecraft world forming the world grid.

We can check if the cell at position (x, y, z) is alive by using the following command:
`execute if block x y z glowstone run say cell is alive`
The `say` command is only execute if the if the block at (x, y, z) is glowstone.

`execute` can also be used for counting the number of living neighbours of a cell at position (x, y, z).
After replacing all `[formula]` with specific values the following commands would be valid: 
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
The only way to store values is by using the scoreboard.
Therefore we created beforehand the scoarbard `count`.
`@a` is a slector for all players.
The first command initializes the count score for all players to `0`.
The following commands are checking all sourrounding cells, if the checked cell is alive the score is increased by one.
In the end the score is equal to the count of living neighbours of the given cell.

Command blocks can execute those commands.
However for larger screens it would be insane to hardcode all those coordinates by hand.
Relative coordinates save the day.
With `~dx ~dy ~dz` we can address blocks relativly from the current player position or command block position.

By placing the neighbour counting commands blocks at specific relative offsets (dx,dy,dz) to the cell we can use the following commands:
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
This looks more complicated, however it allows us to just copy those commands into multiple commands at different locations.
We can therefore use the same command for counting neighbours for different cells.

Next we have to discuess how we can update
How to update cells.
   `/execute if block ~ ~ ~-13 glowstone if score @a count < @e[name=Two,limit=1] count run setblock ~ ~ ~-14 minecraft:blackstone`
   `/execute if block ~ ~ ~-14 glowstone if score @acount > @e[name=Three,limit=1] count run setblock ~ ~ ~-15 minecraft:blackstone`
   `/execute if block ~ ~ ~-15 blackstone if score @a count = @e[name=Three,limit=1] count run setblock ~ ~ ~-16 minecraft:glowstone`
   
How to build a 20x20 screen with minimal effort.
   relative addressing, cloning the prototype multiple times

How to prevent race conditions.
   By using two frames. A read frame and a write frame.



As a screen we use `20` rows each containg `20` pixel blocks. Each pixel represents a cell. 

Let's encode dead cells with `blackstone` blocks and alive cells with `glowstone` blocks.

The only way to count something in Minecraft is by incrementing an entities (non player character) score.
However there is no obvious way to use a score of some entity outside of `if` conditions.

Nevertheless behind each pixel block I summoned a `ItemFrame`. 
This entity can't move, can't despawn and is easy to place.

We have to be carful not to accidently damage those, without them the program will stop working.

![itemframes](https://github.com/user-attachments/assets/1940732c-f6f0-44e9-9486-f0efcdda5b0c)

Further we have to represent the values `2` and `3`. 
I choosed to use two bat's for those values. 
Such `ItemFrames` and `bats` are not confused by the command blocks.
By some NBT editing I made them both stick to the same position and disallowed them to despawn. 

![two_three](https://github.com/user-attachments/assets/c02f664d-c180-4aa7-882e-38841f6dc241)

Now we have to implement a pixel procedure that takes a pixel block and calculates the pixel block of the following frame.

To avoid numeric values as much as possible I took the following aproach by utilizing the `execute if` command:
1. `yellow`: Init the **score** of the nearest `ItemFrame` to zero. 
2. `blue`: read the pixel block of all `8` neighbours and check individually if it is `glowstone`.
   
    If it is `glowstone` then increment the nearest `ItemFrame` **score**.
   
4. `blue`:
    - If this pixel block is `glowstone` and the `score` of the nearest `ItemFrame` is greater then the score of the `bat` with the name `Three`:
  
      Replace the pixel block by `blackstone`.
    - If this pixel block is `glowstone` and the `score` of the nearest `ItemFrame` is less then the score of the `bat` with the name `Two`: 

      Replace the pixel block by `blackstone`.
    - If this pixel block is `blackstone` and the `score` of the nearest `ItemFrame` is equal to the score of the `bat` with the name `Three`:
  
      Replace the pixel block by `glowstone`.
   
If the `red_wool` receives a redstone signal, the command chain will be executed and calculate the next pixel block.

![cell](https://github.com/user-attachments/assets/4972ce82-be88-4a60-ad73-dd13b212173f)

Let's stack those pixel procedures behind all pixele blocks.
This can be automated easily, since I used only relative addressing for reading the procedure's pixel block.

![build_sytem](https://github.com/user-attachments/assets/7eb0baf7-caca-437e-b1c5-0093e53f5417)

Implement the main loop.

Let's press the play button.
And, does it now work? 
Yeah something is happing, but it looks weird. 
Those `glider` structures are crashing on the runway.

Well, I think somewhere I made a mistake...

All relative block addressing seems to work, all hardcoded block addresses look right, in isolation a single pixel procedure fullfilles it's task.
- In what order does Minecraft acctually evaluate commands issued by some command block?
- Do all command blocks really read the pixel blocks exactly in the same moment?
 
Hypothesis: one pixel procedure reads a pixel block after another one updated it. 
Let's cache the previous frame somewhere instead of overwriting it inplace.
Then by reading from the cached frame this race conditions will be impossible.

![frames](https://github.com/user-attachments/assets/508d8e13-f942-42e6-822d-23950db3f063)

Pressing play...
And it works!

This initial condition is called the `Penta-Decathlon`.
Do you think it will run infinitly?

https://github.com/user-attachments/assets/3c1f660e-a730-45d5-8e19-34b5d589c328

# Conclusion

We got a Game of Life simulation inside of Minecraft that's running in real time. 

We can also scale the display by copying pixels and their procedures.
Itemframes and blocks of the inital screen must be placed manuelly though.
