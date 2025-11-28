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

## テストネットでの挙動チェック

### 事前準備

```bash
aztec-up -v latest

export NODE_URL=https://aztec-testnet-fullnode.zkv.xyz
export SPONSORED_FPC_ADDRESS=0x299f255076aa461e4e94a843f0275303470a6b8ebe7cb44a471c66711151e529
```

### ウォレット作成

テストネット上にウォレット(スマートウォレット)を作成(コントラクトをデプロイ)する

```bash
aztec-wallet create-account \
    --register-only \
    --node-url $NODE_URL \
    --alias my-wallet
```

以下のようになればOK!

```bash
Address:         0x1789da5323bb81ab10ad44fc41896b227b635441e7a97d420a0f5e29b8352591
Public key:      0x05a121aa953c4584586a2eaf04044fb79792aa53638e39610ef9a087d770bf491c9f608c65b9aa13d5c6a1290b95589623358e49c6d63a7ce69f0bbc9385d49f1cf39146060834eac1e230f270de697a30cedffe7313b631e15116cf568cffe418105c417be106676000c4afdc6929931f788dfe65b5be18f84a91e06a3126901d0a4be0147f382728c63aad829b307452f462deaec2083be930c6a8fce5db1e27488600a5ff6b88850bab6b67315b68bee2196912f421a1fff8b85c489da86e0b92f99da8836c644836a3ccb126a95b3f671bfb8eec7448a3978b41eb59fea11f29c19dd71802564a3af7d4ec3c7f93673a24b8a8e574914eee9c3589d407ba
Secret key:     0x07bdf184b603f5ef0cc525b34bf2843f77c03db3633ca5e7c0a899f0db344d57
Partial address: 0x2c31c1c997fb942580d1cd4b1acdaf4f8893272b1839d53e25aff2aa9f725d1c
Salt:            0x0000000000000000000000000000000000000000000000000000000000000000
Init hash:       0x2c51487e2c7f41aabecbb3d66a3896518cc7c5d4f3050df1a88ec91990669c53
Deployer:        0x0000000000000000000000000000000000000000000000000000000000000000
```

続いて以下のコマンドを実行

```bash
aztec-wallet register-contract \
    --node-url $NODE_URL \
    --from my-wallet \
    --alias sponsoredfpc \
    $SPONSORED_FPC_ADDRESS SponsoredFPC \
    --salt 0
```

```bash
[06:37:03.526] INFO: pxe:data:lmdb Starting data store with maxReaders 16
Contract not found in the node at 0x299f255076aa461e4e94a843f0275303470a6b8ebe7cb44a471c66711151e529. Computing instance locally...
[06:37:04.588] INFO: pxe:service Started PXE connected to chain 11155111 version 845231713
Contract registered: at 0x299f255076aa461e4e94a843f0275303470a6b8ebe7cb44a471c66711151e529
Contract stored in database with aliases last & sponsoredfpc
[06:37:05.282] INFO: pxe:service Added contract SponsoredFPC at 0x299f255076aa461e4e94a843f0275303470a6b8ebe7cb44a471c66711151e529 with class 0x102dc399ad6171c78fc69f6df4db053c626691d5777e04164bd3cd9c8296de7b
```

続いて以下でデプロイ実行

```bash
aztec-wallet deploy-account \
    --node-url $NODE_URL \
    --from my-wallet \
    --payment method=fpc-sponsored,fpc=contracts:sponsoredfpc \
    --register-class
```

## コントラクトのコンパイル

```bash
yarn compile && yarn codegen
```