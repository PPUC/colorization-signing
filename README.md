# Colorization signing

Certificates for the signing keys of colorization authors. PPUC Colorizing DMD signs the
`.cROMc` it exports with the author's key; libserum checks the signature offline and shows
players a notice when a file was changed after it was signed. A certificate says whose key it
is: it binds the key to the GitHub account that asked for it.

## How a certificate is issued

1. The author makes a key in PPUC Colorizing DMD (`Settings -> Signing`) and clicks
   `Request Certificate`. The editor opens an issue here, as the author, with the public key
   (the form in `.github/ISSUE_TEMPLATE/certificate.yml` does the same by hand).
2. The workflow `Certify` takes every new issue titled `Certificate request`, labels it, finds
   the key and waits for approval in the `signing` environment. A member can run a request
   again by adding the label `certificate request`.
3. A trusted member checks that the account belongs to an author and approves the run.
4. The workflow issues the certificate with the issuer key, for `github:<account ID>` with the
   account's login as the name, posts it on the issue and closes it. The editor picks it up.

A request without a key gets a comment. A rejected run leaves the issue open; close it by
hand.

## The keys

- **Root keys**: their public halves are built into libserum (`kBuiltInRootKeys` in
  `src/SerumSigning.cpp`). The secret halves are kept offline by the trusted members and
  never come here. A root key signs issuer certificates only.
- **Issuer key**: certifies authors. Its certificate, signed by a root key, is
  `issuer.certificate`; its secret half is the `ISSUER_KEY` secret of the `signing`
  environment.
- **Revoking**: a key that got out or signed what it should not have goes into
  `RevokedKeys()` in libserum's `src/SerumSigning.cpp`; from that release on, files signed
  through it show as modified. A new issuer key is made with `serum-signing keygen`,
  certified by a root key (`serum-signing certify <root-key> issuer <public-key>
  ppuc:issuer-<n> "PPUC issuing key <n>"`), and its certificate and secret replace the old
  ones here.

If GitHub cannot be used, a trusted member certifies by hand with the same tool:
`serum-signing certify <issuer-key> author <public-key> github:<id> <login> issuer.certificate`.
Existing certificates and signed files do not depend on this repository.

## Setting up the repository

- `Settings -> Environments -> New environment`: `signing`, with the trusted members as
  required reviewers and `main` as the only deployment branch. Add the secret `ISSUER_KEY`:
  the 64 hex digits of the issuer's secret key.
- The label `certificate request` (`gh label create "certificate request"`).
- Optional: the variable `LIBSERUM_REF`, the libserum commit or tag whose `serum-signing` the
  workflow builds (default `master`).
