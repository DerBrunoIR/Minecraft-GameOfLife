# Minecraft-GameOfLife

This repository contains a implementation of Conway's "Game of Life" inside of Minecraft.

https://github.com/user-attachments/assets/3c1f660e-a730-45d5-8e19-34b5d589c328

# How to install
- unzip `TheGameOfLife.zip` and put `TheGangOfLife` directory into your minecraft saves folder `saves`. 


# How it works

## constraints
1. this project aims to build a "Game of Life" simulator by utilizing in game functionalities.
2. espacially `command blocks` are used for programming.
3. it should ran in real time
4. it should be easy to extend the screen size 

As far as I know in Minecraft it's not easily possible to store values inside of variables without excessive use of the scoreboard.


## 3 phases
![cell](https://github.com/user-attachments/assets/27aaa251-ab3c-41d7-819c-7e63230e5753)
The value of each cell is computed in 3 steps.
1. Initialize a scoreboard value for storing the number of living neighbouring cells.
2. For each cell 
