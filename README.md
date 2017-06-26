# Readline Proxy

This enables custom interactive REPLs, while still providing
(as far as possible) normal shell/readline/history behaviour and without leaving the current process.


## Example: Python w/o Shell Expansion

This is an example where we decided to run expressions within a python
interpreter if their first line starts with a space.

[![asciicast](https://asciinema.org/a/ECilkB2ymMV12txtqSRVjvZJd.png)](https://asciinema.org/a/ECilkB2ymMV12txtqSRVjvZJd)

This is for sure not super useful as is, but think of `python -c` being replaced by e.g. interactions with a (stateful) (co)process via bash functions.

## Usage

source the script and supply the functions on top with your own versions.  
You are getting called for

- the welcome message
- the decision if an entered line is 'special'
- before such a 'special' expression is pushed to the history and run, so that you can
  decide what to really run.


### WTFs

Here are known differences to a normal shell session after the script is sourced.

> Those only apply when you enter statements interactively

Behaviour when sourcing scripts or running them should be of no difference.


#### Multiline Expressions

If a line is not 'special', i.e. entered bash expressions we must support multiline entry mode. We are by far not as smart as bash regarding that - but hopefully tolerable.

Currently we look at (whitespace cleared) endings of those lines to derive that we shall switch to multiline mode automatically.

We set multiline mode if the right-stripped first line ends with

- ';'
- ' do'
- '$' (convention to force multiline mode. the character is removed at evaluation time)


- Biggest wtf: You have to end them with a dot, to trigger evaluation. Sorry, not yet understood
  how bash knows when to evaluate. Trying to catch `syntax error: unexpected end of file` on
  stderr was a dead end road in deed (`tee`-ing stderr when user command is e.g. `bash` is a blocker).
- (Only) on the first line of a multiline expression you have to supply either ";"
- Here document first lines and terminators are detected without any begin/end indicators.
- We store multiline expressions "as is" - i.e. w/o merging them into a single line, using semicolons as bash does.


#### Prompt

- Normally you see your command on stdout before the result is shown. Now you
  get it on stderr and the prompt is just a $ infront of it



## Tech

bash allows to provide custom key-bindings via the `bind` function.   
This passes any enter-key hit into a custom function:

    bind -x '"\C-M": "rl_handle_enter"'

while normal behaviour is this:

    1 ~ $ bind -P |grep accept
    accept-line can be found on "\C-j", "\C-m".

Since it seems not possible to invoke readline's standard accept-line behaviour  in a custom enter handler, this repo was created.

What it does is basically:

- checks for customizable criteria if the current line should be treated "special"
- checks if it looks like a bash multiline expression and starts asking for
  more input until that multiline expression is deemed complete (e.g. here doc
  terminator or '.' entered)
- when an expression is deemed complete it is pushed to the history - and run
  via `fc -s`.
- fc repeats the command on stderr, we just add a "$" prompt before it.


