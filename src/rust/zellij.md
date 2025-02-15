## Zellij Commands

### Delete all sessions

You can delete them either through the session-manager (`Ctrl o` + `w`) or with `zellij delete-all-sessions`. 

### Kill all sessions

zellij kill-all-sessions


### attach [session-name]

short: a

Zellij will attempt to attach to an already running session, with the name [session-name]. If given no [session-name] and there is only one running session, it will attach to that session.

The attach subcommand will also accept the optional options subcommand.

### list-sessions

short: ls

Will list all the names of currently running sessions.

### kill-sessions [target-session]

short: k

Will kill the session with the name of [target-session], if it is currently running.

### kill-all-sessions

short: ka

Will prompt the user to kill all running sessions.
