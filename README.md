# Blackfly's fork of Luke's Auto-Rice Bootstraping Scripts (LARBS)

## Installation:

On an Arch-based distribution as root, run the following:

```sh
curl -LO https://raw.githubusercontent.com/theblackfly/LARBS/blackfly/larbs.sh
sh larbs.sh
```

That's it.

## What is LARBS?

LARBS is a script that autoinstalls and autoconfigures a fully-functioning
and minimal terminal-and-vim-based Arch Linux environment.

LARBS can be run on a fresh install of Arch or Artix Linux, and provides you
with a fully configured diving-board for work or more customization.

## Customization

By default, LARBS uses the programs [here in progs.csv](progs.csv) and installs
[my dotfiles repo (voidrice) here](https://github.com/theblackfly/voidrice),
but you can easily change this by either modifying the default variables at the
beginning of the script or giving the script one of these options:

- `-r`: custom dotfiles repository (URL)
- `-p`: custom programs list/dependencies (local file or URL)
- `-a`: a custom AUR helper (must be able to install with `-S` unless you
  change the relevant line in the script

### The `progs.csv` list

LARBS will parse the given programs list and install all given programs. Note
that the programs file must be a three column `.csv`.

The first column is a "tag" that determines how the program is installed, ""
(blank) for the main repository, `A` for via the AUR or `G` if the program is a
git repository that is meant to be `make && sudo make install`ed.

The second column is the name of the program in the repository, or the link to
the git repository, and the third column is a description (should be a verb
phrase) that describes the program. During installation, LARBS will print out
this information in a grammatical sentence. It also doubles as documentation
for people who read the CSV and want to install my dotfiles manually.

Depending on your own build, you may want to tactically order the programs in
your programs file. LARBS will install from the top to the bottom.

If you include commas in your program descriptions, be sure to include double
quotes around the whole description to ensure correct parsing.

### The script itself

The script is extensively divided into functions for easier readability and
trouble-shooting. Most everything should be self-explanatory.

The main work is done by the `installationloop` function, which iterates
through the programs file and determines based on the tag of each program,
which commands to run to install it. You can easily add new methods of
installations and tags as well.

Note that programs from the AUR can only be built by a non-root user. What
LARBS does to bypass this by default is to temporarily allow the newly created
user to use `sudo` without a password (so the user won't be prompted for a
password multiple times in installation). This is done ad-hocly, but
effectively with the `newperms` function. At the end of installation,
`newperms` removes those settings, giving the user the ability to run only
several basic sudo commands without a password (`shutdown`, `reboot`,
`pacman -Syu`).

## What's different from Luke's configuration

### Main Changes

- `xbacklight` does not work on some laptops. So my dwm config uses `light` instead. Check [this](https://stackoverflow.com/questions/23866335/arch-xbacklight-no-outputs-have-backlight-property) for more info.
- The [dotfiles repository](https://github.com/theblackfly/voidrice) is cloned into `~/voidrice` and the deployed to the users home directory by changing git worktree. See `putdotfiles` in the [larbs.sh script file](larbs.sh) for more info.
- [slock](https://github.com/theblackfly/slock) (my patched version of suckless' slock).
- `bluez`, `bluez-utils` and `pulseaudio-modules-bt` are installed for using bluetooth headsets.
- `youtube-dl` has been removed. If you need this tool, use `pip` to install it with `pip install -U youtube-dl`.

## Firmware

If you see something like, `WARNING: Possibly missing firmware for module:
wd719x`, install the missing firmware from AUR.

```sh
paru -S wd719x-firmware
paru -S aic94xx-firmware
paru -S upd72020x-fw
```

## HDMI Audio

```sh
sudo pacman -Syu xf86-video-ati
```

## Eduroam

Example for HS-Mittweida, Germany:

```sh
sudo pacman -S python-dbus
wget -O eduroam.py https://cat.eduroam.org/user/API.php?action=downloadInstaller&api_version=2&lang=en&device=linux&profile=6198
python eduroam.py
```

See <https://cat.eduroam.org/> for more info.

## Still more programs?

### Emacs with multiple configurations

```sh
sudo pacman -S emacs

git clone -b develop https://github.com/hlissner/doom-emacs ~/.local/src/doom-emacs
cd ~/.local/src/doom-emacs/bin
doom install

git clone -b develop https://github.com/syl20bnr/spacemacs ~/.local/src/spacemacs

git clone https://github.com/plexus/chemacs.git ~/.local/src/chemacs
sh ~/.local/src/chemacs/install.sh

mkdir -p ~/.config/emacs
touch ~/.config/emacs/init.el

echo -e '
(("default"       . ((user-emacs-directory . "~/.local/src/doom-emacs")))
 ("vanilla-emacs" . ((user-emacs-directory . "~/.config/emacs")))
 ("doom-emacs"    . ((user-emacs-directory . "~/.local/src/doom-emacs")))
 ("spacemacs"     . ((user-emacs-directory . "~/.local/src/spacemacs"))))
' > ~/.emacs-profiles.el

mkdir -p ~/.local/bin/chemacs
for emacsconfig in "vanilla-emacs" "doom-emacs" "spacemacs"
do
    echo "emacs --with-profile $emacsconfig" > ~/.local/bin/chemacs/$emacsconfig
    chmod +x ~/.local/bin/chemacs/$emacsconfig
done

source ~/.profile
```

Now, you should be able to find `spacemacs` and `doom-emacs` from dmenu. You may
have to first delete `~/.cache/dmenu_run` though.

To get the most out of doom-emacs you may also want to install any missing
programs depending on your doom-emacs configuration. Run `doom doctor` to check
this.

The user configuration files for doom-emacs and spacemacs can be found under
`DOOMDIR` and `SPACEMACSDIR` respectively. This is set from `~/.zprofile`.
doom-emacs can be updated by simply running `doom upgrade`. To update spacemacs,
first run:

```sh
cd ~/.local/src/spacemacs
git pull
```

and then restart spacemacs.

Run `git config --global github.oauth-token <token>` with a token generated from
https://github.com/settings/tokens in order to use GitHub from
[magit](https://www.spacemacs.org/layers/+source-control/github/README.html).

### Topgrade (https://github.com/r-darwish/topgrade)

``` sh
sudo pacman -Syu rustup
git clone git@github.com:r-darwish/topgrade.git ~/.local/src/topgrade
cargo install cargo-update
cargo install --path ~/.local/src/topgrade
```

### Delta (https://github.com/dandavison/delta/)

```sh
cargo install git-delta
```

### Python dev

```sh
pip install -U yapf
pip install -U autoflake
pip install -U pyflakes
pip install -U isort
pip install -U pipenv
pip install -U nose
pip install -U pytest
pip install -U importmagic
pip install -U epc
pip install -U ptvsd
```

If you would like to use Microsoft's pyright language server, install it with:

```sh
paru -S npm
npm config set cache $XDG_CACHE_HOME/npm
sudo npm install -g pyright
```

### Julia dev

```sh
julia -E 'using Pkg; Pkg.add("IJulia")'
julia -E 'using Pkg; Pkg.add("Debugger")'
julia -E 'using Pkg; Pkg.add("JuliaFormatter")'
julia -E 'using Pkg; Pkg.add("LanguageServer")'
julia -E 'using Pkg; Pkg.add("Pluto")'
julia -E 'using Pkg; Pkg.add("PyCall"); Pkg.add("RCall")'
julia -E 'using Pkg; Pkg.add("Revise")'
julia -E 'using Pkg; Pkg.add("Documenter")'
```

### Go dev

```sh
go get -u github.com/fatih/gomodifytags
go get -u github.com/nsf/gocode
go get -u github.com/cweill/gotests
go get -u github.com/motemen/gore
go get -u zgo.at/guru
```

### Hugo

```sh
git clone https://github.com/gohugoio/hugo.git ~/.local/src/hugo
cd ~/.local/src/hugo
go install --tags extended
```

### Web dev

```sh
paru -S js-beautify
paru -S stylelint
```

### Visual Studio Code extensions

```sh
code --install-extension bodil.file-browser
code --install-extension cometeer.spacemacs
code --install-extension jacobdufault.fuzzy-search
code --install-extension kahole.magit
code --install-extension ms-azuretools.vscode-docker
code --install-extension ms-python.python
code --install-extension ms-python.vscode-pylance
code --install-extension ms-toolsai.jupyter
code --install-extension njpwerner.autodocstring
code --install-extension ms-vscode-remote.remote-containers
code --install-extension ms-vscode-remote.remote-ssh
code --install-extension ms-vscode-remote.remote-ssh-edit
code --install-extension ms-vsliveshare.vsliveshare
code --install-extension vscode-icons-team.vscode-icons
code --install-extension vscodevim.vim
code --install-extension VSpaceCode.vspacecode
code --install-extension VSpaceCode.whichkey
```
