# omafly

An Omarchy overlay panel for [gnat](https://github.com/lubabs770/gnat), the
fruit fly that is simulated from its connectome and walks on your Hyprland
windows.

Open it and you see what the fly is doing (walking, grooming, flying, asleep),
how many flies there are, the population firing rate, the neuron count, how
many window ledges it can see, and whether the brain view is open. From there
you can pause, scare, open the brain view, add or remove flies, or quit. If no
fly is running, you get a start button instead. Colours come from the shell's
live `[menu]` theme, so it follows every theme switch.

## Install

omafly is only the panel. The fly itself is the `gnat` binary, so you need both.

**1. Install gnat v0.1.0.** Download the release, check it against the
published SHA-256, then unpack it and link the binary onto `PATH` (x86-64 Linux):

```sh
curl -fLO https://github.com/lubabs770/gnat/releases/download/v0.1.0/gnat-linux-x86_64.tar.gz
echo "cee5f89445b6c6dc3435f3750f77fcb2dd8b02c40da4f24e52cfad82066eabc0  gnat-linux-x86_64.tar.gz" | sha256sum -c
mkdir -p ~/.local/share ~/.local/bin
tar -C ~/.local/share -xzf gnat-linux-x86_64.tar.gz
ln -sfn ~/.local/share/gnat/gnat ~/.local/bin/gnat
```

`sha256sum -c` stops with `FAILED` if the download is not the reviewed build;
don't unpack it then. The release is
[immutable](https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/immutable-releases),
so the tag and its asset can't be replaced after publishing. With `gh`, you can
also check the file against GitHub's release attestation:
`gh release verify-asset v0.1.0 gnat-linux-x86_64.tar.gz -R lubabs770/gnat`.
The binary finds its connectome in `data/` beside it, which the archive
includes.

To build from source instead, check out the same tag:

```sh
git clone --branch v0.1.0 https://github.com/lubabs770/gnat
cd gnat && cargo build --release --locked
ln -sfn "$PWD/target/release/gnat" ~/.local/bin/gnat
```

**2. Add the plugin:**

```sh
omarchy plugin add https://github.com/lubabs770/omafly
```

From a checkout of this repo, link it in instead:

```sh
ln -sfn "$PWD" ~/.config/omarchy/plugins/io.github.lubabs770.omafly
omarchy-shell -q shell rescanPlugins && omarchy plugin enable io.github.lubabs770.omafly
```

## Use

```sh
omarchy-shell shell toggle io.github.lubabs770.omafly
```

Bind that to a key in `~/.config/hypr/bindings.lua`, or use it as the
`on-click` of gnat's Waybar module.

| Key | Does |
|---|---|
| Space | pause / resume |
| `s` | scare |
| `b` | open the brain view |
| `-` `+` | one fewer / one more fly (1 to 64) |
| `q` | quit the fly |
| Enter | start a fly, when none is running |
| Esc | close (or click outside the card) |

## Remove

```sh
omarchy plugin remove io.github.lubabs770.omafly   # unlinks the plugin; nothing else is touched
gnat quit; rm ~/.local/bin/gnat                   # and gnat, if you no longer want it
rm -r ~/.local/share/gnat                          # (or your gnat checkout)
```

## What it does and doesn't touch

- **Writes nothing.** No files and no config. The panel's only state is in
  memory while it is open.
- **Runs only `gnat`.** It finds the binary through `$GNAT_BIN`, then `PATH`,
  then `~/.local/bin`, and never through a folder the shell hands it. If there
  is none, the panel says so instead of showing an empty card. Every action is a
  `gnat` subcommand (`status`, `toggle`, `scare`, `brain`, `flies N`, `quit`,
  `--run`) run as an argument vector, and no shell string is built from data.
- **Doesn't trust what it reads.** gnat answers over its control socket in
  `$XDG_RUNTIME_DIR`, which is user-private. Each `status` reply is capped at
  4 KiB and must carry every expected field with the expected type, or it is discarded.
  Error text is cut to one line, and every `Text` renders as `PlainText`.

## Files

| File | What it is |
|---|---|
| `manifest.json` | plugin id `io.github.lubabs770.omafly`, kind `overlay` |
| `qml/Overlay.qml` | the shell host: the layer-shell window and the theme tokens |
| `qml/Panel.qml` | the panel, and the calls to `gnat` |
| `qml/Tokens.qml`, `qml/FlatButton.qml` | colours and the button, shared with [optination](https://github.com/lubabs770/optination) |

## License

MIT. gnat, which this drives, is MIT with CC BY-NC 4.0 connectome data; see its
repository.
