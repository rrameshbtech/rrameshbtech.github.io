---
layout: post
title:  "Fixing a ticking time bomb or Working in a badly written code base"
date:   2024-11-02 02:20:05 +0530
categories: best practices
tags: experience best practices object calisthenics
published: true
excerpt_separator: <!--more-->
---
Hai there!

## Is your code base a ticking time bomb?
Let's go ahead with asking a series of questions to understand how good our code is.
### Am I reading with out much cognitive load?

### Can I predict the intention of the code with out much hassel?

### Am I confident to make a push a change at the end of the day?

## What makes your code a ticking time bomb?

### Bad names
In early days most of the indian movies have this scene where the hero deactivates the time bomb at the very last moment to explode. Luckily our heros know that red wires is the one to cut and saves the world and some super lucky heros gets time bombs with on/off switches which makes their life easier (How kind these villans are?). But just think what if the red wire or switch is not the one deactivates the bomb but bursts it. Are we, the software programmers kind enough like Indian movie villans?
![Time Bomb with switch to turn-off](/assets/images/time-bomb-with-switch.png "Time Bomb with switch to turn-off" )

### Tangled wires or mixed responsibilities

### Timer with no timer or unpredictability

### The unwanted wires or commented or non functioning code
cut the crap

### No box or Schemaless
Now a days with the boom of No SQL DBs, schemaless became a choice No.1. But why? Most never asks this quesiton. Any choice comes with a trade-off are we aware of it when we are making it schemaless. Do we really nead it?
As Martin fowler says programming is 90% decision making 10% typing. Chosing correct data types make the decision making a lot easier.

### No Manual