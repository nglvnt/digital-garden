# Keyboard shortcut issues in VSCode

## General

In the Command Palette use `Preferences: Open Keyboard Shortcuts` to see the list of shortcuts.

In the Command Palette use `Developer: Toggle Keyboard Shortcuts Troubleshooting` to see what command is linked to a shortcut (it can happen that an extension overrides the default behaviour of the shortcut). "Keyboard event cannot be dispatched" means that an other software intercepts the shortcut, <https://defkey.com/shortcut-actions> can be used to track that software down.

## Show Explorer (Ctrl + Shift + e)

On Ubuntu, `Ctrl` + `Shift` + `e` is the shortcut for emoji entry. It can be disabled in the IBus Preferences (run `ibus-setup`), on the Emoji tab, change/delete the Emoji annotation keyboard shortcut. However, when using the Snap version of VSCode, one needs to take further actions to disable the emoji entry shortcut: <https://askubuntu.com/questions/1125726/how-to-disable-ctrl-shift-e-keybinding-from-showing-eeeee-and-loading-emoji-opti/1269239#1269239>

## Show Source Control (Ctrl + Shift + g)

Normally, `Ctrl` + `Shift` + `g` jumps to the source control, however, when the GitLens extension is installed, then by default it uses this shortcut as a prefix for its keybindings (GitLens keymap is "chorded"). In this case one can jump to source control using `Ctrl` + `Shift` + `g` `g`. Or one can set the GitLens keymap to "none" to restore the normal behaviour.

## Resources

<https://stackoverflow.com/questions/58946149/vscode-some-shortcuts-not-working-properly>

<https://askubuntu.com/questions/1083913/what-does-ctrl-shift-e-do-while-typing-text>

<https://askubuntu.com/questions/1125726/how-to-disable-ctrl-shift-e-keybinding-from-showing-eeeee-and-loading-emoji-opti/1269239#1269239>

<https://github.com/gitkraken/vscode-gitlens/issues/423>
