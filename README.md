# Dotfiles
These are some of the configurations and scripts that I use to make my life in Linux more functional and enjoyable.

[![GitHub Super-Linter](https://github.com/lbcnz/dotfiles/workflows/Lint%20Code%20Base/badge.svg)](https://github.com/marketplace/actions/super-linter)

<!-- TOC GFM -->

* [Structure](#structure)
* [Environment](#environment)
* [FAQ](#faq)
    * [How this works?](#how-this-works)
    * [Why you don't use a bare git repo?](#why-you-dont-use-a-bare-git-repo)
    * [Why you don't use a dotfiles manager?](#why-you-dont-use-a-dotfiles-manager)
    * [How to preserve my 'goodies' when doing things as root?](#how-to-preserve-my-goodies-when-doing-things-as-root)
  * [What shortcuts do you use on the terminal?](#what-shortcuts-do-you-use-on-the-terminal)
    * [Have anything to recommend?](#have-anything-to-recommend)
    * [How to record and share terminal sessions?](#how-to-record-and-share-terminal-sessions)
    * [What is the license?](#what-is-the-license)

<!-- /TOC -->

## Structure
```sh
.
├── assets                      # screenshots
├── dots                        # dotfiles
│   └── WARNING.MD              # warning to not delete hidden files
├── gnome                       # hacks for gnome environment
│   ├── nautilus-scripts        # context menu scripts for nautilus
│   ├── nvim-override.desktop   # hack for opening text files w/ alacritty
│   └── README.md               # gnome hacks readme
├── HISTORY.md                  # changes over time
├── LICENSE                     # mit license
├── link2home                   # script to link dotfiles to $HOME
└── README.md                   # dotfiles readme
```

## Environment
- **shell interpreter**: zsh
- **terminal emulator**: alacritty
- **desktop environment**: gnome-shell
- **text editor**: nvim

![Desktop in 2021](https://github.com/lbcnz/dotfiles/raw/main/assets/21-desktop.png)

## FAQ
#### How this works?
I wrote, **link2home** a script to link the dotfiles in this repo to `$HOME`. I use a declarative approach instead of iterating over everything inside `dots` so to allow fine control of what is being linked. The script by default backup original files. The `-f` option use `trash` or `rm` to remove them instead.

```sh
_link ".bashrc"
_link ".zshrc"
...
```

#### Why you don't use a bare git repo?
I tried it originally but just didn't like the flow. Maybe doing things differently would improve it but I am happy with my current approach.

I followed the instructions by [harfangk](https://harfangk.github.io/2016/09/18/manage-dotfiles-with-a-git-bare-repository.html)
```sh
git init --bare $HOME/.dotfiles`
echo 'alias g...="/usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME"' >> $HOME/.aliases
g... config --local status.showUntrackedFiles no
```

#### Why you don't use a dotfiles manager?
[twpayne/chezmoi](https://github.com/twpayne/chezmoi) is interesting but I don't need those features right now or they are superseded by other tools that I currently use and so it would add unneeded complexity. That may change in the future though.

> "If you do not personalize your configuration or only ever use a single operating system with a single account and none of your dotfiles contain secrets then you don't need chezmoi." — chezmoi README

#### How to preserve my 'goodies' when doing things as root?
Let's think of a common use case, editing files as superuser. There are many ways you can run a text editor as root and they all apparently do the same thing albeit being very different.

As it boils down to preserving environment variables I wrote, to demonstrate the different behaviour of these tools, a simple script that prints the variables `$USER`, `$HOME` and `$VIMINIT`. The later being a long string represented simply by yes/no bellow, meaning that it was passed along or not to the new shell.

| command | USER | HOME       | VIMINIT |
| ---     | ---  | ---        | ---     |
| sudo    | root | /root      | no      |
| sudo -E | root | /home/user | yes     |
| su      | user | /root      | yes     |
| sudo su | root | /root      | no      |

What basically happens is that `sudo` reset the environment variables while `su` just overwrite some of them.

su manpage:
> For backward compatibility, su defaults to not change the current directory and to only set the environment variables HOME and SHELL  (plus
> USER  and LOGNAME if the target user is not root).  It is recommended to always use the --login option (instead of its shortcut -) to avoid
> side effects caused by mixing environments.

sudo manpage:
> When sudo executes a command, the security policy specifies the execution environment for the command.

And that policy is to unset all variables unless `Defaults env_reset` in `/etc/sudoers` is changed to `Defaults !env_reset`. Despite I despising typing `sudo` before every command, it is indeed more robust and safer than `su` for use with desktop environments. At least by default.

**"So what if you want to make vim or whatever else work with 'su'?"**

First you have to link or copy your vim configuration to `/root` and if you use `$VIMINIT` you can't run `sudo su`. If you link the files you are increasing surface for root privilege escalation and copying the files, even if automated, is avoidable boring maintenance.

Also you would not want to have a `vimrc` over one 1K lines causing problems when things fall apart and you have to log in as root as last resort to restore the system.

**"So what if I want to use 'sudo' for that."**

You can use `sudo -E` to preserve environment or edit `sudoers` file for having it by default. But still, you will be increasing surface for root privilege escalation and mixing environments is bad practice and can cause tragic side effects. Sometimes it's not a good idea to run things in mixed environments, things may not work and user files may be overwritten as root.

The only 'sane' alternative is to use `sudoedit` or `su -e` which creates a temporary file for editing and overwrites the actual file with it when exiting. It uses the editor set at `$SUDO_EDITOR`, `$VISUAL` or `$EDITOR`, whichever comes first. Hacks are not desired here.

### What shortcuts do you use on the terminal?

**Zsh customs**

| key    | action                |
| ---    | ---                   |
| C^r    | fzf history search    |
| C^t    | fzf directory search  |
| C^e    | edit line with vim    |
| Home   | go to start of line   |
| End    | go to end of line     |
| PgUp   | move to forward word  |
| PgDown | move to backward word |
| Ins    | delete word           |
| Del    | delete char           |

**Bash defaults**

| key | action                      |
| --- | ---                         |
| C^a | move to start of line       |
| C^e | move to end of line         |
| M^b | move backwards word-by-word |
| M^f | move forwards word-by-word  |
| C^r | do a reverse search         |
| C^w | delete word left of cursor  |
| C^l | clear history               |

*Note: C is CTRL and M is META/ALT*

#### Have anything to recommend?
- [arbie](https://github.com/lbcnz/arbie): automatic and robust backup
- [journal](https://github.com/lbcnz/journal): CLI management of notes and tasks
- [fzfx](https://github.com/lbcnz/fzfx): battle-tested use cases for fzf

*Note: those are side projects of mine*

#### How to record and share terminal sessions?
Use [asciicinema](https://asciinema.org/) or termtosvg.

#### What is the license?
My own code is under MIT license but components may be under other licenses.

