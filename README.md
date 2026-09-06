## Reasonable Defaults

If you have a terminal open in `my_project/`, and you rename that folder in another window, Bash won't tell you. It will keep showing `my_project/` in your prompt. If you edit files or `git commit`, you are silently working inside the newly renamed folder. This makes it look like your work vanished when you try to find it later.

This script hooks into Bash's `PROMPT_COMMAND` to check if your logical path matches physical reality every time you press enter. If the ground shifts beneath your feet, it drops a wall of doom so you know exactly where your files actually are.

```text
 ============================================================ 
                 WARNING: GHOST SHELL DETECTED                
 ============================================================ 
 Your logical path no longer matches physical reality.
 Bash thinks you are in: /home/user/GHOST_DIRECTORY
 The directory was actually renamed to: /home/user/SHADOW_WORLD
 (Warning 1 of 3. Muting after limit reached.)

```

## Installation

1. Open your bash config:

```bash
nano ~/.bashrc

```

2. Paste this at the very bottom:

```bash
# Set how many times you want the red warning before it leaves you alone
export GHOST_SHELL_MAX_WARNINGS=3

# Internal state variables
export _GHOST_SHELL_COUNT=0
export _GHOST_LAST_PWD=""

check_ghost_shell() {
    local logical_pwd="$PWD"
    local physical_pwd="$(pwd -P 2>/dev/null)"

    # If the directory was deleted completely, pwd -P fails
    if [[ -z "$physical_pwd" ]]; then
        physical_pwd="[DELETED]"
    fi

    # If physical matches logical, we are in reality. Reset the counter.
    if [[ "$logical_pwd" == "$physical_pwd" ]]; then
        _GHOST_SHELL_COUNT=0
        _GHOST_LAST_PWD="$logical_pwd"
        return
    fi

    # If the user changed to a DIFFERENT ghost shell, reset the counter
    if [[ "$_GHOST_LAST_PWD" != "$logical_pwd" ]]; then
         _GHOST_SHELL_COUNT=0
         _GHOST_LAST_PWD="$logical_pwd"
    fi

    # If we are under the max warning limit, throw the red wall of doom
    if (( _GHOST_SHELL_COUNT < GHOST_SHELL_MAX_WARNINGS )); then
        echo -e "\n\e[1;41;97m ============================================================ \e[0m"
        echo -e "\e[1;41;97m                 WARNING: GHOST SHELL DETECTED                \e[0m"
        echo -e "\e[1;41;97m ============================================================ \e[0m"
        echo -e "\e[1;31m Your logical path no longer matches physical reality.\e[0m"
        echo -e "\e[1;31m Bash thinks you are in: $logical_pwd\e[0m"

        if [[ "$physical_pwd" == "[DELETED]" ]]; then
            echo -e "\e[1;31m The directory you are standing in has been DELETED.\e[0m"
        else
            echo -e "\e[1;31m The directory was actually renamed to: $physical_pwd\e[0m"
        fi

        echo -e "\e[1;31m (Warning $((_GHOST_SHELL_COUNT + 1)) of $GHOST_SHELL_MAX_WARNINGS. Muting after limit reached.)\e[0m\n"

        ((_GHOST_SHELL_COUNT++))
    fi
}

# Attach our patch to the Bash prompt hook safely
if [[ "$PROMPT_COMMAND" != *"check_ghost_shell"* ]]; then
    PROMPT_COMMAND="check_ghost_shell; $PROMPT_COMMAND"
fi
```

3. Reload your config:

```bash
source ~/.bashrc

```

## Configuration

Change `export GHOST_SHELL_MAX_WARNINGS=3` to whatever number you want. It will warn you X times before shutting up so you can continue working in the void if you really want to.
