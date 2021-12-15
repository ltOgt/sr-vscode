# SmallRead Grammar for Syntax Highlighting

Add syntax highlighting for https://github.com/ltOgt/SR-dart to vscode.

Since this extension is not published, you have to clone it into `~/.vscode/extensions/`.

## Explenation
```json
"scope": {
    "patterns": [
        {
            "name": "scope.sr",
            "match": "(?<=^)(\\.)+"
            // any number of dots beginning at line start
        }
    ]
},
"name": {
    "patterns": [
        {
            "name": "name.sr",
            "match": "(?<=(^\\.*))([^:.]+[^:]*)(?=:)"
            // Names may not start with dot, and may not contain any colons
            // They may only be preceeded with scoping dots
        }
    ]
},
"value": {
    "patterns": [
        {
            "name": "value.sr",
            "match": "(?<=[^:]:)([^:]{1}[^(**)]*)"
            // Values start after a single colon (after scope or name)
            // They may start with dot, but not with colon
            // They may contain colons after the first character
            // Values end at line end (or at comment marker)
        }
    ]
},
"object-head": {
    "patterns": [
        {
            "name": "object-head.sr",
            "match": "(?<=[^:]::)([0-9]+)"
            // Object heads start with two colons and specify their scope size
        }
    ]
},
"list-head": {
    "patterns": [
        {
            "name": "list-head.sr",
            "match": "(?<=[^:]:::)([0-9]+)"
            // Object heads start with three colons and specify their scope size
        }
    ]
},
"comment": {
    "name": "comment.sr",
    "begin": "(\\*\\*)(?=\\s)",
    "end": "$",
    "patterns": [
        {
            "name": "constant.character.escape.sr",
            "match": "\\\\."
        }
    ]
}
```

## Custom coloring

Add the following to your `settings.json` (e.g. at `~/.config/Code/User/`):

```json
{
    //...
    "editor.tokenColorCustomizations": {
        "textMateRules": [
            {   
                "scope": "scope.sr",
                "settings": {
                    "foreground": "#8787AF"
                }   
            },
            {   
                "scope": "name.sr",
                "settings": {
                    "foreground": "#b65c63"
                }   
            },
            {   
                "scope": "value.sr",
                "settings": {
                    "foreground": "#90b773"
                }   
            },
            {   
                "scope": "object-head",
                "settings": {
                    "foreground": "#1a8cdd"
                }   
            },
            {   
                "scope": "list-head",
                "settings": {
                    "foreground": "#1a8cdd"
                }   
            },
            {   
                "scope": "comment.sr",
                "settings": {
                    "foreground": "#8787AF"
                }   
            }
        ]   
    }   
}
```

## Small note on the regex
For you and my future self to better understand the regular expressions, here is a small explanation of `base.gidsym`:

`(?<=^|[\\s|\\|])(>|\\.|\\(|\\)|<|\\^|°|\\?|:|:=|\\^=|~|/|§|¬|!=|i|0|=|x|->|=>|<-|[0-9]+\\)|--|\\+|-|//|\\$|>>|::|\\|-|E|e|00|\\*|#|\\{|\\}|d|v|@|\\*\\*|_|\\[ \\]|\\[x\\]|\\[\\*\\]|\\[\\?\\]|\\[p\\]|\\[~\\])(?=$|[\\s|\\|])`

- escaped characters
    - `\\s` spaces + tabs (vs the regular character `s`)
    - `\\|` pipe (vs `|` which has the special meaning `OR`)
    - `\\.` dot (vs `.` which has the special meaning `ANY`)
    - `\\(` brace (vs `(` which has the special meaning `START MATCH GROUP`) (same for `)`)
    - `\\[` bracket (vs `[` which has the special meaning `START SYMBOL GROUP`) (same for `]`)
    - `\\^` hat (vs `^` which has the special meaning `START OF LINE` (or `NOT` inside `[...]`))
    - `\\?` hat (vs `?` which has special meanings e.g. in look-arounds)
    - `\\+` plus (vs `+` which has the special meaning `AT LEAST ONE`)
    - `\\*` star (vs `*` which has the special meaning `AT LEAST ZERO`)
    - `\\$` dollar (vs `$` which has the special meaning `END OF LINE`)
    - `\\{` curly (vs `{` which can be used to set counts and ranges, e.g. `[a]{1,10}` meaning between 1 and 10 `a`s) (same for `}`)
- positive look-ahead
    - `(?<=^|[\\s|\\|])` makes sure that the next match group `(>|...)` is preceded (`(?<=...)`) by space (`\\s`), a pipe (`\\|`), or the beginning of the line (`^`) without those symbols being a part of the match
    - this means " S problem" and " S|? problem" will match the symbols, but "Some problem?" will not.
    - also note how `<` and `=` must not be escaped, they become special characters only in this context
    - `^` can't be inside the `[...]` since then it would be the `NOT` symbol (see [here](https://stackoverflow.com/a/9155707/7215915))
- positive look-behind
    - `(?=$|[\\s|\\|])` same as positive look-ahead but behind the previous match group

for text mate regex in general see https://macromates.com/manual/en/regular_expressions
