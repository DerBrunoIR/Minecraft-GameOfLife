# Minecraft-GameOfLife

This repository contains a implementation of Conway's "Game of Life" inside of Minecraft using command blocks.

https://github.com/user-attachments/assets/3c1f660e-a730-45d5-8e19-34b5d589c328

# How to install
- unzip `World.zip` and put `TheGangOfLife` directory into your minecraft worlds folder `saves`. 


# How it works

## requirements
1. This project aims to build a "Game of Life" simulator by utilizing in game functionalities.
2. `command blocks` and `redstone` are used for programming.
3. The simulation should be running in real time.
4. It should be easy to increase the screen resolution.

## How the Game of Life works
```
  init all cells either dead or alive
  for each cell:
    count = number of living neighborus
    if cell is alive:
      if count = 3:
        cell gets revived
    else if cell is dead:
      if count < 2:
        cell dies
      if count > 3:
        cell dies
```
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

Now we have to implement the cell logic.

To avoid numeric values as much as possible I took the following aproach by utilizing the `execute if` command:
1. `yellow`: Init the **score** of the nearest `ItemFrame` to zero. 
2. `blue`: read the pixel block of all `8` neighbours and check individually if it is `glowstone`.
   
    If it is `glowstone` then increment the nearest `ItemFrame` **score**.
   
4. `blue`:
    - If this pixel is alive and the `score` of the nearest `ItemFrame` is greater then the score of the `bat` with the name `Three`:
  
      Replace the pixel block by `blackstone`.
    - If this pixel is alive and the `score` of the nearest `ItemFrame` is less then the score of the `bat` with the name `Two`: 

      Replace the pixel block by `blackstone`.
    - If this pixel is dead and the `score` of the nearest `ItemFrame` is equal to the score of the `bat` with the name `Three`:
  
      Replace the pixel block by `glowstone`.
   

Now by triggering the first command block via redstone the execution of our command chain we can update the pixel block of a single cell.

![cell](https://github.com/user-attachments/assets/4972ce82-be88-4a60-ad73-dd13b212173f)

Let's stack those pixel procedures behind all pixele blocks.
This can be automated easily, since I used only relative addressing for reading the procedure's pixel block.

![build_sytem](https://github.com/user-attachments/assets/7eb0baf7-caca-437e-b1c5-0093e53f5417)

Implement the main loop.

Let's press the play button.
And, does it now work? 
Yeah something is happing, but it looks weird. 
Those `gliders` are not even gliding.

Well, I think somewhere I made a mistake...

All relative block addressing seems to work, all hardcoded block addresses look right, in isolation a single pixle procedure fullfills it's job.
- In what order does Minecraft acctually evaluate commands issued by some command block?
- Do all command blocks really read the pixel blocks exactly in the same moment?
What if one pixel procedure reads a pixel block after another one updated it. 

Let's cache the previous frame somewhere instead of overwriting it.
Then by reading from the cached frame this race conditions shouldn't appear.

![frames](https://github.com/user-attachments/assets/508d8e13-f942-42e6-822d-23950db3f063)

Pressing play...
And it works!

This initial condition is called the `Penta-Decathlon`.
Do you think it will run infinitly?

https://github.com/user-attachments/assets/3c1f660e-a730-45d5-8e19-34b5d589c328

# Conclusion

We got a Game of Life simulation inside of Minecraft that's running in real time. 
We can also scale the display by copying pixels and their procedures.

It would be intresting to find the largest possible screen size Minecraft can deal with.
But I'm looking towards programming with variables.
