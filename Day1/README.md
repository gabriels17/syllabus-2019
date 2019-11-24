# Getting to know Bash

Introduction into the bash scripting language and learning how to setup our local dev environment.

## Questions (1-2 lines each)

- [ ] What is Linux?
- [ ] What is bash?
- [ ] What is a package manager?
- [ ] What is git?
- [ ] What is npm?
- [ ] What is NodeJS?

## Objectives

- [ ] Create a git repository on GitHub for the course
- [ ] Set up Linux Ubuntu 18.04 on your machine (options include, mono/dual
      boot, Virtual Machine options (VMWare, VirtualBox, Parallels), Boot from
      USB) **(Not necessary on macOS)**
- [ ] Install an editor of choice (e.g. VS Code, Atom, Sublime, etc.)
- [ ] Install a package manager, for macOS install [Homebrew](https://brew.sh/). (apt-get is included in Ubuntu)

## Assignment

Create a bash script `scripts/setup_local_dev_environment.sh` that checks required programs/dependencies.

- [ ] Make sure all bash commands are commented.
- [ ] The script should prompt the user with:
  - [ ] A welcome message that includes the current user’s username (the
        username should not be hard coded).
  - [ ] Information on what the script does
  - [ ] What type of operating system it is running on
- [ ] Display the date and time when the script starts and ends.
- [ ] The script should install the following tools:
  - [ ] brew
  - [ ] git
  - [ ] NodeJS
- [ ] The script should print versions of all tools.
  - [ ] brew
  - [ ] git
  - [ ] NodeJS (node)
  - [ ] npm
- [ ] The script should generate a log file and output to terminal.

## Adding an SSH key to GitHub

The following information can be found
[here for creating new key](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)
and
[here for adding to github](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/)

Now review the handin information, commit your changes and clone the repository with SSH instead of HTTPS. SSH
is more convenient when automating build systems because it allows for
password-less authentication. It is also more secure e.g. keyloggers.

## How do I know I'm done?

- [ ] I have answered the questions
- [ ] I have created an executable script that completes all requirements
- [ ] I have commented the script (what is the purpose of each line or function)

## Handin

You should store the answers and setup scripts inside your repository:

```text
.
├── scripts
│   └── setup_local_dev_environment.sh
└── assignments
    └── day01
        └── answers.md
```

They must be placed at this location to get full marks.\
YourGitRepository/assignments/day01/answers.md\
YourGitRepository/setup_local_dev_environment.sh

You should make a copy of your setup script and maintain it through out the
course for the final handin.

## Tips and tricks
Bash supports functions. Functions in bash behave like commands, not like functions in regular programming
languages. That means they have stdout, stderr and return codes.

The command "tee" redirects output to a file and stdout.

Google can help you find a solution to almost every problem in bash. You just have to know how to phrase your search.
