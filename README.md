# NixOS config, home-manager config with Flakes

First version:

```bash
nix-config/
├── flake.nix
├── flake.lock
├── hosts/
│   └── laptop.nix
├── modules/
│   ├── system/
│   │   └── common.nix
│   └── home/
│       └── user.nix
├── home/
│   └── laptop-user.nix
```

- System-level config goes in `hosts/` or `modules/system/`
- User-level config (Home Manager) goes in `home/` or `modules/home/`

Second version with `overlays/` and `pkgs/`:

🔍 Purpose `overlays/`:

- Used to override, patch, or extend existing packages in nixpkgs
- Also used to define custom packages or versions

📦 Examples:

- Patch Firefox to use a custom profile
- Replace the version of Neovim with one from GitHub
- Add a package that’s not in nixpkgs at all

🔍 Purpose `pkgs/`:

- Define custom packages or applications that are not in nixpkgs
- Think of this as your local nixpkgs subset

📦 Examples:

- A custom CLI tool you wrote
- A shell script you want to wrap as a package
- An overlay-independent custom build

Third version with `libs:

The `lib/` directory is for your custom helper functions and utilities written in Nix. Think of it as your personal toolbox 🧰 for reusable logic that doesn’t fit into a package, overlay, or module

## configuration.nix

In the flake-based NixOS config structure, the file `configuration.nix` is optional — but it's traditionally where all your system settings live when you’re not using flakes.
In non-flake NixOS setups, ``/etc/nixos/configuration.nix`` is the main entry point that defines your whole system: what packages are installed, what services are running, system users, host settings, etc.

But in a flake-based setup, you often split things up more cleanly:

- You define host-specific config in something like `hosts/laptop.nix`
- You define common modules in modules/*.nix
- And you reference those in flake.nix under nixosConfigurations

TLDR: If you're using flakes to manage your NixOS configuration, you do not need to use `configuration.nix`

## flake-utils

`flake-utils` is a helper library that simplifies the boilerplate and adds multi-platform support when you're working with Nix flakes.
Without `flake-utils`, you'd have to manually write out separate outputs (like `packages`, `devShells`, `nixosConfigurations`) per system architecture - `x86_64-linux`, `aarch64-darwin`, etc. It gets messy fast.

- Automatically loop over supported systems
- Define shared logic per system (like importing nixpkgs)
- Reduce repetition in your flake.nix

Without `flake-utils`

```nix
outputs = { nixpkgs, ... }: {
  packages.x86_64-linux.default = import nixpkgs { system = "x86_64-linux"; };
  packages.aarch64-linux.default = import nixpkgs { system = "aarch64-linux"; };
  ...
};
```

With flake-utils

```nix
outputs = { self, nixpkgs, flake-utils, ... }:
  flake-utils.lib.eachDefaultSystem (system:
    let pkgs = import nixpkgs { inherit system; };
    in {
      packages.default = pkgs.hello;
    }
  );
```
