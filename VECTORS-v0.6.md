# Normative publication vectors

Status: **v0.6 spatial-only correction under review**. v0.6 is **reopened in place** by Founder decision on **2026-10-29**; the **2026-10-27 freeze was explicitly reopened**. **Re-freeze happens only after independent evidence plus Founder acceptance**. The `punkti/0.6` namespace and frozen v0.5.1 history remain unchanged.

These vectors are self-contained over logical bytes, not filesystem paths.
They use the [carrier](Punkti-v0.6.md), [spatial profile](Punkti-v0.6-spatial.md)
and [publication contract](Punkti-v0.6-publication.md). Success verifies referenced
objects at read time; it makes no global completeness or freshness claim.

### Deterministic fixture construction

* Signing seed: SHA-256 of the ASCII literal `static-test-key`, used as the
  32-byte Ed25519 private seed. Use standard Ed25519 over
  `ASCII("PUNKTI-L3-SIGN") || 0x00 || C` and the unchanged carrier framing.
* V4 embeds every positive base record as exact `R` hex. Decode C and sig from R;
  compute MID with `ASCII("PUNKTI-L3-MID") || 0x00 || C`, and record digest with
  SHA-256(R). The former empty-address fixture is preserved byte-for-byte as a
  V10 negative record; it is excluded from every conforming publication.
* `index`: publish all V4 records without packs. `packs`: the same records with
  packs. `small-pack`: the first two V4 records with packs. `poison`: all four
  V5 variants without packs, including the intentionally forged variant.
  These adversarial fixtures test reader admission, not publisher eligibility.
* Build leaves at `K(xy)[:10]`, sorted by record SHA-256; build all nonempty
  interior shards at depths 9 down to 2. Each shard has exactly `children`,
  `packs`, `prefix`, `records`, `v`. Empty arrays/maps remain present. In the
  pack profile, group sorted records in consecutive groups of at most 256;
  each pack is `ASCII("PUNKTIPACK1") || U32(count) || LP(R)...` (11-byte magic).
  Sort pack addresses. All record objects remain separately available.
* Hash canonical JSON (UTF-8, sorted keys, separators `,` and `:`, no newline),
  retaining every immutable object. The manifest has `index`, `pack_count`,
  `prev`, `record_count`, `seq`, `v`; start at seq 1, prev null. Root has
  `head`, `seq`, `v`; publication v is `punkti/pub/0.6`.
* `newer`: publish the same V4 record set with packs at seq 2 and prev equal
  to the `index` manifest digest. This changes published state. Prime the valid
  store and last accepted root with this source, then reject the older `index`
  root. `retry` repeats the successful read into the same stores; it leaves
  contents unchanged. `admitted` counts eligible VALID records processed by
  that read, including duplicate no-ops, not newly inserted records.
* `tamper-manifest`, `tamper-shard`, `tamper-record`: start with `index`; choose
  the lexicographically smallest digest of the named object kind and XOR the
  first byte with 1, retaining its original address. `missing`: remove the
  record object with the smallest digest. Reads stage admission until all
  in-scope objects have been checked.
* `pack-*`: start with `small-pack`. Mutate its sole pack as follows, then
  recompute its digest and every ancestor shard, manifest and root digest.
  `count`: increment the U32 count only; `truncation`: drop the final byte;
  `trailing`: append a zero byte; `foreign`: add V4's third record and sort by
  SHA-256; `duplicate`: replace entries with two copies of the first sorted
  entry; `reversed`: reverse entry order. Keep the original leaf record entries.
* `scope` omitted members are null/unbounded. `forbidden_records` must not be
  fetched; `required_records` must be fetched even if exact Z filtering excludes
  them. Empty scope selects all anchored records, including the real zero cell.
* V10 envelope cases start from V4's first decoded record; set `alg` as shown
  and sign with its original valid `h`, then replace/remove only `h` as shown.
  Their signatures do not excuse invalid structure. Test **both** supported and
  unsupported algorithms. `input` is the complete envelope. Raw negative
  records carry correctly computed MID/digest audit values; those do not make
  a structurally invalid record admissible or give it an admitted MID.
* V11 starts from `index`. `index-key`: add `~` mapping to the shard already
  indexed by `u4`; `shard-prefix`: replace the `u4` shard's prefix with `~`;
  `child-key`: add `~` to the `u4` shard's children, pointing to its first child
  in sorted order. Rehash changed objects and ancestors, including root.head,
  so these fail geographical structure, not digest integrity. Validate all
  encountered index/child keys before scope pruning. `~` is removed and MUST
  NOT be emitted or accepted in a conforming publication.
* V12 uses V4's first decoded record, replacing `h` and `alg` as evident in
  its R, then re-signing with the same seed. It pins valid length boundaries
  and continued opaque carriage of supported structure with an unknown alg.

V1–V9 retain every original case. Only the expectations that depended on the
removed empty-address fixture change. V9 defines the common expected result for
independent static readers; the vectors themselves do not certify an implementation.
V10–V11 are negative structural cases; V12 supplies additional positive cases.

## V1

```json
{
  "cases": [
    {
      "expected": {
        "admitted": 5,
        "candidate_count": 0,
        "candidates": {},
        "error_class": null,
        "head": "55abfde8510889f4dad27a69d4b8d9d15a44168a23f5fc44dfae042efc305728",
        "reason": null,
        "rejected": 0,
        "seq": 1,
        "valid": {
          "punkti/0.6:0d627f184791bb85d0cbd5dbd8a4fda461de4ad9462066820f099f220462194f": "cdc66507dfe4e5e39d9c8efbafabeb1520a4c7d13301289d70c33af436e983bb",
          "punkti/0.6:2db7fa7aa91963a454c14b5bf33b6b68691e1addc3f9726c10b7398035add7e9": "109139f14294cdd1133348ce1002bb8f6883f9636756a636e21cc2bd44f9d584",
          "punkti/0.6:9b944678c2cb3a22263caf4140da8091e3c723ee9cf4475c62f5d52e32d662f6": "0be5211906518d907f5a9bbfd5f36e05a84a6a5ec31891d27de18017c1ee61c6",
          "punkti/0.6:c6a6afc75f76a5cd6cebd8830742ec856d421ae3f2b0f03ce622c8af0b85cca6": "25ce5db732f99db12584af17cdefedf2798a28174685cb094f0c95f2db531916",
          "punkti/0.6:fe600612fe892a9fa7952213528e637cb96f2e15c4a401d63fa58892826ec0f7": "c79451cba4bf2995d8df8958957d53b95cbf17276f44f3e71326a0cdf6c0a4b0"
        },
        "valid_count": 5
      },
      "name": "root-manifest",
      "packs": false,
      "source": "index"
    },
    {
      "expected": {
        "admitted": 5,
        "candidate_count": 0,
        "candidates": {},
        "error_class": null,
        "head": "5c91c2f2a2fd98ff83f4bef18106b378c9c7cabaa9a7ae65b7ef3cca60031fd9",
        "reason": null,
        "rejected": 0,
        "seq": 1,
        "valid": {
          "punkti/0.6:0d627f184791bb85d0cbd5dbd8a4fda461de4ad9462066820f099f220462194f": "cdc66507dfe4e5e39d9c8efbafabeb1520a4c7d13301289d70c33af436e983bb",
          "punkti/0.6:2db7fa7aa91963a454c14b5bf33b6b68691e1addc3f9726c10b7398035add7e9": "109139f14294cdd1133348ce1002bb8f6883f9636756a636e21cc2bd44f9d584",
          "punkti/0.6:9b944678c2cb3a22263caf4140da8091e3c723ee9cf4475c62f5d52e32d662f6": "0be5211906518d907f5a9bbfd5f36e05a84a6a5ec31891d27de18017c1ee61c6",
          "punkti/0.6:c6a6afc75f76a5cd6cebd8830742ec856d421ae3f2b0f03ce622c8af0b85cca6": "25ce5db732f99db12584af17cdefedf2798a28174685cb094f0c95f2db531916",
          "punkti/0.6:fe600612fe892a9fa7952213528e637cb96f2e15c4a401d63fa58892826ec0f7": "c79451cba4bf2995d8df8958957d53b95cbf17276f44f3e71326a0cdf6c0a4b0"
        },
        "valid_count": 5
      },
      "name": "pack-discovery",
      "packs": true,
      "source": "packs"
    }
  ],
  "id": "V1"
}
```

## V2

```json
{
  "cases": [
    {
      "expected": {
        "admitted": 0,
        "candidate_count": 0,
        "candidates": {},
        "error_class": "retryable",
        "reason": "corrupt object",
        "rejected": 0,
        "valid": {},
        "valid_count": 0
      },
      "name": "manifest",
      "source": "tamper-manifest"
    },
    {
      "expected": {
        "admitted": 0,
        "candidate_count": 0,
        "candidates": {},
        "error_class": "retryable",
        "reason": "corrupt object",
        "rejected": 0,
        "valid": {},
        "valid_count": 0
      },
      "name": "shard",
      "source": "tamper-shard"
    },
    {
      "expected": {
        "admitted": 0,
        "candidate_count": 0,
        "candidates": {},
        "error_class": "retryable",
        "reason": "corrupt object",
        "rejected": 0,
        "valid": {},
        "valid_count": 0
      },
      "name": "record",
      "source": "tamper-record"
    }
  ],
  "id": "V2"
}
```

## V3

```json
{
  "cases": [
    {
      "expected": {
        "admitted": 2,
        "candidate_count": 0,
        "candidates": {},
        "error_class": null,
        "head": "1d5da15b1d875867e6cbb94b21f2713818a9ba167a162017a967794814b74b1a",
        "reason": null,
        "rejected": 0,
        "seq": 1,
        "valid": {
          "punkti/0.6:0d627f184791bb85d0cbd5dbd8a4fda461de4ad9462066820f099f220462194f": "cdc66507dfe4e5e39d9c8efbafabeb1520a4c7d13301289d70c33af436e983bb",
          "punkti/0.6:c6a6afc75f76a5cd6cebd8830742ec856d421ae3f2b0f03ce622c8af0b85cca6": "25ce5db732f99db12584af17cdefedf2798a28174685cb094f0c95f2db531916"
        },
        "valid_count": 2
      },
      "name": "valid",
      "packs": true,
      "source": "small-pack"
    },
    {
      "expected": {
        "admitted": 0,
        "candidate_count": 0,
        "candidates": {},
        "error_class": "retryable",
        "reason": "invalid object",
        "rejected": 0,
        "valid": {},
        "valid_count": 0
      },
      "name": "count",
      "packs": true,
      "source": "pack-count"
    },
    {
      "expected": {
        "admitted": 0,
        "candidate_count": 0,
        "candidates": {},
        "error_class": "retryable",
        "reason": "invalid object",
        "rejected": 0,
        "valid": {},
        "valid_count": 0
      },
      "name": "truncation",
      "packs": true,
      "source": "pack-truncation"
    },
    {
      "expected": {
        "admitted": 0,
        "candidate_count": 0,
        "candidates": {},
        "error_class": "retryable",
        "reason": "invalid object",
        "rejected": 0,
        "valid": {},
        "valid_count": 0
      },
      "name": "trailing",
      "packs": true,
      "source": "pack-trailing"
    },
    {
      "expected": {
        "admitted": 0,
        "candidate_count": 0,
        "candidates": {},
        "error_class": "retryable",
        "reason": "pack membership",
        "rejected": 0,
        "valid": {},
        "valid_count": 0
      },
      "name": "foreign",
      "packs": true,
      "source": "pack-foreign"
    },
    {
      "expected": {
        "admitted": 0,
        "candidate_count": 0,
        "candidates": {},
        "error_class": "retryable",
        "reason": "pack membership",
        "rejected": 0,
        "valid": {},
        "valid_count": 0
      },
      "name": "duplicate",
      "packs": true,
      "source": "pack-duplicate"
    },
    {
      "expected": {
        "admitted": 0,
        "candidate_count": 0,
        "candidates": {},
        "error_class": "retryable",
        "reason": "pack membership",
        "rejected": 0,
        "valid": {},
        "valid_count": 0
      },
      "name": "reversed",
      "packs": true,
      "source": "pack-reversed"
    }
  ],
  "id": "V3"
}
```

## V4

```json
{
  "cases": [
    {
      "expected": {
        "admitted": 5,
        "candidate_count": 0,
        "candidates": {},
        "error_class": null,
        "head": "55abfde8510889f4dad27a69d4b8d9d15a44168a23f5fc44dfae042efc305728",
        "reason": null,
        "rejected": 0,
        "seq": 1,
        "valid": {
          "punkti/0.6:0d627f184791bb85d0cbd5dbd8a4fda461de4ad9462066820f099f220462194f": "cdc66507dfe4e5e39d9c8efbafabeb1520a4c7d13301289d70c33af436e983bb",
          "punkti/0.6:2db7fa7aa91963a454c14b5bf33b6b68691e1addc3f9726c10b7398035add7e9": "109139f14294cdd1133348ce1002bb8f6883f9636756a636e21cc2bd44f9d584",
          "punkti/0.6:9b944678c2cb3a22263caf4140da8091e3c723ee9cf4475c62f5d52e32d662f6": "0be5211906518d907f5a9bbfd5f36e05a84a6a5ec31891d27de18017c1ee61c6",
          "punkti/0.6:c6a6afc75f76a5cd6cebd8830742ec856d421ae3f2b0f03ce622c8af0b85cca6": "25ce5db732f99db12584af17cdefedf2798a28174685cb094f0c95f2db531916",
          "punkti/0.6:fe600612fe892a9fa7952213528e637cb96f2e15c4a401d63fa58892826ec0f7": "c79451cba4bf2995d8df8958957d53b95cbf17276f44f3e71326a0cdf6c0a4b0"
        },
        "valid_count": 5
      },
      "name": "exact-R-MID",
      "source": "index"
    }
  ],
  "id": "V4",
  "records": [
    {
      "hex": "000000690000000a70756e6b74692f302e3600000001310000000e753470727579647171765f2d313800000009666978747572652d31000000076564323535313900000020ef3dfc2a574748f04377ff1abb00c43c897e4d64058de1ba690b3895fbb34007000000000000000000000040c7092f1147a414675c618a64b6040ea941efcb7e72b7696414f21a1242ede73fa40a2ffa3930701cff96c46dea14d095cc291b4627788c43275e6381bf28660f",
      "mid": "punkti/0.6:0d627f184791bb85d0cbd5dbd8a4fda461de4ad9462066820f099f220462194f",
      "sha256": "cdc66507dfe4e5e39d9c8efbafabeb1520a4c7d13301289d70c33af436e983bb"
    },
    {
      "hex": "0000006c0000000a70756e6b74692f302e36000000013200000011753470727579647171765f32332e30323500000009666978747572652d32000000076564323535313900000020ef3dfc2a574748f04377ff1abb00c43c897e4d64058de1ba690b3895fbb34007000000000000000000000040fb58c62f2e2105a9f90755c59ee3a9c7b20f58c4f9a755de87b76382600499e4ef55077a354e101cb934829efbea1d0dfd3ffbb0f5fe43d73ca1810e0987320a",
      "mid": "punkti/0.6:c6a6afc75f76a5cd6cebd8830742ec856d421ae3f2b0f03ce622c8af0b85cca6",
      "sha256": "25ce5db732f99db12584af17cdefedf2798a28174685cb094f0c95f2db531916"
    },
    {
      "hex": "000000620000000a70756e6b74692f302e36000000013300000007657a7334325f3000000009666978747572652d33000000076564323535313900000020ef3dfc2a574748f04377ff1abb00c43c897e4d64058de1ba690b3895fbb340070000000000000000000000402572295d7e7eb49d5641531966c79666df008ded2bdf2dcae77ac3fb839cc5afcd365edcf68d75f1ae3e1326aeddae27a138a6f1d170edb8736d2ea290f6eb06",
      "mid": "punkti/0.6:2db7fa7aa91963a454c14b5bf33b6b68691e1addc3f9726c10b7398035add7e9",
      "sha256": "109139f14294cdd1133348ce1002bb8f6883f9636756a636e21cc2bd44f9d584"
    },
    {
      "hex": "000000610000000a70756e6b74692f302e36000000013500000006303030305f3000000009666978747572652d35000000076564323535313900000020ef3dfc2a574748f04377ff1abb00c43c897e4d64058de1ba690b3895fbb34007000000000000000000000040d99c08654fa7bfc0c6cc983320a64d6cfe5652f64eeab4567ac0a7fc2efb43c9cebd9dcae52cbbca4d6e7a20e2be1d5e003a87f203d9daf4aae59006fee25306",
      "mid": "punkti/0.6:9b944678c2cb3a22263caf4140da8091e3c723ee9cf4475c62f5d52e32d662f6",
      "sha256": "0be5211906518d907f5a9bbfd5f36e05a84a6a5ec31891d27de18017c1ee61c6"
    },
    {
      "hex": "0000007b0000000a70756e6b74692f302e36000000013600000020753470727579647171765f2d302e30303030303030303030303030303030303100000009666978747572652d36000000076564323535313900000020ef3dfc2a574748f04377ff1abb00c43c897e4d64058de1ba690b3895fbb3400700000000000000000000004078577030c43a69fa99f4db3970dc3130f7bd1d9bd558fb56f4cea847e3b174352fe1da3c7397a5222ace32286a05207ab31a587e16f014954f17e9b2bb510701",
      "mid": "punkti/0.6:fe600612fe892a9fa7952213528e637cb96f2e15c4a401d63fa58892826ec0f7",
      "sha256": "c79451cba4bf2995d8df8958957d53b95cbf17276f44f3e71326a0cdf6c0a4b0"
    }
  ]
}
```

## V5

```json
{
  "cases": [
    {
      "expected": {
        "admitted": 1,
        "candidate_count": 2,
        "candidates": {
          "30412bbfb1526f0c36fda7c23f923a37a98337fa0fc6ebe091a6daf67956ce2e": "0000006a0000000a70756e6b74692f302e3600000001370000000c753470727579647171765f3000000009666978747572652d370000000a6675747572652d616c6700000020ef3dfc2a574748f04377ff1abb00c43c897e4d64058de1ba690b3895fbb340070000000000000000000000405d5b32366f23a9b4a97f5e55ff63ffce5700403e6edbc6b7d36ea9551ac86f4dd41afa7b9eca251ac048dec5a687d6b76f2b0ac419dcdc2acd332a1bcf189f04",
          "a959a99ee4f56aa5a805bc3d1eea4834b610dc62e564f59126b44e41eaca46a6": "0000006a0000000a70756e6b74692f302e3600000001370000000c753470727579647171765f3000000009666978747572652d370000000a6675747572652d616c6700000020ef3dfc2a574748f04377ff1abb00c43c897e4d64058de1ba690b3895fbb340070000000000000000000000405d5b32366f23a9b4a97f5e55ff63ffce5700403e6edbc6b7d36ea9551ac86f4dd41afa7b9eca251ac048dec5a687d6b76f2b0ac419dcdc2acd332a1bcf189f05"
        },
        "error_class": null,
        "head": "4f15fb5deb39c86d23d2faaae353eef7e997e062a47afe47aad0da79066d7236",
        "reason": null,
        "rejected": 1,
        "seq": 1,
        "valid": {
          "punkti/0.6:0d627f184791bb85d0cbd5dbd8a4fda461de4ad9462066820f099f220462194f": "cdc66507dfe4e5e39d9c8efbafabeb1520a4c7d13301289d70c33af436e983bb"
        },
        "valid_count": 1
      },
      "name": "variants",
      "source": "poison"
    }
  ],
  "id": "V5",
  "variants": [
    {
      "hex": "000000690000000a70756e6b74692f302e3600000001310000000e753470727579647171765f2d313800000009666978747572652d31000000076564323535313900000020ef3dfc2a574748f04377ff1abb00c43c897e4d64058de1ba690b3895fbb34007000000000000000000000040c7092f1147a414675c618a64b6040ea941efcb7e72b7696414f21a1242ede73fa40a2ffa3930701cff96c46dea14d095cc291b4627788c43275e6381bf28660f",
      "mid": "punkti/0.6:0d627f184791bb85d0cbd5dbd8a4fda461de4ad9462066820f099f220462194f",
      "sha256": "cdc66507dfe4e5e39d9c8efbafabeb1520a4c7d13301289d70c33af436e983bb",
      "state": "VALID"
    },
    {
      "hex": "000000690000000a70756e6b74692f302e3600000001310000000e753470727579647171765f2d313800000009666978747572652d31000000076564323535313900000020ef3dfc2a574748f04377ff1abb00c43c897e4d64058de1ba690b3895fbb34007000000000000000000000040c7092f1147a414675c618a64b6040ea941efcb7e72b7696414f21a1242ede73fa40a2ffa3930701cff96c46dea14d095cc291b4627788c43275e6381bf28660e",
      "mid": "punkti/0.6:0d627f184791bb85d0cbd5dbd8a4fda461de4ad9462066820f099f220462194f",
      "sha256": "102b622f347944d7db7501124e8e2487b583a554eb7c5165a82d040a5f89126c",
      "state": "INVALID-provenance"
    },
    {
      "hex": "0000006a0000000a70756e6b74692f302e3600000001370000000c753470727579647171765f3000000009666978747572652d370000000a6675747572652d616c6700000020ef3dfc2a574748f04377ff1abb00c43c897e4d64058de1ba690b3895fbb340070000000000000000000000405d5b32366f23a9b4a97f5e55ff63ffce5700403e6edbc6b7d36ea9551ac86f4dd41afa7b9eca251ac048dec5a687d6b76f2b0ac419dcdc2acd332a1bcf189f04",
      "mid": "punkti/0.6:d84ec6f7a8623d4ea8c4215a24e6d8892f86d216a07b5e2ff8cfcb8c1402cbbb",
      "sha256": "30412bbfb1526f0c36fda7c23f923a37a98337fa0fc6ebe091a6daf67956ce2e",
      "state": "UNVERIFIABLE-alg"
    },
    {
      "hex": "0000006a0000000a70756e6b74692f302e3600000001370000000c753470727579647171765f3000000009666978747572652d370000000a6675747572652d616c6700000020ef3dfc2a574748f04377ff1abb00c43c897e4d64058de1ba690b3895fbb340070000000000000000000000405d5b32366f23a9b4a97f5e55ff63ffce5700403e6edbc6b7d36ea9551ac86f4dd41afa7b9eca251ac048dec5a687d6b76f2b0ac419dcdc2acd332a1bcf189f05",
      "mid": "punkti/0.6:d84ec6f7a8623d4ea8c4215a24e6d8892f86d216a07b5e2ff8cfcb8c1402cbbb",
      "sha256": "a959a99ee4f56aa5a805bc3d1eea4834b610dc62e564f59126b44e41eaca46a6",
      "state": "UNVERIFIABLE-alg"
    }
  ]
}
```

## V6

```json
{
  "cases": [
    {
      "expected": {
        "admitted": 3,
        "candidate_count": 0,
        "candidates": {},
        "error_class": null,
        "head": "55abfde8510889f4dad27a69d4b8d9d15a44168a23f5fc44dfae042efc305728",
        "reason": null,
        "rejected": 0,
        "seq": 1,
        "valid": {
          "punkti/0.6:0d627f184791bb85d0cbd5dbd8a4fda461de4ad9462066820f099f220462194f": "cdc66507dfe4e5e39d9c8efbafabeb1520a4c7d13301289d70c33af436e983bb",
          "punkti/0.6:c6a6afc75f76a5cd6cebd8830742ec856d421ae3f2b0f03ce622c8af0b85cca6": "25ce5db732f99db12584af17cdefedf2798a28174685cb094f0c95f2db531916",
          "punkti/0.6:fe600612fe892a9fa7952213528e637cb96f2e15c4a401d63fa58892826ec0f7": "c79451cba4bf2995d8df8958957d53b95cbf17276f44f3e71326a0cdf6c0a4b0"
        },
        "valid_count": 3
      },
      "forbidden_records": [
        "109139f14294cdd1133348ce1002bb8f6883f9636756a636e21cc2bd44f9d584",
        "0be5211906518d907f5a9bbfd5f36e05a84a6a5ec31891d27de18017c1ee61c6"
      ],
      "name": "XY",
      "scope": {
        "xy_hi": "u5",
        "xy_lo": "u4"
      },
      "source": "index"
    },
    {
      "expected": {
        "admitted": 1,
        "candidate_count": 0,
        "candidates": {},
        "error_class": null,
        "head": "55abfde8510889f4dad27a69d4b8d9d15a44168a23f5fc44dfae042efc305728",
        "reason": null,
        "rejected": 0,
        "seq": 1,
        "valid": {
          "punkti/0.6:9b944678c2cb3a22263caf4140da8091e3c723ee9cf4475c62f5d52e32d662f6": "0be5211906518d907f5a9bbfd5f36e05a84a6a5ec31891d27de18017c1ee61c6"
        },
        "valid_count": 1
      },
      "forbidden_records": [],
      "name": "zero-cell-is-geographical",
      "scope": {
        "xy_hi": "1",
        "xy_lo": "0"
      },
      "source": "index"
    },
    {
      "expected": {
        "admitted": 5,
        "candidate_count": 0,
        "candidates": {},
        "error_class": null,
        "head": "55abfde8510889f4dad27a69d4b8d9d15a44168a23f5fc44dfae042efc305728",
        "reason": null,
        "rejected": 0,
        "seq": 1,
        "valid": {
          "punkti/0.6:0d627f184791bb85d0cbd5dbd8a4fda461de4ad9462066820f099f220462194f": "cdc66507dfe4e5e39d9c8efbafabeb1520a4c7d13301289d70c33af436e983bb",
          "punkti/0.6:2db7fa7aa91963a454c14b5bf33b6b68691e1addc3f9726c10b7398035add7e9": "109139f14294cdd1133348ce1002bb8f6883f9636756a636e21cc2bd44f9d584",
          "punkti/0.6:9b944678c2cb3a22263caf4140da8091e3c723ee9cf4475c62f5d52e32d662f6": "0be5211906518d907f5a9bbfd5f36e05a84a6a5ec31891d27de18017c1ee61c6",
          "punkti/0.6:c6a6afc75f76a5cd6cebd8830742ec856d421ae3f2b0f03ce622c8af0b85cca6": "25ce5db732f99db12584af17cdefedf2798a28174685cb094f0c95f2db531916",
          "punkti/0.6:fe600612fe892a9fa7952213528e637cb96f2e15c4a401d63fa58892826ec0f7": "c79451cba4bf2995d8df8958957d53b95cbf17276f44f3e71326a0cdf6c0a4b0"
        },
        "valid_count": 5
      },
      "name": "unrestricted-all-anchored",
      "scope": {
        "xy_hi": null,
        "xy_lo": null,
        "z_max": null,
        "z_min": null
      },
      "source": "index"
    }
  ],
  "id": "V6"
}
```

## V7

```json
{
  "cases": [
    {
      "expected": {
        "admitted": 2,
        "candidate_count": 0,
        "candidates": {},
        "error_class": null,
        "head": "55abfde8510889f4dad27a69d4b8d9d15a44168a23f5fc44dfae042efc305728",
        "reason": null,
        "rejected": 0,
        "seq": 1,
        "valid": {
          "punkti/0.6:0d627f184791bb85d0cbd5dbd8a4fda461de4ad9462066820f099f220462194f": "cdc66507dfe4e5e39d9c8efbafabeb1520a4c7d13301289d70c33af436e983bb",
          "punkti/0.6:fe600612fe892a9fa7952213528e637cb96f2e15c4a401d63fa58892826ec0f7": "c79451cba4bf2995d8df8958957d53b95cbf17276f44f3e71326a0cdf6c0a4b0"
        },
        "valid_count": 2
      },
      "name": "negative-exact-Z",
      "required_records": [
        "25ce5db732f99db12584af17cdefedf2798a28174685cb094f0c95f2db531916"
      ],
      "scope": {
        "z_max": "0",
        "z_min": "-18"
      },
      "source": "index"
    },
    {
      "expected": {
        "admitted": 1,
        "candidate_count": 0,
        "candidates": {},
        "error_class": null,
        "head": "55abfde8510889f4dad27a69d4b8d9d15a44168a23f5fc44dfae042efc305728",
        "reason": null,
        "rejected": 0,
        "seq": 1,
        "valid": {
          "punkti/0.6:fe600612fe892a9fa7952213528e637cb96f2e15c4a401d63fa58892826ec0f7": "c79451cba4bf2995d8df8958957d53b95cbf17276f44f3e71326a0cdf6c0a4b0"
        },
        "valid_count": 1
      },
      "name": "decimal-upper-exclusive",
      "scope": {
        "xy_hi": "u5",
        "xy_lo": "u4",
        "z_max": "23.025",
        "z_min": "-1"
      },
      "source": "index"
    }
  ],
  "id": "V7"
}
```

## V8

```json
{
  "cases": [
    {
      "expected": {
        "admitted": 0,
        "candidate_count": 0,
        "candidates": {},
        "error_class": "retryable",
        "reason": "missing object",
        "rejected": 0,
        "valid": {},
        "valid_count": 0
      },
      "name": "missing",
      "source": "missing"
    },
    {
      "expected": {
        "admitted": 0,
        "candidate_count": 0,
        "candidates": {},
        "error_class": "retryable",
        "reason": "corrupt object",
        "rejected": 0,
        "valid": {},
        "valid_count": 0
      },
      "name": "corrupt",
      "source": "tamper-record"
    },
    {
      "expected": {
        "admitted": 0,
        "candidate_count": 0,
        "candidates": {},
        "error_class": "retryable",
        "reason": "stale root",
        "rejected": 0,
        "valid": {
          "punkti/0.6:0d627f184791bb85d0cbd5dbd8a4fda461de4ad9462066820f099f220462194f": "cdc66507dfe4e5e39d9c8efbafabeb1520a4c7d13301289d70c33af436e983bb",
          "punkti/0.6:2db7fa7aa91963a454c14b5bf33b6b68691e1addc3f9726c10b7398035add7e9": "109139f14294cdd1133348ce1002bb8f6883f9636756a636e21cc2bd44f9d584",
          "punkti/0.6:9b944678c2cb3a22263caf4140da8091e3c723ee9cf4475c62f5d52e32d662f6": "0be5211906518d907f5a9bbfd5f36e05a84a6a5ec31891d27de18017c1ee61c6",
          "punkti/0.6:c6a6afc75f76a5cd6cebd8830742ec856d421ae3f2b0f03ce622c8af0b85cca6": "25ce5db732f99db12584af17cdefedf2798a28174685cb094f0c95f2db531916",
          "punkti/0.6:fe600612fe892a9fa7952213528e637cb96f2e15c4a401d63fa58892826ec0f7": "c79451cba4bf2995d8df8958957d53b95cbf17276f44f3e71326a0cdf6c0a4b0"
        },
        "valid_count": 5
      },
      "name": "stale",
      "prime": "newer",
      "source": "index"
    },
    {
      "expected": {
        "admitted": 5,
        "candidate_count": 0,
        "candidates": {},
        "error_class": null,
        "head": "55abfde8510889f4dad27a69d4b8d9d15a44168a23f5fc44dfae042efc305728",
        "reason": null,
        "rejected": 0,
        "seq": 1,
        "valid": {
          "punkti/0.6:0d627f184791bb85d0cbd5dbd8a4fda461de4ad9462066820f099f220462194f": "cdc66507dfe4e5e39d9c8efbafabeb1520a4c7d13301289d70c33af436e983bb",
          "punkti/0.6:2db7fa7aa91963a454c14b5bf33b6b68691e1addc3f9726c10b7398035add7e9": "109139f14294cdd1133348ce1002bb8f6883f9636756a636e21cc2bd44f9d584",
          "punkti/0.6:9b944678c2cb3a22263caf4140da8091e3c723ee9cf4475c62f5d52e32d662f6": "0be5211906518d907f5a9bbfd5f36e05a84a6a5ec31891d27de18017c1ee61c6",
          "punkti/0.6:c6a6afc75f76a5cd6cebd8830742ec856d421ae3f2b0f03ce622c8af0b85cca6": "25ce5db732f99db12584af17cdefedf2798a28174685cb094f0c95f2db531916",
          "punkti/0.6:fe600612fe892a9fa7952213528e637cb96f2e15c4a401d63fa58892826ec0f7": "c79451cba4bf2995d8df8958957d53b95cbf17276f44f3e71326a0cdf6c0a4b0"
        },
        "valid_count": 5
      },
      "name": "retry-idempotent",
      "retry": true,
      "source": "index"
    }
  ],
  "id": "V8"
}
```

## V9

```json
{
  "cases": [
    {
      "expected": {
        "admitted": 5,
        "candidate_count": 0,
        "candidates": {},
        "error_class": null,
        "head": "55abfde8510889f4dad27a69d4b8d9d15a44168a23f5fc44dfae042efc305728",
        "reason": null,
        "rejected": 0,
        "seq": 1,
        "valid": {
          "punkti/0.6:0d627f184791bb85d0cbd5dbd8a4fda461de4ad9462066820f099f220462194f": "cdc66507dfe4e5e39d9c8efbafabeb1520a4c7d13301289d70c33af436e983bb",
          "punkti/0.6:2db7fa7aa91963a454c14b5bf33b6b68691e1addc3f9726c10b7398035add7e9": "109139f14294cdd1133348ce1002bb8f6883f9636756a636e21cc2bd44f9d584",
          "punkti/0.6:9b944678c2cb3a22263caf4140da8091e3c723ee9cf4475c62f5d52e32d662f6": "0be5211906518d907f5a9bbfd5f36e05a84a6a5ec31891d27de18017c1ee61c6",
          "punkti/0.6:c6a6afc75f76a5cd6cebd8830742ec856d421ae3f2b0f03ce622c8af0b85cca6": "25ce5db732f99db12584af17cdefedf2798a28174685cb094f0c95f2db531916",
          "punkti/0.6:fe600612fe892a9fa7952213528e637cb96f2e15c4a401d63fa58892826ec0f7": "c79451cba4bf2995d8df8958957d53b95cbf17276f44f3e71326a0cdf6c0a4b0"
        },
        "valid_count": 5
      },
      "name": "static-index",
      "source": "index"
    },
    {
      "expected": {
        "admitted": 5,
        "candidate_count": 0,
        "candidates": {},
        "error_class": null,
        "head": "5c91c2f2a2fd98ff83f4bef18106b378c9c7cabaa9a7ae65b7ef3cca60031fd9",
        "reason": null,
        "rejected": 0,
        "seq": 1,
        "valid": {
          "punkti/0.6:0d627f184791bb85d0cbd5dbd8a4fda461de4ad9462066820f099f220462194f": "cdc66507dfe4e5e39d9c8efbafabeb1520a4c7d13301289d70c33af436e983bb",
          "punkti/0.6:2db7fa7aa91963a454c14b5bf33b6b68691e1addc3f9726c10b7398035add7e9": "109139f14294cdd1133348ce1002bb8f6883f9636756a636e21cc2bd44f9d584",
          "punkti/0.6:9b944678c2cb3a22263caf4140da8091e3c723ee9cf4475c62f5d52e32d662f6": "0be5211906518d907f5a9bbfd5f36e05a84a6a5ec31891d27de18017c1ee61c6",
          "punkti/0.6:c6a6afc75f76a5cd6cebd8830742ec856d421ae3f2b0f03ce622c8af0b85cca6": "25ce5db732f99db12584af17cdefedf2798a28174685cb094f0c95f2db531916",
          "punkti/0.6:fe600612fe892a9fa7952213528e637cb96f2e15c4a401d63fa58892826ec0f7": "c79451cba4bf2995d8df8958957d53b95cbf17276f44f3e71326a0cdf6c0a4b0"
        },
        "valid_count": 5
      },
      "name": "static-packs",
      "packs": true,
      "source": "packs"
    }
  ],
  "id": "V9"
}
```

## V10

```json
{
  "cases": [
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "ed25519",
        "ext": "",
        "h": "",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "xwkvEUekFGdcYYpktgQOqUHvy35yt2lkFPIaEkLt5z+kCi/6OTBwHP+WxG3qFNCVzCkbRid4jEMnXmOBvyhmDw==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "empty-ed25519"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "future-alg",
        "ext": "",
        "h": "",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "FJYnB+ToeIxb16iyQWVloUhbLpD+9s7Yh5gDsQtgfTspNeUcpXvEM/65JA0j7ukxZaqomByfNrEUCBYs5qrnCQ==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "empty-future-alg"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "ed25519",
        "ext": "",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "xwkvEUekFGdcYYpktgQOqUHvy35yt2lkFPIaEkLt5z+kCi/6OTBwHP+WxG3qFNCVzCkbRid4jEMnXmOBvyhmDw==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "missing-ed25519"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "future-alg",
        "ext": "",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "FJYnB+ToeIxb16iyQWVloUhbLpD+9s7Yh5gDsQtgfTspNeUcpXvEM/65JA0j7ukxZaqomByfNrEUCBYs5qrnCQ==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "missing-future-alg"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "ed25519",
        "ext": "",
        "h": null,
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "xwkvEUekFGdcYYpktgQOqUHvy35yt2lkFPIaEkLt5z+kCi/6OTBwHP+WxG3qFNCVzCkbRid4jEMnXmOBvyhmDw==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "null-ed25519"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "future-alg",
        "ext": "",
        "h": null,
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "FJYnB+ToeIxb16iyQWVloUhbLpD+9s7Yh5gDsQtgfTspNeUcpXvEM/65JA0j7ukxZaqomByfNrEUCBYs5qrnCQ==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "null-future-alg"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "ed25519",
        "ext": "",
        "h": "u3b_0",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "xwkvEUekFGdcYYpktgQOqUHvy35yt2lkFPIaEkLt5z+kCi/6OTBwHP+WxG3qFNCVzCkbRid4jEMnXmOBvyhmDw==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "xy-too-short-ed25519"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "future-alg",
        "ext": "",
        "h": "u3b_0",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "FJYnB+ToeIxb16iyQWVloUhbLpD+9s7Yh5gDsQtgfTspNeUcpXvEM/65JA0j7ukxZaqomByfNrEUCBYs5qrnCQ==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "xy-too-short-future-alg"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "ed25519",
        "ext": "",
        "h": "u3butz7k9q7v0_0",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "xwkvEUekFGdcYYpktgQOqUHvy35yt2lkFPIaEkLt5z+kCi/6OTBwHP+WxG3qFNCVzCkbRid4jEMnXmOBvyhmDw==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "xy-too-long-ed25519"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "future-alg",
        "ext": "",
        "h": "u3butz7k9q7v0_0",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "FJYnB+ToeIxb16iyQWVloUhbLpD+9s7Yh5gDsQtgfTspNeUcpXvEM/65JA0j7ukxZaqomByfNrEUCBYs5qrnCQ==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "xy-too-long-future-alg"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "ed25519",
        "ext": "",
        "h": "u3ba_0",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "xwkvEUekFGdcYYpktgQOqUHvy35yt2lkFPIaEkLt5z+kCi/6OTBwHP+WxG3qFNCVzCkbRid4jEMnXmOBvyhmDw==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "xy-bad-alphabet-ed25519"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "future-alg",
        "ext": "",
        "h": "u3ba_0",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "FJYnB+ToeIxb16iyQWVloUhbLpD+9s7Yh5gDsQtgfTspNeUcpXvEM/65JA0j7ukxZaqomByfNrEUCBYs5qrnCQ==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "xy-bad-alphabet-future-alg"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "ed25519",
        "ext": "",
        "h": "U3BU_0",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "xwkvEUekFGdcYYpktgQOqUHvy35yt2lkFPIaEkLt5z+kCi/6OTBwHP+WxG3qFNCVzCkbRid4jEMnXmOBvyhmDw==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "xy-uppercase-ed25519"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "future-alg",
        "ext": "",
        "h": "U3BU_0",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "FJYnB+ToeIxb16iyQWVloUhbLpD+9s7Yh5gDsQtgfTspNeUcpXvEM/65JA0j7ukxZaqomByfNrEUCBYs5qrnCQ==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "xy-uppercase-future-alg"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "ed25519",
        "ext": "",
        "h": "u3bu_+1",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "xwkvEUekFGdcYYpktgQOqUHvy35yt2lkFPIaEkLt5z+kCi/6OTBwHP+WxG3qFNCVzCkbRid4jEMnXmOBvyhmDw==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "z-plus-ed25519"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "future-alg",
        "ext": "",
        "h": "u3bu_+1",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "FJYnB+ToeIxb16iyQWVloUhbLpD+9s7Yh5gDsQtgfTspNeUcpXvEM/65JA0j7ukxZaqomByfNrEUCBYs5qrnCQ==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "z-plus-future-alg"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "ed25519",
        "ext": "",
        "h": "u3bu_-0",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "xwkvEUekFGdcYYpktgQOqUHvy35yt2lkFPIaEkLt5z+kCi/6OTBwHP+WxG3qFNCVzCkbRid4jEMnXmOBvyhmDw==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "z-negative-zero-ed25519"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "future-alg",
        "ext": "",
        "h": "u3bu_-0",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "FJYnB+ToeIxb16iyQWVloUhbLpD+9s7Yh5gDsQtgfTspNeUcpXvEM/65JA0j7ukxZaqomByfNrEUCBYs5qrnCQ==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "z-negative-zero-future-alg"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "ed25519",
        "ext": "",
        "h": "u3bu_0.0",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "xwkvEUekFGdcYYpktgQOqUHvy35yt2lkFPIaEkLt5z+kCi/6OTBwHP+WxG3qFNCVzCkbRid4jEMnXmOBvyhmDw==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "z-decimal-zero-ed25519"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "future-alg",
        "ext": "",
        "h": "u3bu_0.0",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "FJYnB+ToeIxb16iyQWVloUhbLpD+9s7Yh5gDsQtgfTspNeUcpXvEM/65JA0j7ukxZaqomByfNrEUCBYs5qrnCQ==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "z-decimal-zero-future-alg"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "ed25519",
        "ext": "",
        "h": "u3bu_23.",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "xwkvEUekFGdcYYpktgQOqUHvy35yt2lkFPIaEkLt5z+kCi/6OTBwHP+WxG3qFNCVzCkbRid4jEMnXmOBvyhmDw==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "z-empty-fraction-ed25519"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "future-alg",
        "ext": "",
        "h": "u3bu_23.",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "FJYnB+ToeIxb16iyQWVloUhbLpD+9s7Yh5gDsQtgfTspNeUcpXvEM/65JA0j7ukxZaqomByfNrEUCBYs5qrnCQ==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "z-empty-fraction-future-alg"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "ed25519",
        "ext": "",
        "h": "u3bu_2.3e1",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "xwkvEUekFGdcYYpktgQOqUHvy35yt2lkFPIaEkLt5z+kCi/6OTBwHP+WxG3qFNCVzCkbRid4jEMnXmOBvyhmDw==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "z-exponent-ed25519"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "future-alg",
        "ext": "",
        "h": "u3bu_2.3e1",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "FJYnB+ToeIxb16iyQWVloUhbLpD+9s7Yh5gDsQtgfTspNeUcpXvEM/65JA0j7ukxZaqomByfNrEUCBYs5qrnCQ==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "z-exponent-future-alg"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "ed25519",
        "ext": "",
        "h": "u3bu_٢٣",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "xwkvEUekFGdcYYpktgQOqUHvy35yt2lkFPIaEkLt5z+kCi/6OTBwHP+WxG3qFNCVzCkbRid4jEMnXmOBvyhmDw==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "z-non-ascii-ed25519"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "future-alg",
        "ext": "",
        "h": "u3bu_٢٣",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "FJYnB+ToeIxb16iyQWVloUhbLpD+9s7Yh5gDsQtgfTspNeUcpXvEM/65JA0j7ukxZaqomByfNrEUCBYs5qrnCQ==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "z-non-ascii-future-alg"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "ed25519",
        "ext": "",
        "h": "location",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "xwkvEUekFGdcYYpktgQOqUHvy35yt2lkFPIaEkLt5z+kCi/6OTBwHP+WxG3qFNCVzCkbRid4jEMnXmOBvyhmDw==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "arbitrary-ed25519"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "future-alg",
        "ext": "",
        "h": "location",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "FJYnB+ToeIxb16iyQWVloUhbLpD+9s7Yh5gDsQtgfTspNeUcpXvEM/65JA0j7ukxZaqomByfNrEUCBYs5qrnCQ==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "arbitrary-future-alg"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "ed25519",
        "ext": "",
        "h": "u3bu0",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "xwkvEUekFGdcYYpktgQOqUHvy35yt2lkFPIaEkLt5z+kCi/6OTBwHP+WxG3qFNCVzCkbRid4jEMnXmOBvyhmDw==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "missing-separator-ed25519"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "future-alg",
        "ext": "",
        "h": "u3bu0",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "FJYnB+ToeIxb16iyQWVloUhbLpD+9s7Yh5gDsQtgfTspNeUcpXvEM/65JA0j7ukxZaqomByfNrEUCBYs5qrnCQ==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "missing-separator-future-alg"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "ed25519",
        "ext": "",
        "h": "u3bu__0",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "xwkvEUekFGdcYYpktgQOqUHvy35yt2lkFPIaEkLt5z+kCi/6OTBwHP+WxG3qFNCVzCkbRid4jEMnXmOBvyhmDw==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "double-separator-ed25519"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "future-alg",
        "ext": "",
        "h": "u3bu__0",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "FJYnB+ToeIxb16iyQWVloUhbLpD+9s7Yh5gDsQtgfTspNeUcpXvEM/65JA0j7ukxZaqomByfNrEUCBYs5qrnCQ==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "double-separator-future-alg"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "ed25519",
        "ext": "",
        "h": "u3bu_",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "xwkvEUekFGdcYYpktgQOqUHvy35yt2lkFPIaEkLt5z+kCi/6OTBwHP+WxG3qFNCVzCkbRid4jEMnXmOBvyhmDw==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "empty-z-ed25519"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "future-alg",
        "ext": "",
        "h": "u3bu_",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "FJYnB+ToeIxb16iyQWVloUhbLpD+9s7Yh5gDsQtgfTspNeUcpXvEM/65JA0j7ukxZaqomByfNrEUCBYs5qrnCQ==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "empty-z-future-alg"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "ed25519",
        "ext": "",
        "h": "u3bu_1111111111111111111111111111",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "xwkvEUekFGdcYYpktgQOqUHvy35yt2lkFPIaEkLt5z+kCi/6OTBwHP+WxG3qFNCVzCkbRid4jEMnXmOBvyhmDw==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "overlength-ed25519"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "future-alg",
        "ext": "",
        "h": "u3bu_1111111111111111111111111111",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "FJYnB+ToeIxb16iyQWVloUhbLpD+9s7Yh5gDsQtgfTspNeUcpXvEM/65JA0j7ukxZaqomByfNrEUCBYs5qrnCQ==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "overlength-future-alg"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "ed25519",
        "ext": "",
        "h": "u3bu_023",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "xwkvEUekFGdcYYpktgQOqUHvy35yt2lkFPIaEkLt5z+kCi/6OTBwHP+WxG3qFNCVzCkbRid4jEMnXmOBvyhmDw==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "z-leading-zero-ed25519"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "future-alg",
        "ext": "",
        "h": "u3bu_023",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "FJYnB+ToeIxb16iyQWVloUhbLpD+9s7Yh5gDsQtgfTspNeUcpXvEM/65JA0j7ukxZaqomByfNrEUCBYs5qrnCQ==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "z-leading-zero-future-alg"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "ed25519",
        "ext": "",
        "h": "u3bu_23.50",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "xwkvEUekFGdcYYpktgQOqUHvy35yt2lkFPIaEkLt5z+kCi/6OTBwHP+WxG3qFNCVzCkbRid4jEMnXmOBvyhmDw==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "z-trailing-zero-ed25519"
    },
    {
      "expected": "INVALID-structural",
      "input": {
        "alg": "future-alg",
        "ext": "",
        "h": "u3bu_23.50",
        "pk": "7z38KldHSPBDd/8auwDEPIl+TWQFjeG6aQs4lfuzQAc=",
        "refs": [],
        "sig": "FJYnB+ToeIxb16iyQWVloUhbLpD+9s7Yh5gDsQtgfTspNeUcpXvEM/65JA0j7ukxZaqomByfNrEUCBYs5qrnCQ==",
        "t": 1,
        "v": "punkti/0.6",
        "x": "fixture-1"
      },
      "kind": "envelope",
      "name": "z-trailing-zero-future-alg"
    },
    {
      "expected": "INVALID-structural",
      "hex": "0000005b0000000a70756e6b74692f302e3600000001340000000000000009666978747572652d34000000076564323535313900000020ef3dfc2a574748f04377ff1abb00c43c897e4d64058de1ba690b3895fbb3400700000000000000000000004070a540df8c8ec551a9c20ea65575e44b8fbed42558e5f7594db7bc3880a27de7adcf1794bb56ca168685823c022216643096ce8862c05155badfb2b0994b1206",
      "kind": "record",
      "mid": "punkti/0.6:ccd9220ade890ff8f68f4f90f9ea0b49dc566c5d0f9252fef8bb4145c8550098",
      "name": "former-empty-record",
      "sha256": "f473c6bc447d71bd40bab3cb07fa5a6e972a121b1dd4cdb037ef92449d81c691"
    },
    {
      "expected": "INVALID-structural",
      "hex": "0000005e0000000a70756e6b74692f302e3600000001340000000000000009666978747572652d340000000a6675747572652d616c6700000020ef3dfc2a574748f04377ff1abb00c43c897e4d64058de1ba690b3895fbb340070000000000000000000000404f07604456d51f3a908efe15e6dfb8960b4a0473d51dcb74cb420baef63cd35cb6c4c928c2bd06bc591e0781d3339bc5baf54bb11341488745d70cddda568b0b",
      "kind": "record",
      "mid": "punkti/0.6:9e8260ebc8e4273936820e88d83bfe1a36b8266d2b4ee9698ad698daef911811",
      "name": "empty-record-unsupported-alg",
      "sha256": "0a3f1915298903311b61f84e41bf7faa38316c57bcdd323414b8fa9f571a976c"
    }
  ],
  "id": "V10"
}
```

## V11

```json
{
  "cases": [
    {
      "expected": {
        "admitted": 0,
        "candidate_count": 0,
        "error_class": "retryable",
        "reason": "non-conforming geographical key",
        "state": "NON-CONFORMING-publication"
      },
      "mutation": "index-key",
      "name": "tilde-index-unrestricted",
      "scope": {}
    },
    {
      "expected": {
        "admitted": 0,
        "candidate_count": 0,
        "error_class": "retryable",
        "reason": "non-conforming geographical key",
        "state": "NON-CONFORMING-publication"
      },
      "mutation": "index-key",
      "name": "tilde-index-scoped",
      "scope": {
        "xy_hi": "u5",
        "xy_lo": "u4"
      }
    },
    {
      "expected": {
        "admitted": 0,
        "candidate_count": 0,
        "error_class": "retryable",
        "reason": "non-conforming geographical key",
        "state": "NON-CONFORMING-publication"
      },
      "mutation": "shard-prefix",
      "name": "tilde-shard",
      "scope": {}
    },
    {
      "expected": {
        "admitted": 0,
        "candidate_count": 0,
        "error_class": "retryable",
        "reason": "non-conforming geographical key",
        "state": "NON-CONFORMING-publication"
      },
      "mutation": "child-key",
      "name": "tilde-child-before-pruning",
      "scope": {
        "xy_hi": "u5",
        "xy_lo": "u4"
      }
    }
  ],
  "id": "V11"
}
```

## V12

```json
{
  "cases": [
    {
      "expected": "VALID",
      "hex": "000000610000000a70756e6b74692f302e36000000013100000006303030305f3000000009666978747572652d31000000076564323535313900000020ef3dfc2a574748f04377ff1abb00c43c897e4d64058de1ba690b3895fbb34007000000000000000000000040f59a5d58a4b8738528b7527d4242d8817665291f3a3e83fcaa78c41d7ccae2fb62d6e3350038449e32d3a835d1d64f9229be2cab84e0ccd01bb5a710ae5f2e00",
      "kind": "record",
      "mid": "punkti/0.6:f67b1838cf3c3314b097f04a26c0476293c194a236349a74ca17779c2a57819f",
      "name": "minimum-length",
      "sha256": "660a3fc03d6310ec6a74cc9d14e98c6fb59f2944a48312a38811e90893df5948"
    },
    {
      "expected": "VALID",
      "hex": "0000007b0000000a70756e6b74692f302e36000000013100000020753362755f31313131313131313131313131313131313131313131313131313100000009666978747572652d31000000076564323535313900000020ef3dfc2a574748f04377ff1abb00c43c897e4d64058de1ba690b3895fbb34007000000000000000000000040d22355d47b8c178db073e70785873c797bf52bf9f7c4869236a1a9cb17811c4d8b1ef252ef0ee1a68cece3ba365d3ac7f2903c6496f462c4a279785e342f270f",
      "kind": "record",
      "mid": "punkti/0.6:4fa43ad64929e96a3cf81038b5dd911b6fa15e843a4b8dc1d61043f083acda3f",
      "name": "maximum-length",
      "sha256": "4b58cb0c8121d560d197ba0f513f42b829c0e634b3ebb25260c005b7314fb7a4"
    },
    {
      "expected": "VALID",
      "hex": "000000690000000a70756e6b74692f302e3600000001310000000e75336275747a376b397137765f3000000009666978747572652d31000000076564323535313900000020ef3dfc2a574748f04377ff1abb00c43c897e4d64058de1ba690b3895fbb34007000000000000000000000040c38f67699b5dace75cab32fa2acca9d5bb8e22ec1080c314463303649e71bcb60939d3445115097b062def81ade701578056e56c0fe0122ca91e17ae5f6c0e05",
      "kind": "record",
      "mid": "punkti/0.6:42f3d6d7ee0fd2e6563cbec1d171940fa6f506db48019627f1033252858f0a66",
      "name": "maximum-xy",
      "sha256": "94ea837c6e097aa81704d26f62bfb6fa860c03012078bb7e08a03a671797c6c6"
    },
    {
      "expected": "UNVERIFIABLE-alg",
      "hex": "0000006b0000000a70756e6b74692f302e3600000001310000000d75336275747a376b39715f323300000009666978747572652d310000000a6675747572652d616c6700000020ef3dfc2a574748f04377ff1abb00c43c897e4d64058de1ba690b3895fbb340070000000000000000000000401fb627075da4f41b8d7cd2f31b8e3a52622c7dda7091ab18a337aee86d0be3d99d6aa7cb89c2490bcfcbdb39732cee8110f085ed7eb96f9474e8c06b7bc40a0c",
      "kind": "record",
      "mid": "punkti/0.6:2c1eaad50738b7d0a6a1326f4cd32c0aba9c6df9c843ea4d07339c1fd5188ce7",
      "name": "anchored-unsupported-alg",
      "sha256": "c4d2b5a8d9fd40185417dfdd525b9504dd58bc5a268a14cf617fea65d0393ddd"
    }
  ],
  "id": "V12"
}
```
