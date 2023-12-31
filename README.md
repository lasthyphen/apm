# Dijets Plugin Manager (apm)

**Note: This code is currently in Alpha. Proceed at your own risk.**

`apm` is a command-line tool to manage ECC Engines binaries for
[dijetsgo](https://github.com/lasthyphen/dijetsgo).

`apm` allows users to build their own custom repositories to provide ECC Engine and subnet definitions outside of
the [dijets-plugins-core](https://github.com/lasthyphen/dijets-plugins-core) repository. `dijets-plugins-core`
is a community-sourced set of plugins and subnets that ships with the `apm`, but users have the option of adding their own using
the `add-repository` command.

## Installation

### Pre-Built Binaries
#### Instructions

To download a binary for the latest release, run:

```
curl -sSfL https://raw.githubusercontent.com/lasthyphen/apm/master/scripts/install.sh | sh -s
```

The binary will be installed inside the `./bin` directory (relative to where the install command was run).

_Downloading binaries from the Github UI will cause permission errors on Mac._

To add the binary to your path, run

```
cd bin
export PATH=$PWD:$PATH
```

To add it to your path permanently, add an export command to your shell initialization script (ex: .bashrc).

#### Installing in Custom Location

To download the binary into a specific directory, run:

```
curl -sSfL https://raw.githubusercontent.com/lasthyphen/apm/master/scripts/install.sh | sh -s -- -b <relative directory>
```

### Source
If you are planning on building from source, you will need [golang](https://go.dev/doc/install) >= 1.18.x installed.

To build from source, you can use the provided build script from the repository root.
```
./scripts/build.sh
```
The resulting `apm` binary will be available in `./build/apm`.

## Commands

### add-repository
Starts tracking a plugin repository.

```shell
apm add-repository --alias ava-labs/core --url https://github.com/lasthyphen/avalanche-plugins-core.git --branch master
```

#### Parameters:
- `--alias`: The alias of the repository to track (must be in the form of `foo/bar` i.e organization/repository).
- `--url`: The url to the repository.
- `--branch`: The branch name to track.
 
### install-vm
Installs a ECC Engine by its alias. Either a partial alias (e.g `spacesvm`) or a fully qualified name including the repository (e.g `ava-labs/core:spacesvm`) to disambiguate between multiple repositories can be used.

If multiple matches are found (e.g `repository-1/foovm`, `repository-2/foovm`), you will be required to specify the
fully qualified name of the ECC Engine to disambiguate the repository to install from.

This will install the ECC Engine binary to your `avalanchego` plugin path.

```shell
apm install-vm --vm spacesvm
```

#### Parameters:
- `--vm`: The alias of the VM to install.


### join-subnet
Joins a subnet by its alias. Either a partial alias (e.g `spaces`) or a fully qualified name including the repository (e.g `ava-labs/core:spaces`) to disambiguate between multiple repositories can be used.

This will install dependencies for the subnet by calling `install-vm` on each ECC Engine required by the subnet.

If multiple matches are found (e.g `repository-1/foo`, `repository-2/foo`), you will be required to specify the
fully qualified name of the subnet definition to disambiguate the repository to install from.


```shell
apm join-subnet --subnet spaces
```

#### Parameters:
- `--subnet`: The alias of the VM to install.

### list-repositories
Lists all tracked repositories.

```shell
apm list-repositories
```

### uninstall-vm
Installs a ECC Engine by its alias.

If multiple matches are found (e.g `repository-1/foovm`, `repository-2/foovm`), you will be required to specify the
fully qualified name of the ECC Engine to disambiguate the repository to install from.

This will remove the ECC Engine binary from your `avalanchego` plugin path.

```shell
apm uninstall-vm --vm spacesvm
```

#### Parameters:
- `--vm`: The alias of the VM to uninstall.

### update

Fetches the latest plugin definitions from all tracked repositories.


```shell
apm list-repositories
```

### upgrade

Upgrades a ECC Engine binary. If one is not provided, this will upgrade all ECC Engine binaries in your
`avalanchego` plugin path with the latest synced definitions.

For a ECC Engine to be upgraded, it must have been installed using the `apm`.

```shell
apm upgrade
```

#### Parameters
- `--vm`: (Optional) The alias of the VM to upgrade. If none is provided, all VMs are upgraded.

### remove-repository
Stops tracking a repository and wipes all local definitions from that repository.

```shell
apm remove-repository --alias organization/repository
```

#### Parameters:
- `--alias`: The alias of the repository to start tracking.

## Examples

###
1. Install the spaces subnet!
```shell
./build/apm join-subnet --subnet spaces
```

2. You'll see some output like this:
```text
$ ./build/apm join-subnet --subnet spaces

Installing ECC Engines for subnet Ai42MkKqk8yjXFCpoHXw7rdTWSHiKEMqh5h8gbxwjgkCUfkrk.
Downloading https://github.com/lasthyphen/spacesvm/archive/refs/tags/v0.0.3.tar.gz...
HTTP response 200 OK
Calculating checksums...
Saw expected checksum value of 1ac250f6c40472f22eaf0616fc8c886078a4eaa9b2b85fbb4fb7783a1db6af3f
Creating sources directory...
Unpacking ava-labs/avalanche-plugins-core:spacesvm...
Running install script at scripts/build.sh...
Building spacesvm in ./build/sqja3uK17MJxfC7AN8nGadBw9JK5BcrsNwNynsqP5Gih8M5Bm
Building spaces-cli in ./build/spaces-cli
Moving binary sqja3uK17MJxfC7AN8nGadBw9JK5BcrsNwNynsqP5Gih8M5Bm into plugin directory...
Cleaning up temporary files...
Adding ECC Engine sqja3uK17MJxfC7AN8nGadBw9JK5BcrsNwNynsqP5Gih8M5Bm to installation registry...
Successfully installed ava-labs/avalanche-plugins-core:spacesvm@v0.0.4 in /Users/joshua.kim/go/src/github.com/lasthyphen/avalanchego/build/plugins/sqja3uK17MJxfC7AN8nGadBw9JK5BcrsNwNynsqP5Gih8M5Bm
Updating ECC Engines...
Node at 127.0.0.1:9650/ext/admin was offline. ECC Engines will be available upon node startup.
Whitelisting subnet Ai42MkKqk8yjXFCpoHXw7rdTWSHiKEMqh5h8gbxwjgkCUfkrk...
Finished installing ECC Engines for subnet Ai42MkKqk8yjXFCpoHXw7rdTWSHiKEMqh5h8gbxwjgkCUfkrk.
```

### Setting up Credentials for a Private Plugin Repository
You'll need to specify the `--credentials-file` flag which contains your github personal access token. 

Example token file:
```
username: joshua-kim (for GitHub, this field doesn't matter. You can use your username as a placeholder)
password: <personal access token here>
```

Example command to download a subnet's VMs from a private repository:
```
apm join-subnet --subnet=foobar --credentials-file=/home/joshua-kim/token
```
