---
title: Addition
description: 
published: 1
date: 2021-04-19T14:00:06.204Z
tags: 
editor: markdown
dateCreated: 2021-04-19T09:43:25.811Z
---

# When we dont want to consider carry 

## Instructions used
* ### LDA - Load Accumulator  Direct
This instruction loads the stuff into Accumulator 
* ### MVI - Move Immediate 
This instruction moves the stuff into the memory location specified
* ### ADD 
This instruction adds and moves into the accumulator
* ### STA - Store from Accumulator
This instruction stores stuff from the accumulator into a memory location 

## The actual code 
```assembly
mvi A,16H //Store 16H in A where A = Accumulator
mvi B,10H //Store 10H in B 
ADD B //Load from B and then add it to whatever is in A
sta 0010 //Store whatever is in the accumulator to 0010 
hlt //halt the computer 
```

# Considering that carry exists

## Additional Instructions used 
* ## ADC -  Add register with carry
The content of the register and carry bit are all added to the data in accumulator and then stored in acc.

## The actual code 
```
mvi A,29H
mvi B,99H
add B
mov L ,A
MVI A, 00H
ADC A




