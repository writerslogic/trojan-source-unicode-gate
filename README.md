# Trojan Source & Hidden Unicode Gate

Scans every pull request diff for the Unicode attacks that code review cannot see, and fails the check when one is added:

- **Trojan Source** bidirectional overrides and isolates (CVE-2021-42574) that make code execute differently from how it reads
- **Invisible characters**: zero-width spaces and joiners, tag characters, variation selectors, and other default-ignorable code points used to hide prompt injections and payloads
- **Homoglyphs and confusable identifiers** (CVE-2021-42694): a Cyrillic `а` in place of Latin `a`, or two identifiers that render identically
- **Mixed-script identifiers**, unexpected control characters, and normalization tricks

19 deterministic rules (USC001-USC019), so no model decides the verdict. $0.25 per PR.

Every result is a signed provenance envelope (a W3C Verifiable Credential from `did:web:agents.writerslogic.com`), saved as the `report` output. Anyone can verify it for free at `POST https://agents.writerslogic.com/api/v1/verify-provenance`, which makes it audit evidence that the check ran on this exact diff.

```yaml
on: pull_request
jobs:
  unicode:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }
      - uses: writerslogic/trojan-source-unicode-gate@v1
        with:
          wallet-private-key: ${{ secrets.WL_WALLET_KEY }}   # USDC on Base; x402 pays per call, no gas
          fail-on: block                                     # block | warn | never
```

Payment: either `wallet-private-key` (a dedicated wallet holding a little USDC on Base; each call is an x402 `exact` authorization capped by `max-usdc-per-call`) or `api-key` (a hub API key). Store both as secrets.

Want an AI exploit review in the same check? Use [Unicode Safety & Signed PR Review](https://github.com/writerslogic/pre-merge-review-action), which bundles this gate with the exploit gatekeeper.

Outputs: `merge-approved` and `report`. The job summary lists each rule hit with file and line.
