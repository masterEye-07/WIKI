### 1. Start bootnode and get the bootnode key that will be used for parameter --bootnodes
```
./build/bin/bootnode --genkey=boot.key
./build/bin/bootnode --nodekey=boot.key
```

### 2. Start 5 mpc nodes
```
./build/bin/gsmpc --rpcport 5871 --bootnodes "enode://d13ff378464867d4ebef8774c55fb053dfafc97141ad04ad9710d96afff619ba9751c17444b416805cda8eb70f7e3bfd1f1eb1b65c7799a74b38e141c066ad65@127.0.0.1:4440" --port 48541 --nodekey "node1.key" --verbosity 5 --nonetrestrict=false 2>&1 | tee node1.log

./build/bin/gsmpc --rpcport 5872 --bootnodes "enode://d13ff378464867d4ebef8774c55fb053dfafc97141ad04ad9710d96afff619ba9751c17444b416805cda8eb70f7e3bfd1f1eb1b65c7799a74b38e141c066ad65@127.0.0.1:4440" --port 48542 --nodekey "node2.key" --verbosity 5 --nonetrestrict=false 2>&1 | tee node2.log

./build/bin/gsmpc --rpcport 5873 --bootnodes "enode://d13ff378464867d4ebef8774c55fb053dfafc97141ad04ad9710d96afff619ba9751c17444b416805cda8eb70f7e3bfd1f1eb1b65c7799a74b38e141c066ad65@127.0.0.1:4440" --port 48543 --nodekey "node3.key" --verbosity 5 --nonetrestrict=false 2>&1 | tee node3.log

./build/bin/gsmpc --rpcport 5874 --bootnodes "enode://d13ff378464867d4ebef8774c55fb053dfafc97141ad04ad9710d96afff619ba9751c17444b416805cda8eb70f7e3bfd1f1eb1b65c7799a74b38e141c066ad65@127.0.0.1:4440" --port 48544 --nodekey "node4.key" --verbosity 5 --nonetrestrict=false 2>&1 | tee node4.log

./build/bin/gsmpc --rpcport 5875 --bootnodes "enode://d13ff378464867d4ebef8774c55fb053dfafc97141ad04ad9710d96afff619ba9751c17444b416805cda8eb70f7e3bfd1f1eb1b65c7799a74b38e141c066ad65@127.0.0.1:4440" --port 48545 --nodekey "node5.key" --verbosity 5 --nonetrestrict=false 2>&1 | tee node5.log
```


### 3. Create group id containing 5 nodes
- example: 9fffdd35e5570f1a14d9b091a4d81c8fe9750536f6b8e5b9ab9591d8d75f4248b14b49f4a2be2b285a7f5155085b92d5fbdedfcf877492d038039555d93bbd8f
```
./build/bin/gsmpc-client -cmd SetGroup -url http://127.0.0.1:5871 -ts 3/5 -node http://127.0.0.1:5871 -node http://127.0.0.1:5872 -node http://127.0.0.1:5873 -node http://127.0.0.1:5874 -node http://127.0.0.1:5875
```

### 4. Create a subgroup id (select node 1, 2, 3 as example)
- example: 905025506f0217eb8f177fdf27dc46546b8593890f3d238e6ee09a49fb696cef5c7b6d4f71661b05b3655099e3faf6398aa1ac98913527725d4a4eaaa118ce53
```
./build/bin/gsmpc-client -cmd SetGroup -url http://127.0.0.1:5871 -ts 3/5 -node http://127.0.0.1:5871 -node http://127.0.0.1:5872 -node http://127.0.0.1:5873
```

### 5. Get the sign information from each node
- sign infomation is: EnodeID@IP:PORT + hex.EncodeToString(crypto.Sign(crypto.Keccak256(EnodeID), privateKey))
```
./build/bin/gsmpc-client -cmd EnodeSig -url http://127.0.0.1:5871 --keystore
~/Downloads/UTC--2018-10-11T01-26-58.462416324Z--3a1b3b81ed061581558a81f11d63e03129347437 --passwdfile ~/Downloads/passwdfile1

./build/bin/gsmpc-client -cmd EnodeSig -url http://127.0.0.1:5872 --keystore
~/Downloads/UTC--2019-03-11T08-42-59.809814178Z--a0f15f85b7a24b66f1d682b7244242093ec4430d --passwdfile ~/Downloads/passwdfile2

./build/bin/gsmpc-client -cmd EnodeSig -url http://127.0.0.1:5873 --keystore
~/Downloads/UTC--2019-03-11T06-23-44.238608862Z--ecf880e334de65cd32a63b7b7567797ed707583b --passwdfile ~/Downloads/passwdfile3

./build/bin/gsmpc-client -cmd EnodeSig -url http://127.0.0.1:5874 --keystore
~/Downloads/UTC--2019-03-11T06-20-19.810771134Z--88525df23a7f1b3b549bcfd997ce8160ac7976a9 --passwdfile ~/Downloads/passwdfile4

./build/bin/gsmpc-client -cmd EnodeSig -url http://127.0.0.1:5875 --keystore
~/Downloads/UTC--2018-10-12T11-33-28.769681948Z--0963a18ea497b7724340fdfe4ff6e060d3f9e388 --passwdfile ~/Downloads/passwdfile5
```

### 6. Request for generating public key
- gid is the group id containing 5 nodes.
- keytype: EC256K1 or ED25519 or EC256STARK.
- sig is the sign information of node.
- mode: 0 co-management mode or 1 non-co-management mode or 2 random signature subgroup mode 
```
./build/bin/gsmpc-client -cmd REQSMPCADDR --keystore ~/Downloads/UTC--2018-10-11T01-26-58.462416324Z--3a1b3b81ed061581558a81f11d63e03129347437 --passwdfile ~/Downloads/passwdfile1 -ts 3/5 -gid 9fffdd35e5570f1a14d9b091a4d81c8fe9750536f6b8e5b9ab9591d8d75f4248b14b49f4a2be2b285a7f5155085b92d5fbdedfcf877492d038039555d93bbd8f -mode 0 -msgsig=true -url http://127.0.0.1:5871 --keytype EC256K1 -sig enode://524da89d8bd8f051e9b24660941e772f60e2e7a4ab0c48d1671ecfb9844e9cf1f0a9ef697be8118fe7bc9f2782a5a2bad912856bcb3a4dfda0aafcbdc9c282af@127.0.0.1:485410xe959a7de8ad34d8662c993d850de8fc7fcf48a201f95c04861baac72f5e0d44c384f7bf58f011f9525c1ba216e28dac02f98c1c797a0b422a6eb33f54bd9e33001 -sig enode://8908863d56914eaa420afca83f43206f65ff56e42faeecb9e24f77740838819313f789bf7c4941b162e696147df409e47eb95ea351eac016ce8b7bf38fd269b2@127.0.0.1:485420xca2ffe89d3a143be76f86b22e255720c98f56ecf313eab6e9e26b12b9d3ca1b04131e21d3490dafcec6c3a0614173e161939bf0dd682e47201ceb103e3024d9000 -sig enode://ee10b450a564e9cda30b37b9497a11ede5583e8964ba01ae85bbba7b421e403b1ab710f87d5b93c700ca459661b889412b9d781fbef8094ee282b46f4a90508b@127.0.0.1:485430xbcef37a181ccde32172539d2cda69811dacb6929748a269b8116a01b865beb356ff49ca53c04897b88d4a42caa0d3dc0f2b5d07726ca7478478aaaeecac3d76a00 -sig enode://785708c037dcb3527075fe85797e5997c33202508daaa8ee45e99f73121047f3bb3dd2662ce5f0bb9b8e83cd2e56b3d99d2156433fef5185f66d2bb21d944e25@127.0.0.1:485440xaf7c6bbc6fe7b66bce2aeb436eef77f8c4d0ed882a3b3606abf960b9927955977564e71b04fd60828005c623173ec5b35855c48ebfd10f4b587dba7f3a1fd92f00 -sig enode://730c8fc7142d15669e8329138953d9484fd4cce0c690e35e105a9714deb741f10b52be1c5d49eeeb6f00aab8f3d2dec4e3352d0bf56bdbc2d86cb5f89c8e90d0@127.0.0.1:485450x74a089ea6ab94e978d87d4d791ff2b9cc3c370785124ac3548b8b5ad2a714f5308b6f7f35bf12021d1679911057e1f2ed39ff12653356af280e3cd1517b878ad00
```

### 7. Accept the request of generating public key
- key is the unique ID of this request command and is obtained in step 6.
```
./build/bin/gsmpc-client -cmd ACCEPTREQADDR  -url http://127.0.0.1:5871 --keystore ~/Downloads/UTC--2018-10-11T01-26-58.462416324Z--3a1b3b81ed061581558a81f11d63e03129347437 --passwdfile ~/Downloads/passwdfile1 -key 0x99eb5c871ce7dc1a2dfd22339c9266894b3fb7aebe2e7a97a6db16b7f2b0e095 --keytype EC256K1 -msgsig=true -mode 0


./build/bin/gsmpc-client -cmd ACCEPTREQADDR  -url http://127.0.0.1:5872 --keystore ~/Downloads/UTC--2019-03-11T08-42-59.809814178Z--a0f15f85b7a24b66f1d682b7244242093ec4430d --passwdfile ~/Downloads/passwdfile2 -key 0x99eb5c871ce7dc1a2dfd22339c9266894b3fb7aebe2e7a97a6db16b7f2b0e095 --keytype EC256K1 -msgsig=true -mode 0


./build/bin/gsmpc-client -cmd ACCEPTREQADDR  -url http://127.0.0.1:5873 --keystore ~/Downloads/UTC--2019-03-11T06-23-44.238608862Z--ecf880e334de65cd32a63b7b7567797ed707583b --passwdfile ~/Downloads/passwdfile3 -key 0x99eb5c871ce7dc1a2dfd22339c9266894b3fb7aebe2e7a97a6db16b7f2b0e095 --keytype EC256K1 -msgsig=true -mode 0


./build/bin/gsmpc-client -cmd ACCEPTREQADDR  -url http://127.0.0.1:5874 --keystore ~/Downloads/UTC--2019-03-11T06-20-19.810771134Z--88525df23a7f1b3b549bcfd997ce8160ac7976a9 --passwdfile ~/Downloads/passwdfile4 -key 0x99eb5c871ce7dc1a2dfd22339c9266894b3fb7aebe2e7a97a6db16b7f2b0e095 --keytype EC256K1 -msgsig=true -mode 0


./build/bin/gsmpc-client -cmd ACCEPTREQADDR  -url http://127.0.0.1:5875 --keystore ~/Downloads/UTC--2018-10-12T11-33-28.769681948Z--0963a18ea497b7724340fdfe4ff6e060d3f9e388 --passwdfile ~/Downloads/passwdfile5 -key 0x99eb5c871ce7dc1a2dfd22339c9266894b3fb7aebe2e7a97a6db16b7f2b0e095 --keytype EC256K1 -msgsig=true -mode 0
```

### 8. Pre-generated sign data
- mode = 2 does not need to be pre generated or automatically pre generated. Other mode values need to do this.
- if ed, skip this step.
- select node 1,2,3 as example.
-  pubkey is the public key obtained in step 7.
```
./build/bin/gsmpc-client -cmd PRESIGNDATA --keystore /home/go/Downloads/UTC--2018-10-11T01-26-58.462416324Z--3a1b3b81ed061581558a81f11d63e03129347437 --passwdfile ~/Downloads/passwdfile1 -pubkey  04d94dc405077a34b865d9e4e4cd004ee4e20d5e758dd770a49feafef9c3dadb3a98f483e3ab1085629a2d5677ec82e3503d6f5e501a79169c098085672ec5b403  -subgid 905025506f0217eb8f177fdf27dc46546b8593890f3d238e6ee09a49fb696cef5c7b6d4f71661b05b3655099e3faf6398aa1ac98913527725d4a4eaaa118ce53  -url  http://127.0.0.1:5871 --keytype EC256K1 -mode 0 -msgsig=true
```

### 9. Sign
- gid is the subgroup id, and pubkey is the public key obtained in step 7.
- msghash is the hash to be signed, can be plural.
- msgcontext is the information of the cross-chain bridge, you can just ignore this default.
- keytype: EC256K1 or ED25519 or EC256STARK.
- select node 1,2,3 as example.
```
./build/bin/gsmpc-client -cmd SIGN -ts 3/5 -n 1 --loop 1 --keystore ~/Downloads/UTC--2018-10-11T01-26-58.462416324Z--3a1b3b81ed061581558a81f11d63e03129347437 --passwdfile ~/Downloads/passwdfile1 -gid 905025506f0217eb8f177fdf27dc46546b8593890f3d238e6ee09a49fb696cef5c7b6d4f71661b05b3655099e3faf6398aa1ac98913527725d4a4eaaa118ce53 -mode 0 -msgsig=true -url http://127.0.0.1:5871 --keytype EC256K1 -pubkey 04d94dc405077a34b865d9e4e4cd004ee4e20d5e758dd770a49feafef9c3dadb3a98f483e3ab1085629a2d5677ec82e3503d6f5e501a79169c098085672ec5b403 -msghash 0x90e032be062dd0dc689fa23df8c044936a2478cb602b292c7397354238a67d88  -msgcontext '{"swapInfo":{"swapid":"0x4f62545cdd05cc346c75bb42f685a18a02621e91512e0806eac528d0b2f6aa5f","swaptype":1,"bind":"0x0520e8e5e08169c4dbc1580dc9bf56638532773a","identifier":"ssUSDT2FSN"},"extra":{"ethExtra":{"gas":90000,"gasPrice":1000000000,"nonce":1}}}'
```

### 10. Let the nodes in the subgroup agree to sign
- key is the unique ID of this sign request command and is obtained in step 9.
```
./build/bin/gsmpc-client -cmd ACCEPTSIGN  -url http://127.0.0.1:5871 --keystore ~/Downloads/UTC--2018-10-11T01-26-58.462416324Z--3a1b3b81ed061581558a81f11d63e03129347437 --passwdfile ~/Downloads/passwdfile1 -key 0xa60365401eab5069154730cd80f00c568ab6295a0537ac1baf855c0291523c9a --keytype EC256K1 -msgsig=true -mode 0

./build/bin/gsmpc-client  -cmd ACCEPTSIGN -url http://127.0.0.1:5872 --keystore ~/Downloads/UTC--2019-03-11T08-42-59.809814178Z--a0f15f85b7a24b66f1d682b7244242093ec4430d --passwdfile ~/Downloads/passwdfile2 -key 0xa60365401eab5069154730cd80f00c568ab6295a0537ac1baf855c0291523c9a --keytype EC256K1 -msgsig=true -mode 0

./build/bin/gsmpc-client  -cmd ACCEPTSIGN -url http://127.0.0.1:5873 --keystore ~/Downloads/UTC--2019-03-11T06-23-44.238608862Z--ecf880e334de65cd32a63b7b7567797ed707583b --passwdfile ~/Downloads/passwdfile3 -key 0xa60365401eab5069154730cd80f00c568ab6295a0537ac1baf855c0291523c9a --keytype EC256K1 -msgsig=true -mode 0
```

### 11. Get the co-managed public key by the key of generating pubkey key request
```
curl -X POST -H "Content-Type":application/json --data '{"jsonrpc":"2.0","method":"smpc_getReqAddrStatus","params":["0x99eb5c871ce7dc1a2dfd22339c9266894b3fb7aebe2e7a97a6db16b7f2b0e095"],"id":67}' http://127.0.0.1:5871
```

### 12. Get the sign result (rsv) by the key of signing request
```
curl -X POST -H "Content-Type":application/json --data '{"jsonrpc":"2.0","method":"smpc_getSignStatus","params":["0xa60365401eab5069154730cd80f00c568ab6295a0537ac1baf855c0291523c9a"],"id":67}' http://127.0.0.1:5871
```
