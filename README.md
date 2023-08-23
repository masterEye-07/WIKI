### 1. Start bootnode and get the bootnode key that will be used for parameter --bootnodes
- example:
UDP listener up, self enode://42765ed09e22140e40c94be766173c601c88e97362c14c77e815394b9d77a0fd486763ceebb51c196569fed8873b4a5ae42cb3e887216d0515112f2236fbf0ed@[::]:4440
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
./build/bin/gsmpc-client -cmd EnodeSig -url http://127.0.0.1:5871 --keystore ./test/keystore/UTC--2018-10-11T01-26-58.462416324Z--3a1b3b81ed061581558a81f11d63e03129347437 --passwdfile ./test/passwdfile/passwdfile1

./build/bin/gsmpc-client -cmd EnodeSig -url http://127.0.0.1:5872 --keystore ./test/keystore/UTC--2019-03-11T08-42-59.809814178Z--a0f15f85b7a24b66f1d682b7244242093ec4430d --passwdfile ./test/passwdfile/passwdfile2

./build/bin/gsmpc-client -cmd EnodeSig -url http://127.0.0.1:5873 --keystore ./test/keystore/UTC--2019-03-11T06-23-44.238608862Z--ecf880e334de65cd32a63b7b7567797ed707583b --passwdfile ./test/passwdfile/passwdfile3

./build/bin/gsmpc-client -cmd EnodeSig -url http://127.0.0.1:5874 --keystore ./test/keystore/UTC--2019-03-11T06-20-19.810771134Z--88525df23a7f1b3b549bcfd997ce8160ac7976a9 --passwdfile ./test/passwdfile/passwdfile4

./build/bin/gsmpc-client -cmd EnodeSig -url http://127.0.0.1:5875 --keystore ./test/keystore/UTC--2018-10-12T11-33-28.769681948Z--0963a18ea497b7724340fdfe4ff6e060d3f9e388 --passwdfile ./test/passwdfile/passwdfile5
```

### 6. Request for generating public key
- gid is the group id containing 5 nodes.
- keytype: EC256K1 or ED25519 or EC256STARK.
- sig is the sign information of node.
- mode: 0 co-management mode or 1 non-co-management mode or 2 random signature subgroup mode 
```
./build/bin/gsmpc-client -cmd REQSMPCADDR --keystore ./test/keystore/UTC--2018-10-11T01-26-58.462416324Z--3a1b3b81ed061581558a81f11d63e03129347437 --passwdfile ./test/passwdfile/passwdfile1 -ts 3/5 -gid b0991607cc62ab84404ec2dfddf15f85452c5a8c190302e62f473570a57bdb9ba0609f84e60eb5b13046080bcb45f0003743f2f2807efd1bc53073649b1d79d3 -mode 0 -msgsig=true -url http://127.0.0.1:5871 --keytype EC256K1 -sig enode://a4bf71e29738fc406b7d5d8388a553e22baa829dd015f7fdfb9780a40c54db0381eba6857467d32bb3b30cb3609217b50a80664946538711b87c66f77ef90d8c@127.0.0.1:485410x860376ed6b8349caee05bbedd3488b2a088c9915723b3976f9421c5ba8cd8b610eec375b8b9525caa512b174fbc6becbb0f98e1420d45f8b651712b461c8d39e01 -sig enode://85c45e2a409233ae37111b753f9075930eb5485e01681949a55ad84c900bb64506fa31493fa6081698c28179c9c444febf287d9433a6552a7daed96eb8197343@127.0.0.1:485420xe55cd49480b67cfd24488c680052b4526a18e9e655243d478ef40a1b0387d94747432ddef86d09d43f63d9774a91de2c6791992f24aca6fc517f8060bbf3285301 -sig enode://ed0f97089b78f1dcb2aa2931f6b7c38f23893a302b5bcb1f8a213983faffe20b1a6bd2f61ba330f5c2ffb3165078a3616be765de456a7e5f1a6171dd66cc3e67@127.0.0.1:485430x8fad9149a2041a6f31d38874488860d5d01e4ee03f502f2673174461817448e43599e4f38ed0e183cd7e4e2e837b42ff5e7197f74037902f0eb9337535861a3000 -sig enode://627cd370a68496170df42bb1e09c25b0ce4b91fef3957b6cf4b45ff67dfa6b4de0666292d36be67b4dae75a86dfff542110525e12a01e7db4994ddb3be963eab@127.0.0.1:485440x46c3de20c67fb1b69a1c456612c94d38bac1f222c1747cdfc57e07431454757b03c7b82fbd0234a5699979b9f1426ab31024f094f36fdd181fd8cf31e4df36c400 -sig enode://779142644b556cfe0ec5be097259df6ab59a8d88b15720e9dfd5a03cccd210e59614c91db0013250179a1377430480d80730d1000b7f172ca6c9d1f7765d1609@127.0.0.1:485450x9a3c8c403a008bd9ff4878c8b6416243e22cd2d2e0bc854849218293c59d313b5edbbe34004049d0a8b04790c6a7fc468b35aedc92b61c5624d44361dfe152d901
```

### 7. Accept the request of generating public key
- key is the unique ID of this request command and is obtained in step 6.
```
./build/bin/gsmpc-client -cmd ACCEPTREQADDR  -url http://127.0.0.1:5871 --keystore ./test/keystore/UTC--2018-10-11T01-26-58.462416324Z--3a1b3b81ed061581558a81f11d63e03129347437 --passwdfile ./test/passwdfile/passwdfile1 -key 0x3628959e323c100b4f0c098f33bcbd5c381b72489565561b22605e1b08531074 --keytype EC256K1 -msgsig=true -mode 0


./build/bin/gsmpc-client -cmd ACCEPTREQADDR  -url http://127.0.0.1:5872 --keystore ./test/keystore/UTC--2019-03-11T08-42-59.809814178Z--a0f15f85b7a24b66f1d682b7244242093ec4430d --passwdfile ./test/passwdfile/passwdfile2 -key 0x3628959e323c100b4f0c098f33bcbd5c381b72489565561b22605e1b08531074 --keytype EC256K1 -msgsig=true -mode 0


./build/bin/gsmpc-client -cmd ACCEPTREQADDR  -url http://127.0.0.1:5873 --keystore ./test/keystore/UTC--2019-03-11T06-23-44.238608862Z--ecf880e334de65cd32a63b7b7567797ed707583b --passwdfile ./test/passwdfile/passwdfile3 -key 0x3628959e323c100b4f0c098f33bcbd5c381b72489565561b22605e1b08531074 --keytype EC256K1 -msgsig=true -mode 0


./build/bin/gsmpc-client -cmd ACCEPTREQADDR  -url http://127.0.0.1:5874 --keystore ./test/keystore/UTC--2019-03-11T06-20-19.810771134Z--88525df23a7f1b3b549bcfd997ce8160ac7976a9 --passwdfile ./test/passwdfile/passwdfile4 -key 0x3628959e323c100b4f0c098f33bcbd5c381b72489565561b22605e1b08531074 --keytype EC256K1 -msgsig=true -mode 0


./build/bin/gsmpc-client -cmd ACCEPTREQADDR  -url http://127.0.0.1:5875 --keystore ./test/keystore/UTC--2018-10-12T11-33-28.769681948Z--0963a18ea497b7724340fdfe4ff6e060d3f9e388 --passwdfile ./test/passwdfile/passwdfile5 -key 0x3628959e323c100b4f0c098f33bcbd5c381b72489565561b22605e1b08531074 --keytype EC256K1 -msgsig=true -mode 0
```

### 8. Pre-generated sign data
- mode = 2 does not need to be pre generated or automatically pre generated. Other mode values need to do this.
- if ed, skip this step.
- select node 1,2,3 as example.
-  pubkey is the public key obtained in step 7.
```
./build/bin/gsmpc-client -cmd PRESIGNDATA --keystore ./test/keystore/UTC--2018-10-11T01-26-58.462416324Z--3a1b3b81ed061581558a81f11d63e03129347437 --passwdfile ./test/passwdfile/passwdfile1 -pubkey 0463d86d7400ff5d8ed228be11200e1e7dff7c52e345768dd0a95b4441bd56615b17a621e8ac2d1cbadee08423f7e50d93569c98c9175982114ec6ac91759af6ae -subgid 13fb8a42c696c78e15d874d58bde864ef861d7b30a5a6cab4bd4174a20768a87f5b345c2c86f646224a663dfa6ca6127ce911481ccf37ee7765afd8e335c832c -url http://127.0.0.1:5871 --keytype EC256K1 -mode 0 -msgsig=true
```

### 9. Sign
- gid is the subgroup id, and pubkey is the public key obtained in step 7.
- msghash is the hash to be signed, can be plural.
- msgcontext is the information of the cross-chain bridge, you can just ignore this default.
- keytype: EC256K1 or ED25519 or EC256STARK.
- select node 1,2,3 as example.
```
./build/bin/gsmpc-client -cmd SIGN -ts 3/5 -n 1 --loop 1 --keystore ./test/keystore/UTC--2018-10-11T01-26-58.462416324Z--3a1b3b81ed061581558a81f11d63e03129347437 --passwdfile ./test/passwdfile/passwdfile1 -gid 13fb8a42c696c78e15d874d58bde864ef861d7b30a5a6cab4bd4174a20768a87f5b345c2c86f646224a663dfa6ca6127ce911481ccf37ee7765afd8e335c832c -mode 0 -msgsig=true -url http://127.0.0.1:5871 --keytype EC256K1 -pubkey 0463d86d7400ff5d8ed228be11200e1e7dff7c52e345768dd0a95b4441bd56615b17a621e8ac2d1cbadee08423f7e50d93569c98c9175982114ec6ac91759af6ae -msghash 0x90e032be062dd0dc689fa23df8c044936a2478cb602b292c7397354238a67d88  -msgcontext '{"swapInfo":{"swapid":"0x4f62545cdd05cc346c75bb42f685a18a02621e91512e0806eac528d0b2f6aa5f","swaptype":1,"bind":"0x0520e8e5e08169c4dbc1580dc9bf56638532773a","identifier":"ssUSDT2FSN"},"extra":{"ethExtra":{"gas":90000,"gasPrice":1000000000,"nonce":1}}}'
```

### 10. Let the nodes in the subgroup agree to sign
- key is the unique ID of this sign request command and is obtained in step 9.
```
./build/bin/gsmpc-client -cmd ACCEPTSIGN  -url http://127.0.0.1:5871 --keystore ./test/keystore/UTC--2018-10-11T01-26-58.462416324Z--3a1b3b81ed061581558a81f11d63e03129347437 --passwdfile ./test/passwdfile/passwdfile1 -key 0x32f14e52cac59c85077d238b4d06ab755fcbb3037e79b5d76a6f76950d1b300b --keytype EC256K1 -msgsig=true -mode 0

./build/bin/gsmpc-client -cmd ACCEPTSIGN -url http://127.0.0.1:5872 --keystore ./test/keystore/UTC--2019-03-11T08-42-59.809814178Z--a0f15f85b7a24b66f1d682b7244242093ec4430d --passwdfile ./test/passwdfile/passwdfile2 -key 0x32f14e52cac59c85077d238b4d06ab755fcbb3037e79b5d76a6f76950d1b300b --keytype EC256K1 -msgsig=true -mode 0

./build/bin/gsmpc-client -cmd ACCEPTSIGN -url http://127.0.0.1:5873 --keystore ./test/keystore/UTC--2019-03-11T06-23-44.238608862Z--ecf880e334de65cd32a63b7b7567797ed707583b --passwdfile ./test/passwdfile/passwdfile3 -key 0x32f14e52cac59c85077d238b4d06ab755fcbb3037e79b5d76a6f76950d1b300b --keytype EC256K1 -msgsig=true -mode 0
```

### 11. Get the co-managed public key by the key of generating pubkey key request
```
curl -X POST -H "Content-Type":application/json --data '{"jsonrpc":"2.0","method":"smpc_getReqAddrStatus","params":["0x99eb5c871ce7dc1a2dfd22339c9266894b3fb7aebe2e7a97a6db16b7f2b0e095"],"id":67}' http://127.0.0.1:5871
```

### 12. Get the sign result (rsv) by the key of signing request
```
curl -X POST -H "Content-Type":application/json --data '{"jsonrpc":"2.0","method":"smpc_getSignStatus","params":["0x32f14e52cac59c85077d238b4d06ab755fcbb3037e79b5d76a6f76950d1b300b"],"id":67}' http://127.0.0.1:5871
```
