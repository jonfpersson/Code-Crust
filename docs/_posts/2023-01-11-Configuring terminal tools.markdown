---
layout: post
title:  "Configuring Alacritty + Tmux + Oh My Zsh"
date:   2023-01-11 17:00:00 +0100
author: Jonathan Persson
categories: Productivity
---

![My terminal configuration](/assets/terminalConfig.png)

Producing code in a reliable, fast pace and effective environment is more important than you think. When developing code, an enormous amount of time is spent *not* actually coding. One way to mitigate this is by accelerating your usage of the UNIX terminal.

`Alacritty` is a GPU-accelerated terminal emulator written in rust which support a lot of modern features. Such as 24-bit colors, copy/paste, clicking on URLs, and custom key bindings.

`Tmux`, on the other hand, is an open-source terminal manager able to tile multiple terminal sessions in a single window. By tiling terminals and even having multiple tmux sessions you'd be able to swiftly monitor multiple terminals set up for different reasons.

`Oh My Zsh` is a framwork for managing your Z-shell. Oh My Zsh supports plugins and themes for customizing the aesthetic part of your terminal

My alacritty config file, `alacritty.yml`, contains the following line. The most important thing here is to open tmux correctly upon launching alacritty. If done incorrectly, it's going to start up a new session every time you open alacritty, which we don't want. Lets start the Z-shell and attatching the last used tmux session.

{% highlight yaml %}

env:
  TERM: xterm-256color

window:
  padding:
    x: 5
    y: 1

  decorations: full

  title: Alacritty

  dynamic_title: true

scrolling:
  history: 10000
  size: 12

shell:
  program: /bin/zsh
  args:
    - -l
    - -c
    - "tmux attach || tmux"

mouse_bindings:
  - { mouse: Middle, action: PasteSelection }

key_bindings:
  - { key: V, mods: Alt, action: Paste }
  - { key: C, mods: Alt, action: Copy }

{% endhighlight %}

<br>

My tmux configuration is filled with key bindings making the use of it a lot more efficient. By default the prefix key is `Ctrl + b` but my config allows me to simply hold `Left Alt`.

For example, by holding down `Left Alt` and moving the arrow keys, I am able to switch between tmux panels.

{% highlight shell %}
# set scroll history to 8000 lines
set-option -g history-limit 8000

# modern colors
set -g default-terminal "screen-256color"
set-option -ga terminal-overrides ',xterm-256color:Tc'

# unbind the prefix
unbind C-a
set -g prefix M-t
bind M-t send-prefix

# panes
bind -n M-h split-window -h -c "#{pane_current_path}"
bind -n M-j split-window -v -c "#{pane_current_path}"
bind -n M-d kill-pane
bind -n M-n new-session
bind -n M-s choose-session
bind -n M-k copy-mode

bind -n M-Up select-pane -U 
bind -n M-Down select-pane -D
bind -n M-Left select-pane -L 
bind -n M-Right select-pane -R 

bind -n M-H resize-pane -L 2
bind -n M-L resize-pane -R 2
bind -n M-K resize-pane -U 2
bind -n M-J resize-pane -D 2

bind r source-file ~/.tmux.conf

# copy to X clipboard
bind -T copy-mode-vi v send -X begin-selection
bind -T copy-mode-vi y send -X copy-selection-and-cancel
set-window-option -g mode-keys vi

set -s escape-time 0
set -g mode-keys vi
set -g mouse on
set -g status on
set -g focus-events on

# Avoid date/time taking up space
set -g status-right ''
set -g status-right-length 0

# Plugins
set -g @plugin 'tmux-plugins/tpm'

# Init Tmux Plugin Manager
run '~/.tmux/plugins/tpm/tpm'

{% endhighlight %}

Finally, this is how my zshrc looks. I'm using PowerLevel10k as my theme, which I strongly recommend. 
Install it by running following command.

{% highlight shell %}
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/themes/powerlevel10k
{% endhighlight %}
Add `ZSH_THEME="powerlevel10k/powerlevel10k"` to your zshrc.
<br>
<br>

Install plugins by running
{% highlight shell %}
cd $ZSH_CUSTOM/plugins && git clone https://github.com/zsh-users/zsh-autosuggestions.git
&& git clone https://github.com/zsh-users/zsh-syntax-highlighting.git 
&& git clone https://github.com/zsh-users/zsh-completions.git
&& git clone https://github.com/djui/alias-tips.git
{% endhighlight %}


{% highlight shell %}

if [[ -r "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh" ]]; then
  source "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh"
fi

# Path to your oh-my-zsh installation.
export ZSH="$HOME/.oh-my-zsh"

ZSH_THEME="powerlevel10k/powerlevel10k"
plugins=(git zsh-syntax-highlighting alias-tips sudo zsh-completions fzf)

source $ZSH/oh-my-zsh.sh

[ -f ~/.fzf.zsh ] && source ~/.fzf.zsh

# To customize prompt, run `p10k configure` or edit ~/.p10k.zsh.
[[ ! -f ~/.p10k.zsh ]] || source ~/.p10k.zsh


{% endhighlight %}

If you wanna learn more about these amazing tools, here are links to their sites.
<br>
[Alacritty site][alacritty-site]
<br>
[Tmux site][tmux-site]
<br>
[ohmyzsh site][ohmyzsh-site]
<br>

[alacritty-site]: https://alacritty.org/
[tmux-site]:   https://github.com/tmux/tmux
[ohmyzsh-site]: https://ohmyz.sh/
