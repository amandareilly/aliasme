# aliasme
aliasme is utility for customizing bash aliases by directory.  This makes it easy to set up default system-wide aliases, and the override them at a project level.

## Use Case
I developed aliasme because I do personal projects and work projects on the same machine.  I make heavy use of bash aliases, but I've found that I often want to customize them depending on what I'm working on.

For example, I have a `startup` alias that I run each time I sit down to work on a project.  For my day job, I want this alias to:
- Pull down the current status of my remote test server
- Pull all of the most recent changes from github for each repo
- Push the diff back up to the test server
- run any necessary migrations, and finally
- build the latest changes.

On the other hand, when I'm working on a personal project, my startup commands can vary depending on the project.

My previous solution to this was to create separate namespaced aliases for each use case: `startup_work`, `startup_project_a`, `startup_project_b`, etc.  This gets cumbersome really quickly, because I can't just rely on muscle memory.  When switching projects, I'd often find myself using the wrong startup alias for awhile, and I almost always used my work alias first, before remembering I needed to switch for a personal project.

aliasme allows me to simply override any alias at a directory or even project level, so I can simply use `startup` in any directory and it will do the correct thing for that project.

## How It Works
aliasme walks the directory tree, starting from the current directory, until it reaches the root.  At each level, it checks to see whether a `.bash_aliases` file exists.

It then imports each found file in order from least-specific (typically `~/.bash_aliases`) to most specific (`~/path/to/project/.bash_aliases`).  If it exists, the file in the user's home directory is always included, even if the current directory tree does not include the home directory.

This means that if you define an alias in your home directory, you can redefine it anywhere else, and the updated version will take precedence in the directory where it is defined, as well as any subdirectories.

In order to update aliases seamlessly when changing directories, we add a helper function call to the `$PROMPT_COMMAND` variable that will check to see whether the directory has changed before a prompt is displayed.  If it has, it will remove all of the existing aliases and re-source `~/.bash_profile`.

## Usage
1. Clone this repo in your location of choice.
2. Add a symlink in `/usr/bin` (optional - this allows you to use the snippet provided below without changes):
    ```
    sudo ln -s /path/to/repo/aliasme /usr/bin/aliasme
    ```
3. Make sure your `~/.bash_profile` includes the following line.  I recommend putting it last, so that anything included by `~/.bashrc` will override `~/.bash_profile`.
   ```
   if [ -f ~/.bashrc ]; then . ~/.bashrc; fi
   ```
4. In `~/.bashrc`, remove any lines referencing `~/.bash_aliases`, and add the following at the end of the file:
    ```
    # include aliasme script
    if [ -f /usr/bin/aliasme/.aliasme ]; then
        . /usr/bin/aliasme/.aliasme
        sourceAliasFiles
        # add reSource function to PROMPT_COMMAND
        reSourceOnPromptCommand
    fi
    ```
5. Close terminal and reopen it, or simply `source ~/.bash_profile`
