# P2SH
Tutorial on how to deal with very basic P2SH transactions on the Dogecoin blockchain.

This tutorial is based on https://github.com/BlockchainCommons/Learning-Bitcoin-from-the-Command-Line/blob/master/10_2_Building_the_Structure_of_P2SH.md
and the available information on https://en.bitcoin.it/wiki/Script

No warranty or guarantee of any kind! Use at own risk!

## Dependencies
You will need a 
* Dogecoin Core Wallet to create raw transactions, decode the script and broadcast the TXs (https://github.com/dogecoin/dogecoin)
* btcc and btcdeb to serialize the script and for debugging (https://github.com/bitcoin-core/btcdeb)

## Installation script for Windows
Run this commands in your terminal app in your Ubuntu profile:

Required libraries/packages
```shell
sudo apt-get install unzip
apt-get install libtool libssl-dev autoconf pkg-config
sudo apt install g++
sudo apt install make
```

Compile btcc/btcdeb
```shell
wget https://github.com/bitcoin-core/btcdeb/archive/refs/tags/0.3.20.zip
unzip 0.3.20.zip
cd btcdeb-0.3.20/
./autogen.sh
./configure --disable-dependency-tracking
make
sudo make install
```

# Tutorial
The next steps will guide you through the how to.

We will create a script, derive the address from it, fund it, create a redeem script, debug it, create a transaction, sign it manually and broadcast it to the network.

## 1. Create a locking script
The locking script in a human readable way:

x + 7 = 10

This translates to a the following commands:
7 OP_ADD 10 OP_EQUAL

And btcc will output the serialized script:
![image](https://user-images.githubusercontent.com/54002590/174666751-069ecd69-94fc-4f25-90eb-ef97353f89d6.png)

## 2. Derive the address from the script and fund it
To add funds protected by the locking script we will now derive an Dogecoin address form that locking script with the following commands:
decodescript 57935a87

![image](https://user-images.githubusercontent.com/54002590/174667130-958c97fb-ef53-4c80-9670-db92a592a085.png)

For this tutorial I sent 69 Doges to the given address: https://sochain.com/tx/DOGETEST/36695e0bfdec75f106c53634065a8f7b7a998810fde21a5dff424631c5a9c3b6

For sending funds to that address no special logic is required.

## 3. Create and debug the redeem script
The solution for our locking script (x + 7 = 10) is to replace x with 3 - or to push 3 on top of our stack:
3 7 OP_ADD 10 OP_EQUAL

We can debug this script with btcdeb like this:
![image](https://user-images.githubusercontent.com/54002590/174668587-8a830681-6d6c-4e41-a219-aa6ef9228f4b.png)

This means we can add our number 3 (serialized 53) to the locking script to have the redeem script.


## 4. Build the redeem transaction
The redeem script testetd in 3. Create and debug the redeem script was correct, what means we can use that script to redeem the input of 69 Doges we locked with this script before (transaction 526d7a7def36c4b61dedf9acbb50f57fa1421cac5fc7342bf2d83699d377443b, vout 1).

I'll send 68 Doges to nWUGVFF3keeUMbNVKPYcnGE8P8KiQ4rThY and the remaining 1 Doges will be fee for the miner:

![image](https://user-images.githubusercontent.com/54002590/174669000-030ff1b9-22c4-4209-9b80-28f3ac7be700.png)

The redeemscript or the signing is still missing, what we will add per hand now.

To be able to do this, we need to know where to place the redeem script and what extra data needs to be added.

Raw transaction from the debug window:
0100000001b6c3a9c5314642ff5d1ae2fd1088997a7b8f5a063436c506f175ecfd0b5e69360100000000ffffffff0100c44f95010000001976a91418fc1a2ce6106cef632ac755f5d15439565b6d7e88ac00000000

The empty script right now is the "0" right before this part "ffffffff". The script starts with a lenght definition which is currently 0 and no script comes afterwards.

06 - byte length of redeem script
53 - number three (compare "The number 1 is pushed onto the stack." https://en.bitcoin.it/wiki/Script#Constants)
04 - length of locking script
57 - number seven (compare "The number 1 is pushed onto the stack." https://en.bitcoin.it/wiki/Script#Constants)
93 - OP_ADD (https://en.bitcoin.it/wiki/Script#Arithmetic)
5a - number 10 (compare "The number 1 is pushed onto the stack." https://en.bitcoin.it/wiki/Script#Constants)
87 - OP_EQUAL (https://en.bitcoin.it/wiki/Script#Bitwise_logic)

06530457935a87 - this is what will replace the "00" before the "ffffffff"s

0100000001b6c3a9c5314642ff5d1ae2fd1088997a7b8f5a063436c506f175ecfd0b5e69360100000006530457935a87ffffffff0100c44f95010000001976a91418fc1a2ce6106cef632ac755f5d15439565b6d7e88ac00000000

And broadcast that transaction to the network with sendrawtransaction:

![image](https://user-images.githubusercontent.com/54002590/174671067-b258a840-de7b-4109-8ca7-468e3d76a52b.png)

Transaction link: https://sochain.com/tx/DOGETEST/66dcfe7f9e82c1a248ca14e13d7e1a6b6183b977e82e33ae68550509503483ed

Incomming transaction:
![image](https://user-images.githubusercontent.com/54002590/174671287-afea9f97-507e-426e-b156-fd5754a2bb50.png)


## 5. Create the hash of the redeemscript manually (optional)
You won't need this but maybe you are interested "hashed" of the pay-to-script-hash part!
```shell
nformant@dogehouse:~$ redeemscript=57935a87
nformant@dogehouse:~$ echo -n $redeemscript | xxd -r -p | openssl dgst -sha256 -binary | openssl dgst -rmd160
 
(stdin)= 1fec31deece63a911dd9b22fa974ba9760d1bc3d
```
This will be added in your locking script:
"script_asm": "OP_HASH160 1fec31deece63a911dd9b22fa974ba9760d1bc3d OP_EQUAL" - link: https://sochain.com/api/v2/address/DOGETEST/2MvA1tLJeXY3hCiDjqgPTjARKRZAGadEVbj
