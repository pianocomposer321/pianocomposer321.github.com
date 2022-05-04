---
layout: post
title: "Why I'm Rewriting yabs.nvim"
date: 2022-05-03 21:01
---

# Intro

Hello everyone!

I'm the author of [yabs.nvim][yabs], a code-runner/build-system plugin for
neovim written in lua. Recently, I decided to do a full rewrite of the plugin.
This post is to explain why.

If you'd like to check out the changes now, you can look at the
[`rewrite`][yabs-rewrite] branch.

So here are the reasons, in short:

### It tried to be too many things

For example, the use of .yabs files for project-local configurations. Simply
marking files with a certain name as lua files and sourcing them with no prompt
not only is insecure, but is also a problem which should not be up to this kind
of plugin to solve. It should instead by handled by `:set exrc`, or a
[dedicated plugin][nvim-config-local].

### Its concept of an `output` was confusing

This confusion happened mainly because of one thing: the terminal.

For those of you who didn't already know, the point of yabs (in short) was to
make it easy to run commands in different ways (with different "outputs"),
based on filetype or project. One of the "outputs" included by default was the
terminal, and, in fact, this was the main way that I used it personally.

The problem is, the terminal is really a special case when it comes to how to
run commands. With most ways to do it (particularly in vim/neovim), it just
runs the command and either stores the output or makes it immediately available
programatically, e.g. by returning it. Examples of this would by the builtin
`system` function and `vim.loop`/`plenary`.

But the terminal is different - instead of just being a way to run a command
and get output, it shows the output as it's being run, and doesn't provide an
easy way to get the output separately.

So that's where the confusion lay - really, I should have had two separate
concepts (`runner` and `output`) where `runner` would be responsible for
running the command and sending it to `output`, and `output` would be
responsible for displaying it somehow - but because I was focused on the
terminal as the main way I would be using it, I had both of these concepts
together.

This was an issue because:

- #### It was just confusing.

  It's not really expected for an option called `output` to control running the
command as well as displaying it.

- #### It caused a lot of code duplication.

  For example, the quickfix output ran the command using plenary, then displayed
it in the quickfix list. If someone wanted to run it with something else (such
as the builtin `vim.loop`), they would have to compeltely reimplement the part
which displayed it in the quickfix list as well, because these two things
(running and displaying output) were both handled by the same thing, and so
could not be separated out.

The new version of yabs solves this by separating the responsibilities of
running the code and displaying the output to two separate objects - `runner`
and `output`.

### Its grouping of tasks was suboptimal

The old version of yabs only grouped tasks two wasy - by filetype, or globally
(i.e. not at all). Filetype tasks would only be active if they matched the
filetype of the currently focused buffer, and global tasks would be available
all the time. This was confusing because it meant that global tasks would show
up in the list even for projects that weren't applicable at all. This increased
dependence on the .yabs files, but that caused a lot of code duplication for
similar projects, and as mentioned before, wasn't even something that yabs
should have been handling in the first place.

I tried to solve this problem later by adding an `enabled` option, but this
really was more of a bandaid than a solution.

The new version of yabs includes essentially the same thing as the `enabled`
option (instead calling it `active` this time), which is to enable or disable
(or "activate" and "deactivate", if you prefer) tasks based on a given
condition. This is much more versitile, and it outsources the job of deciding
when tasks should be available to you as the user, rather than arbitrarily
deciding that by filetype is the only right way to do so.

### It was just getting to be a mess in general

This includes all of the previous issues and also a bunch of other little ones that crept up while developing it. Basically what happened is that there were a lot of features that got added in after the fact that I never planned to be a part of yabs in the beginning, which means that the codebase was not designed with them in mind. If I'd been better about certain decisions along the way, this wouldn't have been a huge issue, but this was essentially my first big project, so yeah, there were some not-so-great design choices that I made. Fixing them would be a breaking change anyway, so I decided it would be better to do a full rewrite instead.

# Conclusion

I hope that helps explain it!

If anyone is wanting to beta test it, that would certainly be welcome! At the time of writing this I have yet to write any of the documentation for the new branch, but I tried to make the code easy to read, so you may be able to figure it out on your own. If you want to test it, you can always open an issue and ask a question about it too, and I'll be sure to answer as soon as I can.

[yabs]: https://github.com/pianocomposer321/yabs.nvim
[yabs-rewrite]: https://github.com/pianocomposer321/yabs.nvim/tree/rewrite
[nvim-config-local]: https://github.com/klen/nvim-config-local
