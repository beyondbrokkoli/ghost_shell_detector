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
