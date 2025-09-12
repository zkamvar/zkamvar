---
title: Random color scheme picker for neovim
subtitle: Getting random color schems in a way that respects light mode
author: Zhian N. Kamvar
date: '2025-09-11'
slug: neovim-colorscheme
categories: ["example"]
tags: ["neovim", "vim", "lua", "editors", "functions"]
draft: false
---

## My vim journey

When I was in grad school, I learned Git and Vim for the first time. I was
intentionally trying to learn only one of these tools. My main editor was
Text Wrangler, Sublime Text (the shareware version that kept bugging me to
purchase a license), Atom (for a hot minute), RStudio (once it had the ability
to pop out panes), and finally Vim once I needed to start editing scripts on
the SLURM cluster when I was at UNL. By 2018, I made Vim my primary IDE and
never looked back.

Last year, while I was on Sabbatical, I decided to switch over to NeoVim and
subsequently caused [a huge mess in my config files](https://github.com/zkamvar/config-files/tree/main/nvim).
After working that way for a year, on August 1, I decided to yeet the whole
thing and start anew with [LazyVim](https://www.lazyvim.org). I cloned [the
LazyVim starter template](https://github.com/LazyVim/starter) and it has been
so wonderful to have tools like proper LSP support so that I can now jump around
to function definitions without having to use grep. It's also great because I
can use `<Leader>uC` to switch my colorscheme from a list of installed themes.

## Color schemes

So there's one thing about me that may not seem obvious, but I am _heavily_
influenced by my peers. At one of my workplaces, I saw that a colleague was
using a light-color scheme for his editor. He said that it avoids the sudden
change in lighting when he switches between his editor and reading documentation
on a website. I thought it made sense, so I switched over and adopted a light
color scheme as well.

Another thing about me is that I can get bored with the same colorscheme easily.
I started with PaperColor and then moved to Rose Pine and then to Catppuccin.
The good thing about all of these themes is that they all have the ability to
auto-detect what the background color is and switch to the appropriate
dark/light variant.

However, I found myself switching between them often and never being
satisfied and I saw other color schemes that looked good, but either required a
manual switch to the light theme (e.g. `strawberry_dark` vs `strawberry_light`)
or they were only appropriate for one background (e.g. FairyFloss is really
only suited for a dark theme, IMO). This frustrated me because if I wanted to
use these themes, I had to be in the right mode.

## Random choice

![Animated gif of a teminal window on a mac with the color scheme rotating when I switch between dark and light mode](/img/colors.gif)

I wanted a way to randomly choose my colorscheme whenever I started my editor
and I wanted that theme to respect the dark mode setting. Initially, I found
[f-person/auto-dark-mode](https://github.com/f-person/auto-dark-mode.nvim),
which gave me a way to automatically switch between light mode and dark mode,
but that was still unsatisfying because I wanted the randomness.

I was inspired by the [zenbones theme
collection](https://github.com/zenbones-theme/zenbones.nvim) when I stumbled
upon
[randombones](https://github.com/zenbones-theme/zenbones.nvim/blob/a934bc07d2ed4a98b74526c172d7f043736d8935/colors/randombones.lua#L4),
which gives a random zenbones theme. The only downside is that some of the
themes were only light or only dark mode, which made them unfit for my purpose,
so I decided to modify it.

I ended up writing a lua function called [znk_colorscheme()], which picks a
colorscheme randomly from a dictionary and applies it. I have three dictionaries
set up:

1. one with light mode themes
2. one with dark mode themes
3. one with themes that auto-switch

The function takes "light" or "dark" as the argument and then appends the
corresponding table to the auto-switching themes table and then picks a random
theme. This way I can manually curate my themes and know that I will always end
up with a random theme that I like.

With this function, I was able to [set up functions for `auto-dark-mode` to trigger
when dark mode toggles](https://github.com/zkamvar/config-files/blob/9d185e32b4de46ed07156d56f90639e3c15bd930/lazy-nvim/lua/plugins/colorscheme.lua#L57-L71). I guess the function could probably be a plugin
and I could configure it so that it detects whether or not there is a light
variant, but I'm happy with the way it turned out for now.

[znk_colorscheme()]: https://github.com/zkamvar/config-files/blob/9d185e32b4de46ed07156d56f90639e3c15bd930/lazy-nvim/lua/config/functions.lua#L11-L92
