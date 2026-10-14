### 001-configurable-annotations.patch

Add ability to override all annotations via flags.

TODO: in newer versions of pod-reloader module configure to override all this annotations, needs alert or notification for existing exceptions like `reloader.stakater.com/last-reloaded-from` that clients could add.

> Upstream PR: https://github.com/stakater/Reloader/pull/1228

### 002-go-mod.patch

Bump `golang.org/x/net` to v0.61.0 and `golang.org/x/text` to v0.43.0 (the rest of the
`golang.org/x/*` family follows, as `go get` resolves it) to close the CVEs found in the
`reloader` binary: CVE-2026-78659, CVE-2026-78660, CVE-2026-78663, CVE-2026-78669,
CVE-2026-97032 (x/net, fixed in 0.60.0) and CVE-2026-56851 (x/text, fixed in 0.41.0).

Upstream v1.4.22 still pins x/net v0.56.0 and x/text v0.39.0, so there is no release to
bump to. Regenerate after every Reloader bump:

```bash
go get golang.org/x/net@<ver> golang.org/x/text@<ver> && go mod tidy
git diff -- go.mod go.sum > patches/002-go-mod.patch
```
