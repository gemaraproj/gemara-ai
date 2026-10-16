---
name: gemara-evidence-digest-check
description: Use when a Gemara evaluation or audit cites an SBOM, a TEA artifact or an in-toto attestation as evidence for an artifact. Checks that the evidence names the artifact's own digest and that the artifact in hand has that digest, before the evidence is relied on.
---

# Evidence Digest Check

Evidence about an artifact only supports a Gemara assessment if it covers the exact bytes being assessed. A document that names the component and version but no digest, or names a digest the artifact does not have, says nothing checkable about the artifact in hand.

## Tools and Resources

| Tool / Resource | Purpose |
|----------------|---------|
| `uvx agent-evidence-admission==0.1.0 digest-check <document> <artifact>` | Recompute the artifact's digest and compare it with the digest the document names for its own subject |
| `gemara://lexicon` | Term definitions for the Gemara security model |

`<document>` and `<artifact>` are local paths or https URLs. The document can be a CycloneDX SBOM (JSON or XML; the `metadata.component` hashes are read), a TEA artifact (the `checksums` of its formats), or an in-toto statement, bare or in a DSSE envelope (its `subject` digests). Digests listed for dependencies or materials do not count.

## Flow

1. For each piece of evidence the assessment cites, find the artifact it is about and where the artifact can be fetched.
2. Run the check:

   ```bash
   uvx agent-evidence-admission==0.1.0 digest-check sbom.cdx.json component-1.2.3.tar.gz
   ```

3. Read the result:

   | Exit | Output | Meaning for the assessment |
   |------|--------|----------------------------|
   | 0 | `PASS ... has sha256:...` | The evidence covers these bytes. Record the digest with the evidence reference. |
   | 1 | `FAIL digest-mismatch` | The evidence describes different bytes. Do not use it for this artifact. |
   | 1 | `FAIL no-digest-named` | The evidence names no digest for its subject, so it cannot be tied to these bytes. Treat it as unverified. |
   | 2 | `ERROR could not read` | The check did not run. Report that; never record it as a pass or a fail. |

4. When writing the assessment or evaluation log, cite each piece of evidence together with the artifact digest the check matched, so a later reader can repeat the check.

## Notes

- The check does not verify signatures or the truth of the document's claims. It establishes only that the evidence and the artifact refer to the same bytes.
- Plain http URLs are refused; fetch evidence over https.
