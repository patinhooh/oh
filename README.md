# OH

**OH (Orchestration Helper)** is a lightweight tool for organizing and invoking executables.

It provides a simple way to group related executables into modules and invoke them through a consistent command-line interface.

## ToC

- [Why OH?](#why-oh)
- [Modules](#modules)
  - [Module Example](#module-example)
- [Usage](#usage)
- [Provided modules](#provided-modules)
  - [completion](#completion)
- [How I use it](#how-i-use-it)
- [License](#license)

## Why OH?

I usually prefixed my scripts with my username to avoid name conflicts:

```text
patinhooh-waybar-refresh
patinhooh-sway-screenshot
patinhooh-sway-move
```

As the number of scripts grew, it eventually became a small framework:

```text
patinhooh/
├── bin/             # CLI commands
│   └── patinhooh-*
├── lib/             # Supporting scripts
├── libexec/         # Scripts called by other applications
└── templates/       # Script templates
```

At that point, `patinhooh` no longer felt like a good project name. I also wanted something shorter and decided to turn it into a proper CLI.

Instead of:

```bash
patinhooh-waybar-refresh
```

I wanted something more like:

```bash
oh waybar refresh
```

And it fit better than I expected:

```text
Oh, Waybar, refresh!
```

So **OH** became **Orchestration Helper**.

## Modules

A module is a directory containing executables and, optionally, supporting directories. OH uses the module and command names to determine which executable to run.

By default, OH looks for modules in:

```text
~/.local/lib/oh/
```

The location can be overridden with `OH_MODULES`:

```bash
OH_MODULES="/path/to/modules" oh --list
```

### Module Example

```text
~/.local/lib/oh/example/
├── example
├── cleanup
├── move
└── lib/
    └── example.sh
```

Here, `example` is the module's default command, `cleanup` and `move` are commands, and `lib/example.sh` is supporting code.

Modules can also be used solely as libraries for other modules.

Only executable files at the module's top level can be invoked by OH (`cleanup` and `move`). The `lib` directory is ignored by OH's command dispatch.

See the [Usage](#usage) section for how OH dispatches arguments to modules and their commands.

## Usage

OH handles its own options until a module name is provided.

```bash
oh --list
oh -v
```

After the module name, OH decides whether the first argument is a command or an argument for the module's default command.

If the first argument after the module starts with `-`, it is passed to the module's default command:

```text
oh example --help
   │       └─ argument passed to example/example
   └─ module
```

Otherwise, the first argument is treated as a command:

```text
oh example cleanup --all
   │       │       └─ argument passed to example/cleanup
   │       └─ command
   └─ module
```

So these commands resolve to:

```text
oh example
└── ~/.local/lib/oh/example/example

oh example --help
└── ~/.local/lib/oh/example/example --help

oh example cleanup
└── ~/.local/lib/oh/example/cleanup

oh example cleanup --all
└── ~/.local/lib/oh/example/cleanup --all
```

## Provided modules

OH includes a small set of modules that provide functionality around OH itself.

If you have an idea for a useful module for OH, or an improvement to one of the existing modules, feel free to send it my way. I'll take a look when I can.

### completion

The `completion` module generates shell completion for OH.

It provides completion for OH options, modules, and commands.

|Supported shells|
| :------------- |
| bash           |

After adding the module you can add this to you bash config.

```bash
eval "$(oh completion bash)"
```

## How I use it

My dotfiles are managed with [Git](https://git-scm.com/) and [GNU Stow](https://www.gnu.org/software/stow/).

I keep the configuration for a piece of software and its supporting scripts in the same Stow package.

For example:

```text
sway/                   # Stow package
├── .config/            # Configuration
│   └── sway/
│       └── config
└── .local/lib/oh/      # OH Modules
    ├── sway/           # Module for CLI
    │   ├── lib/
    │   │   └── sway.sh # Support Const/Functions for commands
    │   ├── sway
    │   └── screenshot
    └── sway-hooks/     # Module for Hooks
        └── lid-state
```

My typical workflow is:

1. Keep configuration and supporting scripts together in a Stow package.
1. Use Stow to install or remove the package as a single unit.
1. Use OH to invoke the scripts provided by installed packages.

For example:

```bash
stow sway
oh sway screenshot
```

Removing the package removes both the configuration and its OH module:

```bash
stow -D sway
```

This gives my configuration and supporting scripts the same lifecycle: **they are installed together, removed together, and kept together in the dotfiles repository.**

## License

OH is licensed under the [MIT License](LICENSE).
