## snapshot 1:

Path 1:
/home/alfredo/CLionProjects/testnet_snaps/testnet_release
```
git clone https://github.com/bitshares/bitshares-core
git checkout -t origin/testnet_release
git submodule update --init --recursive
cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo .
make

./programs/witness_node/witness_node --plugins "snapshot market_history" --snapshot-at-time "2019-02-22T00:00:00" --snapshot-to "snap1.json"
```

## snapshot 2:

Path 2:
/home/alfredo/CLionProjects/testnet_snaps/testnet
```
git clone https://github.com/bitshares/bitshares-core
git checkout -t origin/testnet
git submodule update --init --recursive
cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo .
make

./programs/witness_node/witness_node --plugins "snapshot market_history" --snapshot-at-time "2019-02-22T00:00:00" --snapshot-to "snap2.json"
```
