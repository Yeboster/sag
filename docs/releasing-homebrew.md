# sag Homebrew Release Playbook

Lightweight flow to ship sag via the `steipete/tap` Homebrew tap (source build).

## 0) Prereqs
- macOS with Homebrew installed.
- Clean git tree on `main`.
- Go toolchain installed (Go version from `go.mod`).
- Access to tap repo sibling at `../homebrew-tap`.

## 1) Verify build is green
```sh
pnpm format
pnpm lint
pnpm test
pnpm build
```

## 2) Bump the version in code
- Update `Version` in `cmd/root.go`.
- Update `CHANGELOG.md` entry to match.

## 3) Tag & push
```sh
git commit -am "release: vX.Y.Z"
git tag vX.Y.Z
git push origin main --tags
```

## 4) Update the Homebrew tap formula
In `../homebrew-tap/Formula/sag.rb`:

1. Set `version "X.Y.Z"`.
2. Set `url` to the tag source tarball:
   ```
   url "https://github.com/steipete/sag/archive/refs/tags/vX.Y.Z.tar.gz"
   ```
3. Update `sha256` for that tarball:
   ```sh
   curl -L -o /tmp/sag.tar.gz https://github.com/steipete/sag/archive/refs/tags/vX.Y.Z.tar.gz
   shasum -a 256 /tmp/sag.tar.gz
   ```
4. Ensure build step uses:
   ```ruby
   system "go", "build", *std_go_args(ldflags: "-s -w"), "./cmd/sag"
   ```

Commit/push in tap repo:
```sh
git commit -am "sag vX.Y.Z"
git push origin main
```

## 5) Sanity-check install from tap
```sh
brew uninstall sag || true
brew untap steipete/tap || true
brew tap steipete/tap
brew install steipete/tap/sag
brew test steipete/tap/sag
sag --version
```

## Notes
- Formula builds from source; no binary assets required.
- Keep formula minimal: version, url, sha256, license, `go` build dep, `std_go_args`.
