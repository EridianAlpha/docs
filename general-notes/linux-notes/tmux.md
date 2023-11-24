# TMUX

### TMUX Commands

| Description                      | Command                                                           |
| -------------------------------- | ----------------------------------------------------------------- |
| Create a new session             | `tmux new -s <SESSION_NAME>`                                      |
| See any existing sessions        | <p><code>tmux list-sessions</code></p><p><code>tmux ls</code></p> |
| Attach to existing session       | `tmux attach -t <SESSION_NAME>`                                   |
| Make all windows the same size   | `Ctrl+b` THEN `Alt+2`                                             |
| Add new tmux window vertically   | `Ctrl + b` THEN `Shift + @`                                       |
| Add new tmux window horizontally | `Ctrl + b` THEN `%`                                               |
| Detach from current session      | `Ctrl+b` THEN `d`                                                 |
| Kill current session             | `Ctrl+c`                                                          |

### Install and configure TMUX

{% code title="Install tmux" %}
```bash
sudo apt-get install -y tmux
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```
{% endcode %}

{% code title="Create new session" %}
```bash
tmux new -s <SESSION_NAME>
```
{% endcode %}

{% code title="- You have to start a session first before these config options exist" %}
```bash
tmux show -g | sed 's/^/set-option -g /' > ~/.tmux.conf
cd
vim .tmux.conf
```
{% endcode %}

* Change `mouse` to `on`

{% code title=".tmux.conf" %}
```bash
sudo apt-get install -y tmux
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```
{% endcode %}



