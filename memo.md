# 検証結果

## DevContainer環境でのチェック

### サンドボックス環境の起動

```bash
aztec start --sandbox
```

以下のようなログが出力されればOK

```bash
478fdce09bc867ad7d19212c8c0f7eba04cd52bd79461cf61a4e484b1679e
[06:12:22.546] INFO: aztecjs:deploy_sent_tx Contract 0x1466c91864211fb5534189f06a2e13dfb4d038f7e33d31f1c545d6e56710738d successfully deployed.
[06:12:22.621] WARN: pxe:service No artifact found for contract class 0x102dc399ad6171c78fc69f6df4db053c626691d5777e04164bd3cd9c8296de7b when looking for its metadata
[06:12:22.626] INFO: aztecjs:contract_interaction Creating request for publishing contract class 0x102dc399ad6171c78fc69f6df4db053c626691d5777e04164bd3cd9c8296de7b as part of deployment for 0x299f255076aa461e4e94a843f0275303470a6b8ebe7cb44a471c66711151e529
[06:12:22.739] INFO: pxe:service Added contract SponsoredFPC at 0x299f255076aa461e4e94a843f0275303470a6b8ebe7cb44a471c66711151e529 with class 0x102dc399ad6171c78fc69f6df4db053c626691d5777e04164bd3cd9c8296de7b
```

### ウォレットの作成

```bash
aztec-wallet import-test-accounts
aztec-wallet create-account -a my-wallet --payment method=fee_juice,feePayer=test0
```

```bash
Waiting for account contract deployment...
[06:14:29.156] INFO: pxe:service Sent transaction 0x0ca5a3f7fb5a351cd2b31975e6d0097726b14847b71f537d4990f633cc853180
Deploy tx hash:  0x0ca5a3f7fb5a351cd2b31975e6d0097726b14847b71f537d4990f633cc853180
Deploy tx fee:   16568880
Account stored in database with aliases last & my-wallet
```

以下のコマンドを実行してうまく作成されたかをチェックできる

```bash
aztec-wallet create-account -a my-wallet --payment method=fee_juice,feePayer=test0
```

### コントラクトのデプロイ

```bash
aztec-wallet deploy TokenContractArtifact --from accounts:test0 --args accounts:test0 TestToken TST 18 -a testtoken
```

```bash
[06:18:44.770] INFO: pxe:bb:native Generated IVC proof {"duration":56110.257829999995,"eventName":"circuit-proving"}
[06:18:44.899] INFO: pxe:service Sent transaction 0x09b4fe62ac47397956a6177e420d49bac494898b5efbe1727ef6e5f1e82bae73
Contract deployed at 0x0a4eafe87d703df4d55588c0393f3b8eef501149b56ad8756fb94f7907280959
Contract partial address 0x2ce4db0013cc537a39ca08e0bf58a11ce831222f9b946cae2702c8e7e1de0000
Contract init hash 0x2632a8c29c0a5f4f5ee0d13e29d56e8517045a4dd96e169af3e829b36bcd6006
Deployment tx hash: 0x09b4fe62ac47397956a6177e420d49bac494898b5efbe1727ef6e5f1e82bae73
Deployment salt: 0x11a5ee91f6ab83365877c3f970000b87ad8d55398d45ffa8dd1799cb8efd50e4
Deployment fee: 415142280
Contract stored in database with aliases last & testtoken
```

### Publicトークンのミント

上記でデプロイしたトークンをミントしてみる

```bash
aztec-wallet send mint_to_public --from accounts:test0 --contract-address contracts:testtoken --args accounts:test0 100
```

以下のようになればOK

```bash
Transaction hash: 0x25869eaee4e3c51b54c99472171945293f52926659a32da26aa9fe7937c8d48f
[06:20:25.273] INFO: pxe:service Sent transaction 0x25869eaee4e3c51b54c99472171945293f52926659a32da26aa9fe7937c8d48f
Transaction has been mined
 Tx fee: 59492160
 Status: success
 Block number: 10
 Block hash: 0x28fec2509c897ac52ab5b376c7bdedc222ef7d8b9986a453cb4ed96cbdd8aeda
Transaction hash stored in database with aliases last & mint_to_public-26bc
```

ミント後に残高を確認してみる

```bash
aztec-wallet simulate balance_of_public --from test0 --contract-address testtoken --args accounts:test0
```

```bash
Simulation result:  100n 
```

### PublicトークンをPrivateステートに移行

上記のPublicトークンのうち、25トークンをプライベートなステート変数として移行させる

```bash
aztec-wallet send transfer_to_private --from accounts:test0 --contract-address testtoken --args accounts:test0 25
```

以下のようになればOK

```bash
Transaction hash: 0x1562e0fbb551c4c1ef3fd3b99afafee18f9938282420172c6cc5bf865e608e4d
[06:23:35.136] INFO: pxe:service Sent transaction 0x1562e0fbb551c4c1ef3fd3b99afafee18f9938282420172c6cc5bf865e608e4d
Transaction has been mined
 Tx fee: 63202680
 Status: success
 Block number: 11
 Block hash: 0x260b2e450a7fba2e4e041979b249a41ae4e088a846f38f3ec1ec470ee69fb590
Transaction hash stored in database with aliases last & transfer_to_private-5701
```

もう一度Publicの残高を取得してみると減っている(ように見えるだけ)

```bash
Simulation result:  75n 
```

Privateステートの残高を確認してみる

```bash
aztec-wallet simulate balance_of_private --from test0 --contract-address testtoken --args accounts:test0
```

```bash
Simulation result:  25n 
```

ここまで動けば一旦サンドボックスでの検証はOK！