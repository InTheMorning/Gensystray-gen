
# Gensystray-gen

Generates a configuration for [Gensystray - *generic system tray*](https://github.com/dardevelin/gensystray).

Searches a path for files you want and add an entry in gensystray for each file, then creates/overwrites a gensystray.cfg file in a named subfolder in `~/.config/gensystray`.  The gensystray menu will also contain a *New File* entry that prompts the creation of a new entry using [Zenity](https://gitlab.gnome.org/GNOME/zenity)

Unless it is run with the '-n' argument, it also autogenerates the following files in `~/.config/systemd/user/`:
- `gensystray@<name>.service` the systemd instance to run gensystray
- `gensystray-refresh@<name>.path` to refresh entries in gensystray when file modifications are detected


## Usage

The systray icon can be started/stopped using
```
% systemctl --user start/stop gensystray@<name>
```
To enable this tray icon at startup, add the above `systemctl start` command to your desktop enviroment's autostart


## Examples

Make a new gensystray icon called 'notes' that shows a listing of `.md` files in the `~/Notes/` folder:
```
% gensystray-gen.py Notes -d ~/Notes -e .md
```


Using the **-c** argument to run scripts using a specific command:
```
% gensystray-gen.py Scripts -d ~/Scripts/ -c 'xfce4-terminal -H -e'
```


## ToDo

+ Add an option for custom editor command for *New File* entry
+ Find the correct path to the gensystray-gen executable
