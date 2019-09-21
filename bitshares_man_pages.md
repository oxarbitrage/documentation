# Bitshares Manual pages

Manual pages(aka: man pages) are well known software documents for programs and libraries in the Unix platforms. This document describes how to create them for 2 binaries present in the **bitshares-core** project: `witness_node` and `cli_wallet`.

## Generate manual pages based on commands help

We already have available `./witness_node --help` and `./cli_wallet --help`. We are going to use this output to generate unix manual pages.

https://www.gnu.org/software/help2man/ is a tool for this purpose. To install it in Linux we do:

`sudo apt install help2man`

We have all needed to create a decent looking man page automatically inside the program folders of the 2 binaries. The `.h2m` are static files used to add some additional info to the output.

The command to create the manual page for the witness_node is the following:

```
help2man ./witness_node --include witness_node.h2m —no-info > witness_node.1
```

To see our creation we just do:

```
man ./witness_node.1
```

Similar to this we can do the same for the command line wallet binary:

```
help2man ./cli_wallet --include cli_wallet.h2m --no-info > cli_wallet.1
man ./cli_wallet.1
```

**This should be done after each release and this file should be linked in the BitShares Git Flow process**


## man2html

Now that we have a manual page we can install another tool:

`sudo apt install man2html-base`

Once we have it we can use our man pages to generate html:

```
man2html ./witness_node.1 > witness_node.html
```

And for the cli wallet:

```
man2html ./cli_wallet.1 > cli_wallet.html

```


https://open-explorer.io/man/witness_node.html

https://open-explorer.io/man/cli_wallet.html

**I dont have an official place for this files yet if we want to implement them**

