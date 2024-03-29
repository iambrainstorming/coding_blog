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


### Use yazi for terminal file manager: 
 
[Yazi](https://github.com/sxyazi/yazi)

You can configure it for helix in yazi.toml file

.config/yazi/

[yazi.toml](https://yazi-rs.github.io/docs/configuration/overview)
```toml
[opener]
edit = [
	{ run = '${EDITOR:=hx} "$@"', desc = "$EDITOR", block = true, for = "unix" },
]
```

### Use Zellij terminal workspace

<https://zellij.dev/>

Zellij Tutorial

<https://zellij.dev/tutorials/basic-functionality/>

### Rust based terminal

<https://github.com/alacritty/alacritty>
