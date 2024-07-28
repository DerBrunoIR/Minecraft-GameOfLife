# Minecraft-GameOfLife

This repository contains a implementation of Conway's "Game of Life" inside of Minecraft using command blocks.

https://github.com/user-attachments/assets/3c1f660e-a730-45d5-8e19-34b5d589c328

# How to install
- unzip `World.zip` and put `TheGangOfLife` directory into your minecraft worlds folder `saves`. 


# How it works

## requirements
1. This project aims to build a "Game of Life" simulator by utilizing in game functionalities.
2. Espacially `command blocks` are used for programming.
3. The simulation should be running in real time.
4. And it should be extendiable.

## How the Game of Life works
```
  init all cells somehow
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
As a screen we use `n` rows each of `n` pixels. Each pixel represents a cell.  

For each pixel we have to create an entity such that we can store the number of living neighbours in a scoreboard.
This is the only way to work with numerical values in minecraft as far as I know and we have to count them.

I choosed to use `ItemFrames` since they are easy to place.

We have to be carful not to accidently damage them, since they can die.

![itemframes](https://github.com/user-attachments/assets/1940732c-f6f0-44e9-9486-f0efcdda5b0c)

Further we have to represent the values `2` and `3`. 
I choosed to use bat's a value holder and with some NBT editing we can prevent them from despawning and moving.

![two_three](https://github.com/user-attachments/assets/c02f664d-c180-4aa7-882e-38841f6dc241)

Now we have to update the blocks that represent cells.

For avoid numeric values as much as possible I took the following aproach by utilizing the `execute if` command:
1. `yellow`: init the cells scoreboard to zero. 
2. `blue![two_three](https://github.com/user-attachments/assets/8531ff94-640a-42cd-b60f-5db7e44ae68b)
`: add for each of the 8 neighbours one if read it's pixel and check if it is alive. 
3. `blue`: write back to the pixel if any of the above condition is met.

A redstone pulse onto the red wool would now start the execution of our command chain.

![cell](https://github.com/user-attachments/assets/4972ce82-be88-4a60-ad73-dd13b212173f)

We can now compute if a single cell dies or is revived.

Now by stacking those compute units behind each pixel and implementing a main loop, we can run our simulation.

![build_sytem](https://github.com/user-attachments/assets/7eb0baf7-caca-437e-b1c5-0093e53f5417)

Does it work? Yeah, but it looks weird.

That's because all compute units are reading and writing to the same buffer. 
If one unit reads a cell value from the screen after another unit updated it we get a race condition where an already updated value is read again.

We can fix this by not writing to the read buffer.
Chaching the last frame for reading and writing to a new frame should solve this issue.

![frames](https://github.com/user-attachments/assets/508d8e13-f942-42e6-822d-23950db3f063)

And it did!

This initial condition is called the `Penta-decathlon`.
Will it run infinitly?

https://github.com/user-attachments/assets/3c1f660e-a730-45d5-8e19-34b5d589c328


