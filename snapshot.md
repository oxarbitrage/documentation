# Snapshot comparison in bitshares

When a new release is about to get merged into the `testnet` or into `master` it is a good practice to do a snapshot comparison between current and new code coming.

A snapshot is just a dump of the internal objects database and it is done by the `snapshot_plugin` at a user selected time or block number.

The following is an example of a comparison done between 2 branches that are about to get merged one into the other, in this case `testnet_release` is going to be merged into `testnet` with hardfork changes.

2 different builds need to be made, this process can be done in 2 different machines for speed purposes, following assumes just 1 machine and creates 1 snap after the other.

Checkpoints are needed as we are in the testnet, after building we are going to be taking a snapshot at `2019-02-22T00:00:00` which is current date.

## Snapshot 1 - testnet_release:

Path 1:
/root/testnet_snaps/testnet_release/bitshares-core
```
git clone https://github.com/bitshares/bitshares-core
git checkout -t origin/testnet_release
git submodule update --init --recursive
cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo .
make

./programs/witness_node/witness_node --plugins "snapshot market_history" --snapshot-at-time "2019-02-22T00:00:00" --snapshot-to "snap1.json" --checkpoint=[22668517,0159e4e579f49d76d221e5a756a88b38dc7ee9a2] --checkpoint=[22668518,0159e4e600cb149e22ef960442ca331159914617]
```

## Snapshot 2 - testnet:

Path 2:
/root/testnet_snaps/testnet/bitshares-core
```
git clone https://github.com/bitshares/bitshares-core
git checkout -t origin/testnet
git submodule update --init --recursive
cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo .
make

./programs/witness_node/witness_node --plugins "snapshot market_history" --snapshot-at-time "2019-02-22T00:00:00" --snapshot-to "snap2.json" --checkpoint=[22668517,0159e4e579f49d76d221e5a756a88b38dc7ee9a2] --checkpoint=[22668518,0159e4e600cb149e22ef960442ca331159914617]
```

## Compare

Compare the 2 result files simply by:

```
diff /root/testnet_snaps/testnet_release/bitshares-core/snap1.json /root/testnet_snaps/testnet/bitshares-core/snap2.json
```

## Analize results

If the objects changed, identify what parts of new code introduced are causing the changes and make sure that is ok to be happening. Check https://github.com/bitshares/bitshares-core/pull/1609#issuecomment-466768898 as an example.
