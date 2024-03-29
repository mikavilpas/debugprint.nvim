# debugprint.nvim

![Test status](https://github.com/andrewferrier/debugprint.nvim/actions/workflows/tests.yaml/badge.svg)

## Overview

`debugprint` is a NeoVim plugin that simplifies debugging for those who prefer a
low-tech approach. Instead of using a sophisticated debugger like
[nvim-dap](https://github.com/mfussenegger/nvim-dap), some people prefer using a
'print' statement to trace the output during execution. With `debugprint`, you
can insert 'print' statements, with debug information pre-populated, relevant to
the language you're editing. These statements include reference information for
quick output navigation and the ability to output variable values.

`debugprint` supports 29 filetypes/programming languages out-of-the-box,
including Python, JavaScript/TypeScript, Java, C/C++ and more. See [the
comparison table](#feature-comparison-with-other-plugins) for the full list. It
can also be extended to support other languages.

## Features

`debugprint` is inspired by
[vim-debugstring](https://github.com/bergercookie/vim-debugstring), but is
updated and refreshed for the NeoVim generation. It has these features:

*   It includes reference information in each 'print line' such as file names,
    line numbers, a monotonic counter, and snippets of other lines to make it easier
    to cross-reference them in output.

*   It can output the value of variables (or in some cases, expressions).

*   It [dot-repeats](https://jovicailic.org/2018/03/vim-the-dot-command/).

*   It can detect a variable name under the cursor if it's a supported Treesitter-based
    language, or will prompt for the variable name with a sensible default if not.

*   It knows which filetype you are working with when embedded inside another
    filetype, e.g. JavaScript-in-HTML, using Treesitter magic.

*   In addition to normal mode, it provides keymappings for visual and operator-pending
    modes, so you can select variables visually and using motions respectively.

*   It provides a command to delete all debugging lines added to the current buffer.

*   It can optionally move to the inserted line (or not).

*   You can add support for languages it doesn't support out of the box.

*   It's [MIT Licensed](LICENSE.txt).

## Demo

<div align="center">
  <video src="https://github.com/andrewferrier/debugprint.nvim/assets/107015/e1a8b93b-0c8f-4f02-86e8-cbe1d476940c" type="video/mp4"></video>
</div>

## Installation

**Requires NeoVim 0.8+.**

Optional dependency for NeoVim 0.8:
[nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter). If this
is not installed, `debugprint` will not find variable names under the cursor and
will always prompt for a variable name. For NeoVim 0.9+, this dependency is
never needed.

Example for [`lazy.nvim`](https://github.com/folke/lazy.nvim):

```lua
return {
    "andrewferrier/debugprint.nvim",
    opts = { … },
    -- Dependency only needed for NeoVim 0.8
    dependencies = {
        "nvim-treesitter/nvim-treesitter"
    },
    -- Remove the following line to use development versions,
    -- not just the formal releases
    version = "*"
}
```

Example for [`packer.nvim`](https://github.com/wbthomason/packer.nvim):

```lua
packer.startup(function(use)
    …
    use({
        "andrewferrier/debugprint.nvim",
        config = function()
            opts = { … }
            require("debugprint").setup(opts)
        end,
    })
    …
end)
```

The sections below detail the allowed options that can appear in the `opts`
object.

Please subscribe to [this GitHub
issue](https://github.com/andrewferrier/debugprint.nvim/issues/25) to be
notified of any breaking changes to `debugprint`.

## Keymappings and Commands

By default, the plugin will create some keymappings and commands for use 'out of
the box'. There are also some function invocations which are not mapped to any
keymappings or commands by default, but could be. This is all shown in the
following table.

| Mode       | Default Key / Cmd    | Purpose                                     | Above/Below Line |
| ---------- | -------------------- | ------------------------------------------- | ---------------- |
| Normal     | `g?p`                | Plain debug                                 | Below            |
| Normal     | `g?P`                | Plain debug                                 | Above            |
| Normal     | `g?v`                | Variable debug                              | Below            |
| Normal     | `g?V`                | Variable debug                              | Above            |
| Normal     | None                 | Variable debug (always prompt for variable) | Below            |
| Normal     | None                 | Variable debug (always prompt for variable) | Above            |
| Visual     | `g?v`                | Variable debug                              | Below            |
| Visual     | `g?v`                | Variable debug                              | Above            |
| Op-pending | `g?o`                | Variable debug                              | Below            |
| Op-pending | `g?O`                | Variable debug                              | Above            |
| Command    | `:DeleteDebugPrints` | Delete all debug lines added to this buffer |                  |

The keys and commands outlined above can be specifically overridden using the
`keymaps` and `commands` objects inside the `opts` object used above during
configuration of debugprint. For example, if configuring via `lazy.nvim`, it
might look like this:

```lua
return {
    "andrewferrier/debugprint.nvim",
    opts = {
        keymaps = {
            normal = {
                plain_below = "g?p",
                plain_above = "g?P",
                variable_below = "g?v",
                variable_above = "g?V",
                variable_below_alwaysprompt = nil,
                variable_above_alwaysprompt = nil,
                textobj_below = "g?o",
                textobj_above = "g?O",
            },
            visual = {
                variable_below = "g?v",
                variable_above = "g?V",
            },
        },
        commands = {
            delete_debug_prints = "DeleteDebugPrints",
        },
    }
    version = "*"
}
```

You only need to include the keys / commands which you wish to override, others
will default as shown above.

The default keymappings are chosen specifically because ordinarily in NeoVim
they are used to convert sections to ROT-13, which most folks don't use.

## Mapping Deprecation

*Note*: as of commit `ee9d6ff`, the old mechanism of configuring keymaps/commands
which specifically allowed for mapping directly to
`require('debugprint').debugprint(...)` is no longer officially supported or
documented. This is primarily because of confusion which arose over how to do
this mapping. Existing mappings performed this way are likely to continue to
work for some time. You should, however, migrate over to the new method outlined
above. If this doesn't give you the flexibility to map how you wish for some
reason, please open an
[issue](https://github.com/andrewferrier/debugprint.nvim/issues/new).

## Other Options

`debugprint` supports the following options in its global `opts` object:

| Option              | Default      | Purpose                                                                                                                                      |
| ------------------- | ------------ | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `move_to_debugline` | `false`      | When adding a debug line, moves the cursor to that line                                                                                      |
| `display_counter`   | `true`       | Whether to display/include the monotonically increasing counter in each debug message                                                        |
| `display_snippet`   | `true`       | Whether to include a snippet of the line above/below in plain debug lines                                                                    |
| `filetypes`         | See below    | Custom filetypes - see below                                                                                                                 |
| `ignore_treesitter` | `false`      | Never use treesitter to find a variable under the cursor, always prompt for it - overrides the same setting on `debugprint()` if set to true |
| `print_tag`         | `DEBUGPRINT` | The string inserted into each print statement, which can be used to uniquely identify statements inserted by `debugprint`.                   |

## Add Custom Filetypes

*Note: If you work out a configuration for a filetype not supported
out-of-the-box, it would be appreciated if you can open an
[issue](https://github.com/andrewferrier/debugprint.nvim/issues/new) to have it
supported out-of-the-box in `debugprint` so others can benefit. Similarly, if
you spot any issues with, or improvements to, the language configurations
out-of-the-box, please open an issue also.*

If `debugprint` doesn't support your filetype, you can add it as a custom
filetype in one of two ways:

*   In the `opts.filetypes` object in `setup()`.

*   Using the `require('debugprint').add_custom_filetypes()` method (designed for
    use from `ftplugin/` directories, etc.

In either case, the format is the same. For example, if adding via `setup()`:

```lua
local my_fileformat = {
    left = 'print "',
    left_var = 'print "', -- `left_var` is optional, for 'variable' lines only; `left` will be used if it's not present
    right = '"',
    mid_var = "${",
    right_var = '}"',
}

require('debugprint').setup({ filetypes = { ["filetype"] = my_fileformat, ["another_filetype"] = another_of_my_fileformats, ... }})
```

or `add_custom_filetypes()`:

```lua
require('debugprint').add_custom_filetypes({ my_fileformat, ... })
```

Your new file format will be *merged* in with those that already exist. If you
pass in one that already exists, your configuration will override the built-in
configuration.

The keys in the configuration are used like this:

| Debug line type     | Default keys            | How debug line is constructed                                                                                                                           |
| ------------------- | ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Plain debug line    | `g?p`/`g?P`             | `my_fileformat.left .. "auto-gen DEBUG string" .. my_fileformat.right`                                                                                  |
| Variable debug line | `g?v`/`g?V`/`g?o`/`g?O` | `my_fileformat.left_var (or my_fileformat.left) .. "auto-gen DEBUG string, variable=" .. my_file_format.mid_var .. variable .. my_fileformat.right_var` |

If it helps to understand these, you can look at the built-in configurations in
[filetypes.lua](lua/debugprint/filetypes.lua).

## Feature Comparison with Other Plugins

(This table is quite wide, you may need to scroll horizontally)

| Feature                                                             | `debugprint.nvim` | [vim-debugstring](https://github.com/bergercookie/vim-debugstring) | [printer.nvim](https://github.com/rareitems/printer.nvim) | [refactoring.nvim](https://github.com/ThePrimeagen/refactoring.nvim) | [vim-printer](https://github.com/meain/vim-printer) | [vim-printf](https://github.com/mptre/vim-printf) | [logsitter](https://github.com/gaelph/logsitter.nvim) |
| ------------------------------------------------------------------- | ----------------- | ------------------------------------------------------------------ | --------------------------------------------------------- | -------------------------------------------------------------------- | --------------------------------------------------- | ------------------------------------------------- | ----------------------------------------------------- |
| Print plain debug lines                                             | :+1:              | :+1:                                                               | :x:                                                       | :+1:                                                                 | :x:                                                 | :x:                                               | :x:                                                   |
| Print variables using current word/heuristic                        | :+1:              | :+1:                                                               | :x:                                                       | :x:                                                                  | :+1:                                                | :+1:                                              | :x:                                                   |
| Print variables using treesitter                                    | :+1:              | :x:                                                                | :x:                                                       | :+1:                                                                 | :x:                                                 | :x:                                               | :x:                                                   |
| Print variables/expressions using prompts                           | :+1:              | :+1:                                                               | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| Print variables using motions                                       | :+1:              | :x:                                                                | :+1:                                                      | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| Print variables using visual mode                                   | :+1:              | :x:                                                                | :+1:                                                      | :+1:                                                                 | :+1:                                                | :x:                                               | :x:                                                   |
| Print debug lines above/below current line                          | :+1:              | :x:                                                                | (via global config)                                       | :x:                                                                  | :+1:                                                | :x:                                               | :x:                                                   |
| Supports [dot-repeat](https://www.vikasraj.dev/blog/vim-dot-repeat) | :+1:              | :+1:                                                               | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| Can control whether to move to inserted lines                       | :+1:              | :x:                                                                | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| Command to clean up all debug lines                                 | :+1:              | :x:                                                                | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| Can put debugprint text into default register                       | :x:               | :x:                                                                | :+1:                                                      | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| *Built-in support for:*                                             | -                 | -                                                                  | -                                                         | -                                                                    | -                                                   | -                                                 | -                                                     |
| arduino                                                             | :x:               | :+1:                                                               | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| bash/sh                                                             | :+1:              | :+1:                                                               | :+1:                                                      | :x:                                                                  | :+1:                                                | :x:                                               | :x:                                                   |
| C                                                                   | :+1:              | :+1:                                                               | :x:                                                       | :x:                                                                  | :x:                                                 | :+1:                                              | :x:                                                   |
| C#                                                                  | :+1:              | :+1:                                                               | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| C++                                                                 | :+1:              | :+1:                                                               | :+1:                                                      | :+1:                                                                 | :+1:                                                | :+1:                                              | :x:                                                   |
| CMake                                                               | :+1:              | :+1:                                                               | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| dart                                                                | :+1:              | :x:                                                                | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| Docker                                                              | :+1:              | :+1:                                                               | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| DOS/Windows Batch                                                   | :+1:              | :x:                                                                | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| fish                                                                | :+1:              | :+1:                                                               | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| Fortran                                                             | :+1:              | :+1:                                                               | :x:                                                       | :x:                                                                  | :+1:                                                | :x:                                               | :x:                                                   |
| Golang                                                              | :+1:              | :+1:                                                               | :+1:                                                      | :+1:                                                                 | :+1:                                                | :x:                                               | :+1:                                                  |
| Haskell                                                             | :+1:              | :+1:                                                               | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| Java                                                                | :+1:              | :+1:                                                               | :+1:                                                      | :+1:                                                                 | :+1:                                                | :x:                                               | :x:                                                   |
| Javascript/Typescript                                               | :+1:              | :+1:                                                               | :+1:                                                      | :+1:                                                                 | :+1:                                                | :x:                                               | :+1:                                                  |
| Kotlin                                                              | :+1:              | :x:                                                                | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| lean                                                                | :+1:              | :x:                                                                | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| lua                                                                 | :+1:              | :+1:                                                               | :+1:                                                      | :+1:                                                                 | :+1:                                                | :x:                                               | :+1:                                                  |
| GNU Make                                                            | :+1:              | :+1:                                                               | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| Perl                                                                | :+1:              | :x:                                                                | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| PHP                                                                 | :+1:              | :+1:                                                               | :x:                                                       | :+1:                                                                 | :x:                                                 | :x:                                               | :x:                                                   |
| Powershell/ps1                                                      | :+1:              | :x:                                                                | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| Python                                                              | :+1:              | :+1:                                                               | :+1:                                                      | :+1:                                                                 | :+1:                                                | :x:                                               | :x:                                                   |
| R                                                                   | :+1:              | :x:                                                                | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| Ruby                                                                | :+1:              | :+1:                                                               | :x:                                                       | :+1:                                                                 | :x:                                                 | :x:                                               | :x:                                                   |
| Rust                                                                | :+1:              | :+1:                                                               | :+1:                                                      | :x:                                                                  | :+1:                                                | :x:                                               | :x:                                                   |
| Swift                                                               | :+1:              | :x:                                                                | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| VimL (vimscript)                                                    | :+1:              | :+1:                                                               | :+1:                                                      | :x:                                                                  | :+1:                                                | :x:                                               | :x:                                                   |
| zsh                                                                 | :+1:              | :+1:                                                               | :+1:                                                      | :x:                                                                  | :+1:                                                | :x:                                               | :x:                                                   |
| Add custom filetypes (doced/supported)                              | :+1:              | :x:                                                                | :+1:                                                      | :x:                                                                  | :x:                                                 | :+1:                                              | :+1:                                                  |
| Customizable callback formatter                                     | :x:               | :x:                                                                | :+1:                                                      | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| Implemented in                                                      | Lua               | VimL                                                               | Lua                                                       | Lua                                                                  | VimL                                                | VimL                                              | Lua                                                   |
