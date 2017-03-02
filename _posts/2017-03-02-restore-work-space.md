---
layout: post
title: "Restore workspace"
date: 2017-03-02
---

Recently, I tried to setup my working environment. The product I worked on mostly using C++, and the application is running on one OS similar as BSD enabled multi-processes programming style. I owned one dedicate Linux machine to code, run function test, etc. but I don't have the root permission. That machine provide nx server, it works quit well, but the nx server is too old that sometimes met annoying issue, only terminate session can solve the problem. Terminate session causes the working environment disappear. Another problem is the machine may reboot by administrator to do maintain work. Here is the current solution, it's quit ok now.

* [tmux](https://tmux.github.io/)

Tmux is used to keep the application without window, and also easy to manage different work area. And use [tmux-resurrect](https://github.com/tmux-plugins/tmux-resurrect) to store the information incase OS is restarted.

* ssh + shell script

To get rid of nx server, use ssh directly. Tmux could not keep the X window's session, here use shell script to restart the X11 application.

```bash
ssh -YXC -c blowfish-cbc,arcfour user@host -t \"tmux attach -t work \\\\; split-window ~/bin/restore.sh\""
```

Below is the `restore.sh`


```bash
#!/bin/sh

PROMPT_START="\[$(hostname)\]"
ORIG_WINDOW_INDEX=`tmux display-message -p '#I'`
ORIG_PANE_INDEX=`tmux display-message -p '#P'`

for window in `tmux list-windows -F '#I'`; do
    tmux select-window -t $window

    for pane in $(tmux list-panes -F '#P'); do
        #tmux send-keys -t ${pane} ""
        outputLines=$(tmux capture-pane -p -t ${pane} | tac)
        prompt_skipped=false
        while read -r line; do
            if [ "$prompt_skipped" = true ]; then
                lastStopApp=$(echo "$line" | perl -lne '/\[\d+\]\s+(Abort|Done|Exit\s+\d+)\s+(.*)$/ && print $2')
                if [ ! -z "$lastStopApp" ]; then
                    #echo "$lastStopApp Runs on pane $pane window $window" >> ~/start.log
                    #TODO do more checks on APP
                    tmux send-keys -t ${pane} "${lastStopApp}&" Enter
                fi
                break
            fi

            if [[ "$line" =~ ^$PROMPT_START.* ]]; then
                prompt_skipped=true
            fi

        done <<< "$outputLines"
    done
done

tmux select-window -t $ORIG_WINDOW_INDEX
tmux select-pane -t $ORIG_PANE_INDEX
```

This script tries to restart the application which executed background and interrupt by the X11 display. Have limitation, this script only handle the last one in pane.

* emacs + rtags

Emacs's desktop can keep the session quit good, rtags manages the C++ AST database.
