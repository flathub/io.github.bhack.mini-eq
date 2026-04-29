# Mini EQ Flathub Packaging

This repository contains the Flathub packaging for Mini EQ. The application
source, user documentation, issue tracker, and release notes live upstream:

https://github.com/bhack/mini-eq

## Repository Role

`io.github.bhack.mini-eq.yaml` is the Flathub publishing manifest. It should
point at an immutable upstream GitHub release archive and include the archive
SHA-256.

The upstream repository also has a manifest with the same name, but that one is
for local development and CI. It builds the checked-out source tree directly:

```yaml
- type: dir
  path: .
```

The expected Flathub source block is:

```yaml
- type: archive
  url: https://github.com/bhack/mini-eq/releases/download/vX.Y.Z/mini_eq-X.Y.Z.tar.gz
  sha256: <release archive sha256>
```

Outside that Mini EQ source block, the two manifests should normally stay in
sync.

## Updating Mini EQ

1. Publish the upstream GitHub release first. Draft release assets use temporary
   URLs and should not be copied into the Flathub manifest.
2. Compute or verify the source archive SHA-256:

   ```bash
   curl -fsSL \
     https://github.com/bhack/mini-eq/releases/download/vX.Y.Z/mini_eq-X.Y.Z.tar.gz \
     | sha256sum
   ```

3. Update only the Mini EQ source URL and SHA-256 in
   `io.github.bhack.mini-eq.yaml`.
4. Keep `python3-dependencies.yaml` unchanged unless Python dependencies changed.
   If dependencies changed, regenerate it upstream and update both repositories.
5. Validate the manifest:

   ```bash
   flatpak run --command=flatpak-builder-lint org.flatpak.Builder manifest io.github.bhack.mini-eq.yaml
   flatpak run --command=flatpak-builder org.flatpak.Builder \
     --force-clean \
     --download-only \
     --install-deps-from=flathub \
     build-dir io.github.bhack.mini-eq.yaml
   ```

6. From an upstream checkout, compare the manifests for unintended drift:

   ```bash
   python3 tools/check_flathub_manifest_drift.py
   ```

7. Open a pull request against `flathub/io.github.bhack.mini-eq:master` and wait
   for the Flathub buildbot checks.

Do not edit bundled Mini EQ application source in this repository. Fix source
code, AppStream metadata, desktop files, icons, and screenshots upstream, cut a
release, then update this manifest to the new release archive.
