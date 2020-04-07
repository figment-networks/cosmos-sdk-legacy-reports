# Cosmoslike Legacy Reports

This repository contains scripts used to generate back-filled income reports for Cosmos Hub 1 and Cosmos Hub 2. It should be possible to use it to do the same for any pre-0.37.0 Cosmos SDK based project.


## Dependencies

- Ubuntu 18.04 (tested)
- Python 3.6+
- Bash 4+
- ZFS (`apt install zfsutils-linux`)
- UFW enabled
- jq 1.5+ (`apt install jq`)
- sqlite3 (`apt install sqlite3 libsqlite3-dev`)
- `gaiad`/`gaiacli` built to appropriate version (Hub 1: `0.33.2`, Hub 2: `0.34.9`)
- password-less `sudo` for running user


## Usage

Make a ZFS pool, and a filesystem to contain your chosen chain:

```
sudo zpool create -f cosmos-legacy-reports /dev/sda
sudo zfs create cosmos-legacy-reports/hub1
sudo chown -R $USER /cosmos-legacy-reports/hub1
```

Setup gaiad/equivalent, for example:

```
gaiad0_34_9 init hub1reportstmp --home /cosmos-legacy-reports/hub1
```

Update configs:

- provide the appropriate `genesis.json` (https://archive.interchain.io/)
- update `config.toml`:
  - add persistent peers
  - remove/comment `index_tags`
  - ensure `index_all_tags = true`
  - ensure `pex = false`

Now we can generate snapshots for every report period on this chain (variables below are the defaults, except for `FINAL_BLOCK` and `DATA_DIR`, which are required):

```
# Ctrl-X Ctrl-E, paste this code, edit & save to run
#export RESET_DATA=1
export FINAL_BLOCK=500000
export DATA_DIR=/cosmos-legacy-reports/hub1
export NODE_BINARY=gaiad
export RPC_URL=localhost:26657
export REPORT_FREQUENCY="1 days"
screen -mS hub1-snapshots bash -c 'bash generate-snapshots.bash; exec bash'
```

This will take a very long time, but once it completes you can run the reports like so (again only required environment variable is `DATA_DIR`, the others are the defaults):

```
# Ctrl-X Ctrl-E, paste this code, edit & save to run
export DATA_DIR=/cosmos-legacy-reports/hub1
export NODE_BINARY=gaiad
export CLI_BINARY=gaiacli
export DENOM=uatom
screen -mS hub1-reports bash -c 'bash report-on-snapshots.bash; exec bash'
````
