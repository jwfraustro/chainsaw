# ChainSaw
ChainSaw is a Markov Chain-based password cracking tool for the video game "Grey Hack". Currently capable of cracking any NPC generated password in the game in 20 seconds or less, with privilege escalation for free!

## Features
- **Markov Chain Password Cracking**: Utilizes the same in-game Markov Chain algorithm used in Grey Hack to generate password guesses.
- **RCE Support**: Can be used to self-escalate privileges by cracking and returning a root shell.
- **Customizable**: Users can modify the Markov Chain parameters to suit their needs.

## Installation

1. Greybel: The tool can be eaily imported and built in Grey Hack using the Greybel VSCode extension / plugin. Details on the Greybel extension can be found [here](https://github.com/ayecue/greybel-vs)

2. Manual: You can also manually copy the files into the Grey Hack game, and build the tool using the code editor in game.

## Usage

All parameters and commands ChainSaw uses are available in game by running `chainsaw help` in the terminal.

### chainsaw run - Terminal password cracking

The `run` command can be used to crack passwords within the terminal. The command will not return a shell, but instead returns the password to standard output. By default, `chainsaw` will always attempt to crack the root user, but a different user can be specified using `--user`.

```bash
chainsaw run --user <username>
```

### chainsaw load/run - RCE privilege escalation

ChainSaw's most powerful feature is the ability to self-escalate the privileges of scripts by using the Grey Hack `get_custom_object` function. When deployed from within a script as an RCE, ChainSaw will return a root shell (if successful) to the calling script. The shell will be accessible from within the ChainSaw object as `chainsaw.shell`.

The following recipe is an example of how to use ChainSaw from within a Grey Hack script. This example assumes that ChainSaw is being included as a source file in the calling script, but the same can be done by transferring and calling the compiled binary.

```miniscript

import_code("chainsaw.src") // Import the ChainSaw module

ChainSaw.load // Load chainsaw into the get_custom_object

rce_string = "
chainsaw = get_custom_object.chainsaw
chainsaw.init()
chainsaw.crack()
"

remote_shell.host_computer.touch(path, "chainsaw_rce.src")
rce_source = remote_shell.host_computer.File(path + "/chainsaw_rce.src")
rce_source.set_content(rce_string)
build = remote_shell.build(path + "/chainsaw.src", path)
remote_shell.launch(path + "/chainsaw")

// Wait for chainsaw to execute

root_shell = get_custom_object.chainsaw.shell
```

### Other Commands

- `chainsaw run --ip <ip> --port <port>`: Attempts to crack the password of the remote service running on the specified IP and port. *Very slow*.
- `chainsaw test --password <password>`: Tests a password against ChainSaw's password cracking algorithm. This is useful for testing the algorithm's accuracy, and for testing password strength.

A number of other parameters, such as `--min/max-length`, `--order` and samples customization are available, but are not recommended for most users. The default values are based on the in-game implementation, and will crack any NPC generated password in the game.