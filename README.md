# Work in progress

## Todo / Bugs

- [ ] Add notify feature for desktop notifications
- [ ] Split tunneling
- [ ] Nix Darwin module
- [ ] Openvpn process not being killed if daemon is shut down before connection is established
- [ ] Set up test suite

## Installation

### Cargo

```sh
cargo install --git https://github.com/juliamertz/protonvpn-rs --branch main
```

### Nix

You can install/download the program with the nix package manager by running

```sh
nix run "github:juliamertz/protonvpn-rs?dir=nix"
# or
nix install "github:juliamertz/protonvpn-rs?dir=nix"
```

There is also a NixOS module included, a darwin module is planned.

<details>
  <summary>Example config</summary>

```nix
{ inputs, ... }: {
  imports = [ inputs.protonvpn-rs.nixosModules.protonvpn ];

  services.protonvpn = {
    enable = true;
    requireSops = true;
    settings = {
      credentials_path = "/run/secrets/openvpn_auth";
      autostart_default = true;
      default_select = "Fastest";
      default_protocol = "Udp";
      default_criteria = {
        country = "NL";
        features = [ "Streaming" ];
      };
      killswitch = {
        enable = false;
        custom_rules = [
          "-a input -s 192.168.0.100 -j accept"
          "-a output -d 192.168.0.100 -j accept"
          "-a input -p tcp -m tcp --dport 22 -j accept"
        ];
      };
    };
  };
}
```

</details>

## Features

### Filtering servers

There are many flags to filter servers, these can be used with the `connect` and `query` subcommands.
| Flag | Description |
| --------------------------|----------------------------------------------------------------------------|
| `-c`, `--country` | Filter servers by country [possible values: [here](src/api/types.rs#L132)] |
| `-t`, `--tier <tier>` | servers by tier [possible values: free, premium, all] |
| `-s`, `--sort <sort>` | [possible values: speed, load] |
| `--max-load <max-load>` | servers by load |
| `--tor` | include servers with the Tor feature |
| `--p2p` | include servers with the P2P feature |
| `--secure-core` | include servers with the Secure Core feature |
| `--streaming` | include servers with the Streaming feature |

### Protocol

You can use either `udp` or `tcp`, change this with the command line flag `--port` or `-p`.
To set a default, change the value of `default_protocol` in your config file.

### Killswitch

Enabling the killswitch will apply some firewall rules that only allow traffic to pass through the openvpn tunnel.
You can enable the killswitch by running `pvpn killswitch enable` or set the `killswitch.enable` config option to `true`

If you require extra firewall rules you can add these under `killswitch.custom_rules`, for example:

```ron
killswitch: (
  enable: true,
  custom_rules: Some([
    "-A INPUT -s 192.168.0.100 -j ACCEPT",
    "-A OUTPUT -d 192.168.0.100 -j ACCEPT",
    "-A INPUT -p tcp -m tcp --dport 22 -j ACCEPT"
  ]),
),
```

#### Notes

**Linux**

> Warning! this alters your iptables, if you're using a non-standard setup make sure the new rules don't conflict

**Macos**

> In order to use the killswitch you have to enable the system filewall, this can be done in the system settings under `Network > Firewall`
