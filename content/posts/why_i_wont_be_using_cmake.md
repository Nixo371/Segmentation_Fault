---
title: Why I won't be using CMake on my CLI chess app
date: 2025-03-16
draft: false
---
# The Mistake
I recently started making a chess app called `chessCLI`. The goal is for it to be a CLI interface to play chess and eventually host it on my server so that you can run it just using `curl`. Even in its earliest version, it was already the largest C++ project I had ever made, and so manually building wasn't an option. I had already done a lot of projects in C because my first introduction to programming was in a programming school called 42 which taught in C, so I was already quite comfortable with Make, but I wanted to see what else was out there.

After some quick googling I found out about CMake, and skimming a few websites and the official CMake one I decided to try it as it sounded like an easier-to-use Make that would also compile to any OS. So I did what any sane person would do and asked ChatGPT to generate a CMake build system for my project and then copy-pasted it into my project. I was surprised that it worked first try (after a lot of compiler errors that I fixed), but decided to move on and forget about it.

The way ChatGPT explained it to me I would have to run `cmake ..` from within a `build/` directory, and then `cmake --build .` for the final binary to be generated there. This got tedious after 2 times so I made a little shell script `compile.sh` and added it to the repository:
```bash
#!/bin/bash
mkdir build
cd build
cmake ..
cmake --build .
```

Until now this has worked fine but I really didn't like having to run a shell script to compile my code, as well as not actually understanding how CMake works behind the hood or if multi-platform compatibility even mattered considering the end goal was a `curl`-able app.

So I did some more extensive research into Make and re-discovered why I love this build system. It's incredibly simple, yet totally customizable. I highly recommend [this video](https://youtu.be/DtGrdB8wQ_8?si=U0rr9qPyuju5aykq) by Gabriel Parmer if you want to learn about Make, as it truly is the resource that helped me the most.

# The Hero I Needed
At first glance, Make doesn't seem all that different to a shell script, essentially just running commands like `g++ main.cpp` or anything else you give it, but the deeper you dive, the more you'll see how powerful this simple tool can really be.

### Rules
Inside your Makefile you can set rules. Rules tell Make how to make a certain file. So for example, a simple Makefile for a simple app could look like:
```make
my_app:
	g++ main.cpp -o my_app
```
This tells Make that to generate the file `my_app`, it just needs to run the commands underneath that rule to create that file.

However, make can be used not just to make files, one very common example of this is something like:
```
clean:
	rm -rf $(OBJS)
```
If you assume that `$(OBJS)` is a variable containing the object files created as an intermediate step of compilation, this line simply deletes those object files. But the true beauty is in the fact that this rule now allows you to run **just** those instructions with `make clean`. No random directory, no `--build`, just that. I've also seen rules such as `fclean` to do a full clean (including the binary).

But it gets even better. After the colon, you can specify some prerequisites that MUST be done before that rule gets executed. So going back to that simple example, we should expand it to look like:
```make
my_app: main.cpp
	g++ main.cpp -o my_app
```

This tells Make that to create the file `my_app`, it must have the file `main.cpp` before it can generate it. So now if we don't have `main.cpp`, Make will throw an error instead of the compiler.

### Variables
We can also set variables in Make, which helps in not having refactors take forever. Here are some common variables I see in a lot of Makefiles, which I have used in my own:
```make
CC = g++
CFLAGS = -Wall -Werror -Wextra
SRCS = some.c \
	list.c \
	of.c \
	files.c
OBJS = $(SRC:.c=.o)
BIN = bin
```

You can probably start to see what these are used for, since now we can replace our simple Makefile with:
```make
$(BIN): $(SRCS)
	$(CC) $(SRCS) -o $(BIN)
```

By default, running `make` will execute the first rule, so a common practice is to have the first rule be `all`:
```make
all: $(BIN)
```

Note that in this case, we don't specify any actual commands to be run in that rule, and that's totally fine. We're just allowing `make` to default into creating the binary.

There's so much more about Make that I might talk about in another post but I'll finish with one of my favorite rules:
```make
%.o: %.c
	$(CC) $(CFLAGS) -c -o $@ $^
```
