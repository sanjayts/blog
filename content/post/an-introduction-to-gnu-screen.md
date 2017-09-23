+++
date = "2017-09-22T12:00:00+06:00"
title = "An Introduction To GNU Screen"
categories = ["programming"]
tags = ["linux", "screen"]
+++
----------------------------------

I personally believe that using a [terminal multiplexer](https://en.wikipedia.org/wiki/Terminal_multiplexer) is a must for anyone who spends a non-trivial amount working on remote *nix servers. This post is a gentle introduction to [GNU screen](https://en.wikipedia.org/wiki/GNU_Screen) but I would like to take a short detour and explain why they are useful.

### Why use a terminal multiplexer?

Some of the most obvious advantages being:

1. Ability to have a saved *session* for your work
2. Cutting down the number of terminal windows you have to open

Let's discuss a usecase wherein you are logged on to your [Edge/Gateway node](https://www.cloudera.com/documentation/enterprise/latest/topics/cdh_sg_gateway_setup.html) and would like to monitor your [Spark](https://spark.apache.org/) jobs, your [HBase](https://en.wikipedia.org/wiki/Apache_HBase) service and at the same time query [Impala](https://en.wikipedia.org/wiki/Apache_Impala) tables. A common solution would be one of:

1. Use a [terminal emulator](https://en.wikipedia.org/wiki/Terminal_emulator) like Putty to log on to the edge node and use Spark/Hbase/Impala by switching directories and managing background processes OR
2. Open three [Putty](https://en.wikipedia.org/wiki/PuTTY) windows and manage one service per window 

This kind-of works but has a few problems:

1. If you now want to log on to multiple servers, it greatly increases the number of windows you have to have open
2. In case you lose connection, you are back to square one since you have lost *all* your sessions you were working on

Enter terminal multiplexers like screen and [tmux](https://en.wikipedia.org/wiki/Tmux): they allow you to manage multiple "virtual" windows within the same terminal emulator application and at the same time provide a mechanism to "save" your session. These sessions stick around till the server is restarted (which happens rarely).

### tmux and screen

`tmux` and `screen` are the most widely used terminal multiplexers. I'm a big fan of tmux but it is almost never installed by default on the "enterprise grade linux installations" (RHEL) and it's a pain to get additional software installed. I feel that screen is "good enough" for what it does and has the added advantage of being installed by default so I'll be discussing screen in this post.

### Basic screen commands

Below is my version of the very basic set of commands required to get started with `screen` and the ones which I most frequently use. The convention followed is `C-a` which means `CTRL` (`Command`) key followed by the character `a`.

* Fire up console and type the command `screen -U -S session_name`. This will create a new screen session which supports `utf8` and has a name `session_name`.
* Use `C-a c` to create a screen window
* Use `C-a k` to kill the screen window
* Use `C-a A` to rename a screen window to something logical. A title "spark monitoring" makes more sense than the default title "bash"!
* Use `C-a d` to detach the screen session in case are about to restart your local box. The good news is that when you lose network connection, screen automatically "detaches" your session; neat!
* Use `C-a "` (that's a double quote) to list all the sessions in an interactive way which can be selected using `vi` keybindings
* Use `C-a num` to jump to the numbered session. For example, let's say you want to jump to jump to the second screen window, you will press `C-a 1`
* Given that screen windows are different from your terimanal emulator windows (with its own buffer), use `C-a ESC` key sequence to drop in the "copy mode" wherein you can move around freely using `vi` key bindings
* To list all the active `screen` sessions, use `screen -ls`. To attach to the first session, use `screen -R`. To attach to a named session (as retrieved from the `screen -ls` command) type `screen -r session_name`.
* If you want to kill a screen session, use `screen -S session_id_or_name -X quit` where `session_id_or_name` is the one which you get from `screen -ls`.

### Recommended Reading

* [How to Use Linux Screen](http://www.rackaid.com/resources/linux-screen-tutorial-and-how-to/)
* [GNU Screen quick reference](http://aperiodic.net/screen/quick_reference)
* [screen - The Terminal Multiplexer](http://www.bangmoney.org/presentations/screen.html)