---
layout: post
title:  "Tamagotchi game development in C++"
date:   2023-02-10 19:30:00 +0100
author: Jonathan Persson
categories: Game Development
---

I've always been fascinated by Tamagotchis as a kid, and although I never had one for myself, I vividly remember when my dad finally got me a pokéwalker. For those who don't know, it's a Tamagotchi with a pokémon inside it. I would strap it along my belt and walk around the house trying to gather steps and eventually get the pokémon to evolve.

Even though I liked pokémon a lot as a kid, I always had a soft spot for Digimon. It, like pokémon, is a Japanese anime about digital monsters going on adventures together. The excitement of seeing them on tv every day is something I'll never forget. That's why I decided to combine these two interesting concepts into a project!


To give myself a challenge, I decided to write the project in C++ as I had prior experience with the C language. Knowing the graphics would be a pain to code myself, I scoured the internet for such a library and ended up with `SFML`.

`SFML` is a portable and easy-to-use media library for C++. The library provides a simple interface to the various components of your PC. In my project, I mainly used it for displaying different kinds of graphics, like sprites and windows.

# Graphics

Now, you can't have a game without some textures, so let's make one!
I started up my favorite open-source illustration program, gimp, and started to create some pixel art.

<img src="{{ site.baseurl }}/assets/2023-02-01/koromon_atlas.png">
In this case, you'll see the picture consisting of two frames, firstly these are meant for creating a simple animation and secondly, having them combined into one image will reduce our IO-operations as they'll always be loaded into memory.

Then I just did the same thing for all of the Digimon evolutions I wanted, here are some examples.
<img src="{{ site.baseurl }}/assets/2023-02-01/0_botamon_texture_atlas">
<img src="{{ site.baseurl }}/assets/2023-02-01/2_agumon_texture_atlas_2">

# Code structure