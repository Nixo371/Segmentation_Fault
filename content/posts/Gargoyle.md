---
title: Why I made an argument parser in C++
date: 2025-03-16
draft: true
---
# The Inspiration
I've been working on a command line chess game, with the idea that eventually you'll be able to run it from your command line without downloading anything (using curl) called chessCLI.
While I was working on it I realized that I wanted to make a lot of the aspects of the game data-driven, which led to me making a very crude argument handler for testing. But I wanted more, I wanted the functionality that commands in the terminal have, but in an easy-to-use library I could use on any other projects.