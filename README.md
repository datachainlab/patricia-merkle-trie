# Merkle Patricia Trie

This library allows users to verify ethereum style merkle patricia proofs as specified in this document: https://ethereum.github.io/execution-specs/autoapi/ethereum/frontier/trie/index.html

```rust
use patricia_merkle_trie::{
    keccak::{keccak_256, KeccakHasher},
    EIP1186Layout, StorageProof,
};
use hex_literal::hex;
use primitive_types::{H256, U256};
use rlp::{Decodable, Rlp};
use rlp_derive::RlpDecodable;
use trie_db::{Trie, TrieDBBuilder};

/// The ethereum account stored in the global state trie.
#[derive(RlpDecodable, Debug)]
struct Account {
    nonce: u64,
    balance: U256,
    storage_root: H256,
    code_hash: H256,
}

fn main() {
    // source: https://medium.com/@chiqing/eip-1186-explained-the-standard-for-getting-account-proof-444bc1e12e03
    let key = keccak_256(&hex!("b856af30b938b6f52e5bff365675f358cd52f91b"));
    let proof = vec![
        hex!("f90211a021162657aa1e0af5eef47130ffc3362cb675ebbccfc99ee38ef9196144623507a073dec98f4943e2ab00f5751c23db67b65009bb3cb178d33f5aa93f0c08d583dda0d85b4e33773aaab742db880f8b64ea71f348c6eccb0a56854571bbd3db267f24a0bdcca489de03a49f109c1a2e7d3bd4e644d72de38b7b26dca2f8d3f112110c6fa05c7e8fdff6de07c4cb9ca6bea487a6e5be04af538c25480ce30761901b17e4bfa0d9891f4870e745509cfe17a31568f870b367a36329c892f1b2a37bf59e547183a0af08f747d2ea66efa5bcd03729a95f56297ef9b1e8533ac0d3c7546ebefd2418a0a107595919d4b102afaa0d9b91d9f554f83f0ad61a1e04487e5091543eb81db8a0a0725da6da3b62f88fc573a3fd0dd9dea9cba1750786021da836fd95b7295636a0fd7a768700af3caadaf52a08a23ab0b71ca52830f2b88b1a6b23a52f9ee05507a059434ae837706d7d317e4f7d03cd91f94ed0465fa8b99eaf18ca363bb318c7b3a09e9b831a5f59b781efd5dae8bea30bfd81b9fd5ea231d6b7e82da495c95dd35da0e72d02a01ed9bc928d94cad59ae0695f45120b7fbdbce43a2239a7e5bc81f731a0184bfb9a4051cbaa79917183d004c8d574d7ed5becaf9614c650ed40e8d123d9a0fa4797dc4a35af07f1cd6955318e3ff59578d4df32fd2174ed35f6c4db3471f9a0fec098d1fee8e975b5e78e19003699cf7cd746f47d03692d8e11c5fd58ba92a680").to_vec(),
        hex!("f90211a07fc5351578eb6ab7618a31e18c87b2b8b2703c682f2d4c1d01aaa8b53343036ea0e8871ae1828c54b9c9bbf7530890a2fe4e160fb62f72c740c7e79a756e07dbf3a04dd116a7d37146cd0ec730172fa97e84b1f13e687d56118e2d666a02a31a629fa08949d66b81ba98e5ca453ba1faf95c8476873d4c32ff6c9a2558b772c51c5768a028db2de6d80f3a06861d3acc082e3a6bb4a6948980a8e5527bd354a2da037779a09b01ba0fe0193c511161448c602bb9fff88b87ab0ded3255606a15f8bca9d348a0c1c1c6a89f2fdbee0840ff309b5cecd9764b5b5815b385576e75e235d1f04656a04e827215bb9511b3a288e33bb418132940a4d42d589b8db0f796ec917e8f9373a099398993d1d6fdd15d6082be370e4d2cc5d9870923d22770aaec9418f4b675d7a00cd1db5e131341b472af1bdf9a1bf1f1ca82bc5b280c8a50a20bcfff1ab0bdd4a09bbcc86c94be1aabf5c5ceced29f462f59103aa6dafe0fc60172bb2c549a8dbaa0902df0ba9eed7e8a6ebff2d06de8bcec5785bb98cba7606f7f40648408157ef4a0ba9dfd07c453e54504d41b7a44ea42e8220767d1e2a0e6e91ae8d5677ac70e50a0f02f2a5e26d7848f0e5a07de68cbbbd24253d545afb74aac81b35a70b6323f1ca0218b955deca7177f8f58c2da188611b333e5c7ef9212000f64ca92cd5bb6e5a0a049cd750f59e2d6f411d7b611b21b17c8eefe637ca01e1566c53f412308b34c6280").to_vec(),
        hex!("f90211a05303302919681c3ad0a56c607c9530ed910f44515f6b40c9633d1900bbbc7e0fa0459fc49e57f39ca6471b1c6905ede7eaa6d7186c8249485cc28338ba18c540cba0825307726d1b7c9d74973d37c12e8b035bf0334836e48ec3f2ff18bf2232dabea0a67ef68daba820c7d6343d1b147b73430ce5c5915a27581cfd12946c2307dc49a003c9b0f0b784de7d72f3b5d5fea87e30dc5fc6f93a0c218240f79a2c74b0f8e2a05a38ddf70df168305b8ba38e8ee54dfadc3f7d81335ec849cb143a10d9738a91a058f0692b5cb07a1c8c800fcf8a70c6e6189a5d72f24ca0040423cf450df1da44a0890dbc62e7429fcca3f1507ad2cd4799c0a7aab25db37ccad434ae23ae889807a075be60d2f635292e69dbc600600605cb8eaf75e96425fd3f2347a5c644db43b9a07b65ba06ee9d2b5dab0a9acc1b8b521cb42f91566de9c174636e817c3d990265a0de65bc6092e28b0cc1ed454fcc07ce26df21bb05efe0a4b4472ff63278e28b95a08077cd7de83428d376ff7588b32df903d2764af7d41deb9c873e9ae889823cd3a0af2f63837dc01e2efb9e40815761015a0d740c2d2549593eefd291a05d40b55aa0c3214baa8d347bd5703926b6fe3ee2f607d0debc0fd73352696dc10f4cbc517da01756cf85b4785bda4a9867a453f8ca1948f015bd329b84083f14d313bddafb80a00dac89194bc1f28d3971b9ca9d1e16a49c6383557187d7bbeb899626d60bfb1980").to_vec(),
        hex!("f90211a0bf50b49fae6cfe8b7671e3fa0c163aed76f6457720a2b7c18f567b3c02194c29a070dc71bb7e399e5ae66958261108c84b75e8aacc8d255ce28cd7c9029358872ba0d2ae86d376e65eee52338ad4a1951deb9312f2c161fdf5cfc3e36d5a07ee4239a0f2029dea5033d0e788191ba25fc25bb0570bdbbaf321dfdb076f6695c649a07ca066074b59980560ecdd8ebc96eeb93f50dc1e92983659ea4a6a61a4cff0f474cba01ad85159ddc98609ea628cd17897fe08b0d9a7bb07a2087d92a673e063039aaba04921580f8766f8f156546abd8f0e44af250b34e7323f35c40fdc078223822344a034e07b24a1c17f5dcff27b766099c206fbbb6e549d3f4c02fd8db0241061482aa0c852267182c35e2e5014ab6d656672e9446aaf79c6248d103870d55ee36368b1a00aed203f7e2684942a64f05306e57d64fd44eded94e2ce95e462be93adff640da02cea88d74264c91c546de3822b6169a427559781a774511409864d70a834706ea01d542f8a9b69674e58a5bb89fafb5e79cbe3607732455b09c2a996df48e48837a04c3bbb4f47041018455347567a4e3af472fafe179871f667c3d26038f5dabacfa03c4f12f7cdd35126ce5452aa8322bc8b497eb06c5c41741b590d40645a8fd14ea01334e9a4160b44b622e9523cb587d8ba4795bbf9ad3dd0aa1a2b7f5c6a5cbe94a08feae3d50602063d65763185633aa6e23bb47eca9a39982a4863a7cc6d3586ff80").to_vec(),
        hex!("f90211a03fc22103871f30d114942583d24adfab1ce2e651ff6705a05964318fde7c425aa01c86fd2e9d2a823db33bf4089ce1af41332b4e3069b31bb70a67861944d71688a08a90ae88b4479d21135517195f50df20fc29dbe495e09440b0fa5797fc0352c8a02a195c4a89ab6322d8daa221124274d711a9435587406addfb289f9360c0b1caa02f7ed0113a1b72febc7ceb7d9193baeaa093aebea76eea4729821f53d29b302ea07d5cbbeffd22fb0f9d510576cac47a604b2121ef8b08588eaefc46a07844515ca01c0c09d203e342fab9f80835f3aa7bb7e94cbf94d3a18b21ce905e75b690673fa07c310f931f12d1651dbb9bdffbe5e0a16db981dccfb3f4a838592e2347b1b187a091b24e00d37034ed70e0c6653f8616363efbea43be86d52aabf6a9c5d5049d3aa017cc8ab2e63508691dee64ddb2dbe5352fe531f55728368cab3d8634450730dca0d8ee92d688eabcdf28af1830d7217fcd1431c0b0e3311039c422c5f45f9d525da0abe4323ef90fb783ffd6bf29d240818b0a2477d2ad4577fce57642a5ba476957a0eee3fbc510b1a6d8da176b9eab2035837769988b216fcef67f6a215e5e261d5ea082bc27591d8b0408739712c2f0e623e3d296d12afd6b7356dae237a315a6ce3ba0634affb8f9744fed774851772cb5ce495c50212961a64262d915632c2bede721a0345017d846f3be29dca1f56a734886c26cb49d3bcbbae5f7356e55e71d84147d80").to_vec(),
        hex!("f90211a0a152d08043b3865248d8ae9c4594d6e09079f61ee4ad9afca98d3befb3a9307ca01313ea4df7ec991f5ba1b2175408765ea67111857ae25cafe21986810d633353a05cb00f30a4b6749cb8d01dda2ad665a2c570b7c6959b364c46da75fc2aebfa14a0a493d42fa40d6c8fb23090f20cebe9f1cde8a47d9594e666096c3f76219cfb34a06d2149e05ba1a31bcc352fd3a79fd42d95383f14a312862fa5f4b7b7bdb63254a0f40094679bf0599fdeecae8c800a423ad2499b67f9546d4085d5b7a351561072a05036fc625ed5a13d143fd0984e99969d2d48f962baec80b3f0e78323c8e864ffa0c0887db54ab0d4309ed5f563448bfc78a4a88c3a5473fcbd9bd263c8fcca4b9fa04287b193a315cba13a49482b4d83c068cf08b622593111e416b7a3b815e3595aa051b82224b54dc4050703157cdd6c74c618872c798badce7192fe1fd534814d5da017ac02273956b25dc2429750156e0a45bf461437bec84b784d19a5b964dd6882a05be9c25f80a6f34e9eb526d6c3c89e3ec2c5dd769d2d915d835208cac3f56d36a0ea7ae8e74baeae9d6307371f85cce46ba7ad46b5c2b3616a3573a26e1260bc31a04446678b6bf75075ae0261c179d88e49fae1d9482c214ec8c693239b583a6b18a09ec91d47f671c343cea224def0afcd62a57a408ac0e36b79cf29a4495ff9055ca0c3da71d14030daf8fb9157ec84f077df97dff395c1fcf8f04361e02aa1af36ff80").to_vec(),
        hex!("f8b1808080a038f5e1b2680d95eaf7f225b996fc482f60cabcebeae26f883a4b58e1e9c7bbeda0c220c5d76d85ad38d8f82d0d6d6f48db3c23ae3657d1ac3ca6e2d98b4e48bfde8080a06fe32c7c1f8f80ebe5128f11a7af3a5bc47dbc6ac0af705069532b1cbefd6792a0015eb7d24835c910fc5f906627968a1a9e810cb164afd634ca683b6bb34f0241808080808080a03ca1d0ead152d38c16bda91dac49e3f0c9a0afeaa67598c1fce2506f5f03162680").to_vec(),
        hex!("f86d9d3c3738deb88e49108e7a5bd83c14ad65b5ba598e2932551dc9b9ad1879b84df84b10874ef05b2fe9d8c8a056e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421a0c5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470").to_vec(),
    ];

    let root = H256(hex!("024c056bc5db60d71c7908c5fad6050646bd70fd772ff222702d577e2af2e56b"));

    let db = StorageProof::new(proof).into_memory_db::<KeccakHasher>();
    let trie = TrieDBBuilder::<EIP1186Layout<KeccakHasher>>::new(&db, &root).build();
    let result = trie.get(&key).unwrap().unwrap();


    // the raw account data stored in the state proof:
    let account = Account::decode(&Rlp::new(&result)).unwrap();

    // some assertions about the account data in the state root.
    assert_eq!(account.balance, U256::from(&hex!("4ef05b2fe9d8c8")[..]));
    assert_eq!(
        account.code_hash,
        H256::from(hex!("c5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470"))
    );
    assert_eq!(
        account.storage_root,
        H256::from(hex!("56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421"))
    );
    assert_eq!(account.nonce, 0x10);
}
```

### No Std 

This library supports `no_std`, simply add `default-features = false` to the dependecncy entry in your `Cargo.toml`