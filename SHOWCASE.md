# debugprint.nvim Configuration Showcase

Out of the box, `debugprint` provides a standard set of language configurations to provide 'print-like' logic for over 30 different languages.

However, you may not like the way this is configured for your language of choice. This showcase is designed to collect together variations to the `debugprint` configuration which may be useful.

## Use `console.info()` rather than `console.warn()` for JavaScript/TypeScript

`debugprint` uses `console.warn()` by default for these languages ([explanation here](https://github.com/andrewferrier/debugprint.nvim/issues/72#issuecomment-1902469694)). However, some folks don't like this. You can change it to use `console.info()` instead like this:

```lua
local js_like = {
    left = 'console.info("',
    right = '")',
    mid_var = '", ',
    right_var = ")",
}

return {
    "andrewferrier/debugprint.nvim",
    opts = {
        filetypes = {
            ["javascript"] = js_like,
            ["javascriptreact"] = js_like,
            ["typescript"] = js_like,
            ["typescriptreact"] = js_like,
        },
    },
}
```

## Use `wat-inspector` to fully dump objects in Python

You can use the packages [wat-inspector](https://pypi.org/project/wat-inspector/) to fully dump contents of objects when printing variables in Python. Use configuration that looks like this (you will need to `pip install wat-inspector` and then `import wat` in your script):

```lua
return {
    "andrewferrier/debugprint.nvim",
    opts = {
        filetypes = {
            ["python"] = {
                left_var = "print('",
                mid_var = "'); wat(",
                right_var = ')',
            },
        },
    },
}
```

## Use lazy-loading with lazy.nvim

`debugprint` can be configured, when using [lazy.nvim](https://github.com/folke/lazy.nvim) as a plugin manager, to lazy load itself. Use configuration that looks like this:

```lua
return {
    "andrewferrier/debugprint.nvim",

    -- opts = { … },

    -- The 'keys' and 'cmds' sections of this configuration will need to be changed if you
    -- customize the keys/commands.

    keys = {
        { "g?", mode = 'n' },
        { "g?", mode = 'x' },
    },
    cmd = {
        "ToggleCommentDebugPrints",
        "DeleteDebugPrints",
    },
}
```

## Use a custom `display_counter` counter

The `display_counter` option can be set to a custom callback function to implement custom counter logic. In this case you are responsible for implementing your own counter. For example, this logic will implement essentially the same as the default counter:

```lua
local counter = 0

local counter_func = function()
    counter = counter + 1
    return '[' .. tostring(counter) .. ']'
end

debugprint.setup({display_counter = counter_func})
```
