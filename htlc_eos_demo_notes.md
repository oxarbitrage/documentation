GWallet
-------

Gwallet is a desktop Bitshares(Graphene) wallet or smart contract browser written in C++.
The idea is a framework for simple dialog forms to create complex operations in the blockchain in an easy manner. GWallet uses the wxWidgets libraries and it works in Windows, Linux and Mac.

Public repo: https://github.com/bitshares/gwallet

Issues: https://github.com/bitshares/gwallet/issues

For demostration purposes there is 1 node of bitshares running locally in this machine in a private testnet setup.

./gwallet

1- Show translation to chinese, connect, unlock, do a transfer and get back to english.

Note on translations are probably wrong as google translation service was used. Need human chinese-english translator to fix them.

2- Show poedit for translations.

3- Get back to english.

4- Restart, connect, unlock.

5- Use accounts combobox to switch between accounts. Demostrate in a transfer how the from field will change to the selected account. Same with a sell asset operation.

6- Similar with the asset combobox and how in the sell asset dialog the amount asset will be MPA(or the currently selected one)

7- While we are here create a sell asset operation selling 1 MPA for 1 BTS with the nathan account.

8- After it is open, demo the cancel order.

9- Extend all the command groups.

10- From the getters use Account History and show the last 25 operations made by nathan.

11- From the getters use Asset and check the Show output in the CLI to show command line functionality.

12- Show drag and drop of panels functionality.

----------------------------------------------------------------

HTLC between EOS and Bitshares
------------------------------

A prototype of EOS and Bitshares blockchains exchange of value in a secure way by HTLC.

Repo: https://github.com/jmjatlanta/htlc

EOS smart contract prototype developed by John Jones from the bitshares core team.

For demostration purposes there is 1 node of bitshares and 1 node of EOS running locally in this machine.

Alice and Bob need both to have accounts in Bitshares and EOS. For this demo we assume alice want 10 BTS and bob will pay with 10 EOS in a 1 to 1 price rate.

1- We have an alice and a bob in the gwallet already for the bitshares side.

2- unlock eos wallet:

cleos wallet unlock --password PW5JTqbh5JzUw9znoDF5UVVkoRVrDhhdMrBZo9eYWVU1Eu935NhKg

3- We have an alice and a bob in the eos side (EOS core token in private testnet = SYS):

cleos get currency balance eosio.token alice SYS
cleos get currency balance eosio.token bob SYS

4- HTLC creation on eos takes 2 steps:

- alice send money to the htlc contract on eos:

cleos push action eosio.token transfer '["alice", "htlc", "10.0000 SYS", "memo"]' -p alice@active

- alice build htlc contract on eos with hash of preimage, the secret word is `SuperSecret`.

https://passwordsgenerator.net/sha256-hash-generator/

cleos push action htlc build '["alice", "bob", "10.0000 SYS" "5899575803417E3356A133C51EFFF8314C0D3D7A52F37472F90B1DCE5288525B", "2019-07-12T00:00:00"]' -p alice@active

5- With the contract created there are 3 things alice will need to send to bob(by text msg, email, whatever):

- Preimage size: this is the number of characters the password haves, in the case of SuperSecret the size is 11.
- HTLC Contract Key obtained from htlc build command so bob can review the conditions in eos side.
- Preimage hash: 5899575803417E3356A133C51EFFF8314C0D3D7A52F37472F90B1DCE5288525B

6- Bob review the contract alice created in the eos side using the id:

cleos push action htlc reviewhtlc '["6"]' -p bob

7- If bob agree with that he will create a contract in the bitshares side with the data provided by alice:

GWallet -> HTLC -> create

8- Bob will use the getters - HTLC from gwallet in the bitshares side to get the id of the htlc contract he just created.

9- Bob sends the id of the htlc he just created to alice. (1.16.X)

10 - Alice will review the contract in the bitshares side by going to getters - htlc and inserting the id.

11- Alice can unlock the funds in the bitshares side by going to HTLC - Redeem and using the id and the password she already knows.

12- This exposes the password to Bob so he can unlock his funds in the eos side:

cleos push action htlc withdrawhtlc '["6", "SuperSecret"]' -p bob
