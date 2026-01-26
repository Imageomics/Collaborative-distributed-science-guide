# Test File for Markdownlint Config


## Rule MD007: Indentation (Should allow 4 spaces)

* Level 1
    * Level 2 (This uses 4 spaces. If the linter errors, the config isn't working.)

## Rule MD013: Line Length (Should be ignored)

This is an incredibly long line of text that exceeds the default 80-character limit usually enforced by markdownlint. Because MD013 is set to false, this line should not trigger any warnings or errors regardless of how long it gets.

## Rule MD026: Trailing Punctuation (Should allow only specific marks)

### This header ends with a period.
### This header ends with a question mark?
### This header ends with an exclamation point!

## Rule MD040: Fenced Code Language (Should be ignored)

```
This code block has no language specified. Normally, this triggers a warning, but it should be ignored now.
```

## Rule MD046: Code Block Style (Should be ignored)

    This is an indented code block (4 spaces).
    Normally, markdownlint prefers fenced blocks (```), but this should be ignored.

## Hard Tabs (Should be allowed)

*	This line uses a hard tab after the bullet point instead of a space.
