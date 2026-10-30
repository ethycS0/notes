## Dev Environment flakes
### Pure Flakes
```Nix
{
	# Specify Description Here
	description = "Nice Project";
	
	# External dependancies locked to specific git commits
	inputs = {
		nixpkgs.url = "github:nixos/nixpkgs/nixpkgs-unstable";
	};
	
	# Function that takes inputs and produces builds/shells
	# '...' allows the function to accept extra arguments without crashing
	outputs = { self, nixpkgs, ... }: let
		# Define variables that you will use later
		system = "x86_64-linux";
		
		# Flake-native way to import packages for our architecture
		pkgs = nixpkgs.legacyPackages.${system};
		
		# Legacy way, requires flake to reval nixpkgs
		pkgs = import nixpkgs { inherit system; };
		
	in {
	
		# Development environments (triggered via 'nix develop') 
		devShells.${system}.default = pkgs.mkShell {
		
			# buildInputs, packages or nativeBuildInputs
			packages = with pkgs; [
		          gcc
		          gnumake
		          bear	
			];
			
			# Shell commands or bash scripts here
			shellHook = "";
		};
	};
}

```

### flake-utils
```Nix
{

	description = "With flake-utils";

	inputs = {
		nixpkgs.url = "github:nixos/nixpkgs/nixpkgs-unstable";
		
		# Get flake-utils dependancies
		flake-utils.url = "github:numtide/flake-utils";
	};
	- [ ] 
	outputs = { self, nixpkgs, flake-utils, ... }: 
	
	# flake-utils allows usage of flake regardless of system
	# Following function loops over and works for default archs
	# x86_64-linux aarch64-linux x86_64-darwin aarch64-darwin
    flake-utils.lib.eachDefaultSystem (
		system:
		let
			pkgs = nixpkgs.legacyPackages.${system};
		in {
			# No need to define arch/system here
			devShells.default = pkgs.mkShell {
				packages ....
			};
		};
	);
}
```