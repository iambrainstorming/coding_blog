 # Switching your text editor

## [Helix editor](https://helix-editor.com/)

<https://helix-editor.com/>

Modal editor

A post-modern text editor.

### Lightweight and Speed:
Helix is ightweight nature and fast performance. It runs in the terminal, making it efficient and responsive even on remote servers or low-resource environments.

### Keyboard-Centric Interface:
Helix  has a unique modal interface that allows users to navigate and edit text using keyboard commands. This can be very efficient for those who prefer to keep their hands on the keyboard without reaching for the mouse.
You use keyboard shortcuts to select, multi-select, delete, copy, and switch modes, making it fast and efficient to write your document or code once you have become familiar with them.

### Extensive Customization:
Helix is highly customizable, allowing users to tailor it to their specific needs. Users can create custom key mappings, and configure settings to create a personalized editing environment.

To have word wrap:

.helix/config.toml

```toml
[editor.soft-wrap]
enable = true
max-wrap = 25 # increase value to reduce forced mid-word wrapping
max-indent-retain = 0
wrap-indicator = ""  # set wrap-indicator to "" to hide it
```

### Learning Curve and Efficiency:
While Helix has a learning curve initially, some users argue that once mastered, it can be extremely efficient for text editing and manipulation. In command mode, activated by pressing the colon key (:), you can explore the tutorial (:tutor) to gain experience with Helix commands within a few days or a week.


### Use Zellij terminal workspace

<https://zellij.dev/>

Zellij Tutorial

<https://zellij.dev/tutorials/basic-functionality/>

#### Zellij strider

```bash
zellij -l strider
```
[Zellij Filepicker](https://zellij.dev/tutorials/filepicker/)

### Rust based terminal

<https://github.com/alacritty/alacritty>

### Use yazi for terminal file manager: 
 
[Yazi](https://github.com/sxyazi/yazi)

yz-fp

```bash
#!/bin/bash

selected_file="$1"

# KEY ASSUMPTIONS
# - editor pane was the most recently active pane
# - yazi fp is only open floating pane

# go back to editor
zellij action toggle-floating-panes

# open selected file in editor
zellij action write 27 # send escape key
zellij action write-chars ":open $selected_file"
zellij action write 13 # send enter key

# go back to floating yazi pane and close it
zellij action toggle-floating-panes
zellij action close-pane
```


yy

```bash
#!/bin/bash

zellij run -c -f -- yazi "$PWD"	
```

copy yz-fp and yy to /bin folder and make it executable

```bash
sudo cp yz-fp /bin
sudo cp yy /bin
sudo chmod +x yz-fp
sudo chmod +x yy
```


You can configure it for helix in yazi.toml file

.config/yazi/

[yazi.toml](https://yazi-rs.github.io/docs/configuration/overview)
```toml
[opener]
helix = [
  { run = 'yz-fp "$0"', desc = "Use yazi as file picker within helix" },
]

[open]
rules = [
  { name = "**/*", use = "helix" },
]
```

~/.config/helix

config.toml

```toml
[keys.normal]
C-y = ":sh yy" # launch filepicker

[editor.lsp]
display-inlay-hints = true


[keys.normal.space]
"H" = ":toggle lsp.display-inlay-hints"
 
```

It will open file picker on Ctrl+y in zellij

Space + Capital H for toggle inlay hints

