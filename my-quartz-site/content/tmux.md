# create new tmux

tmux 

tmux new -s salam


# attach to tmux

tmux attach

tmux a -t salam

# list tmux

tmux ls


# cntr + b

Switch          &nbsp;&nbsp;&nbsp;          w

Detach (exit)      &nbsp;&nbsp;&nbsp;       d


Destroy window      &nbsp;&nbsp;&nbsp;      x


Split horizontally    &nbsp;&nbsp;&nbsp;    %


Split vertically     &nbsp;&nbsp;&nbsp;     "

# other

Switch          &nbsp;&nbsp;&nbsp;          s

Create window       &nbsp;&nbsp;&nbsp;      c



### config

sudo nano ~/.tmux.conf

set -g mouse

set -g set-titles on

#### after change :

exit tmux

tmux kill-server

tmux
