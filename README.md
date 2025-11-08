# go-workflows

Collection of opinionated reusable Github Actions workflows for Go.

The workflows are optimized for execution speed and many are designed with supply chain security in mind. They use [setup-go-faster](https://github.com/WillAbides/setup-go-faster) to install Go, and [cache-go](https://github.com/capnspacehook/cache-go) to cache Go modules and build artifacts separately. I use them to in both projects that are primarily CLI tools, and libraries used as a Go module.

The Go version used in the workflows is the Go version in `go.mod`. I find it helpful for CLI tool projects as
updating the Go version in `go.mod` means the same version of Go will be used locally and in CI. Likewise for
libraries linting and testing against the version in `go.mod` is useful, it will let you know if you are using
an outdated version of Go, and you're not releasing binaries so you don't need to use the latest version of Go.

These workflows are used in my projects and may be useful to others, though I'm aware that they're likely not
customizable enough/too opinionated for many projects. If you'd like to use them but need more customization, feel
free to open an issue or a PR that adds workflow inputs.

## Workflows:

### [lint-go](https://github.com/capnspacehook/go-workflows/blob/master/.github/workflows/lint-go.yml)

Run the latest versions of [`golangci-lint`](https://golangci-lint.run/) and [`staticcheck`](https://staticcheck.io/), and checks that `go.mod` is tidy.

<details>
    <summary>Example usage:</summary>

```yaml
name: Lint Go

on:
  push:
    branches:
      - master
    paths:
      - "**.go"
      - "go.mod"
      - "go.sum"
      - ".github/workflows/lint-go.yml"
  pull_request:
    branches:
      - "*"
    paths:
      - "**.go"
      - "go.mod"
      - "go.sum"
      - ".github/workflows/lint-go.yml"

  workflow_dispatch: {}

jobs:
  lint-go:
    permissions:
      contents: read
    uses: capnspacehook/go-workflows/.github/workflows/lint-go.yml@master
```
</details>

### [test](https://github.com/capnspacehook/go-workflows/blob/master/.github/workflows/test.yml)

Runs `go test` with the race detector, and optionally runs fuzz tests as well.

#### Inputs

| Name | Description | Required | Default |
| --- | --- | --- | --- |
| `run-fuzz-tests` | Run fuzz tests recursively | false | true |
| `fuzz-with-race` | Run fuzz tests with race detector | false | false |

<details>
    <summary>Example usage:</summary>

```yaml
name: Test

on:
  push:
    branches:
      - master
    paths:
      - "**.go"
      - "go.mod"
      - "go.sum"
      - ".github/workflows/test.yml"
  pull_request:
    branches:
      - "*"
    paths:
      - "**.go"
      - "go.mod"
      - "go.sum"
      - ".github/workflows/test.yml"

  workflow_dispatch: {}

jobs:
  test:
    permissions:
      contents: read
    uses: capnspacehook/go-workflows/.github/workflows/test.yml@master
```
</details>

### [check-generated](https://github.com/capnspacehook/go-workflows/blob/master/.github/workflows/check-generated.yml)

Checks that running `go generate` does not change any files.

<details>
    <summary>Example usage:</summary>

```yaml
name: Check generated files

on:
  push:
    branches:
      - master
    paths:
      - "**.go"
      - "go.mod"
      - "go.sum"
      - ".github/workflows/check-generated.yml"
  pull_request:
    branches:
      - "*"
    paths:
      - "**.go"
      - "go.mod"
      - "go.sum"
      - ".github/workflows/check-generated.yml"

  workflow_dispatch: {}

jobs:
  check-generated:
    permissions:
      contents: read
    uses: capnspacehook/go-workflows/.github/workflows/check-generated.yml@master
```
</details>

### [vuln](https://github.com/capnspacehook/go-workflows/blob/master/.github/workflows/vuln.yml)

Scans Go dependencies for known security vulnerabilities using [`govulncheck`](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck).

<details>
    <summary>Example usage:</summary>

```yaml
name: Vulnerability scan

on:
  push:
    branches:
      - master
    paths:
      - "go.mod"
      - "go.sum"
      - "**.go"
      - ".github/workflows/vuln.yml"
  pull_request:
    branches:
      - "*"
    paths:
      - "go.mod"
      - "go.sum"
      - "**.go"
      - ".github/workflows/vuln.yml"

  workflow_dispatch: {}

jobs:
  vuln:
    permissions:
      contents: read
    uses: capnspacehook/go-workflows/.github/workflows/vuln.yml@master
```
</details>

### [release-binaries](https://github.com/capnspacehook/go-workflows/blob/master/.github/workflows/release-binaries.yml)

Builds and releases Go binaries for multiple platforms using [`GoReleaser`](https://goreleaser.com/) when a tag is pushed. Also signs the binaries with [`cosign`](https://github.com/sigstore/cosign).

<details>
    <summary>Example usage:</summary>

```yaml
name: Release binaries

on:
  push:
    tags:
      - "v*"

jobs:
  release-binaries:
    permissions:
      id-token: write
      contents: write
    uses: capnspacehook/go-workflows/.github/workflows/release-binaries.yml@master
```
</details>

### [reproduce-binary](https://github.com/capnspacehook/go-workflows/blob/master/.github/workflows/reproduce-binary.yml)

Checks if binaries built by [`GoReleaser`](https://goreleaser.com/) are reproducible using [`gorepro`](https://github.com/capnspacehook/gorepro).

#### Inputs

| Name | Description | Required | Default |
| --- | --- | --- | --- |
| `extra-build-flags` | Build flags that aren't detected by gorepro | false | ''|

<details>
    <summary>Example usage:</summary>

```yaml
name: Reproduce binary

on:
  push:
    branches:
      - master
    paths:
      - "**.go"
      - "go.mod"
      - "go.sum"
      - ".goreleaser.yml"
      - ".github/workflows/reproduce-binary.yml"
  pull_request:
    branches:
      - "*"
    paths:
      - "**.go"
      - "go.mod"
      - "go.sum"
      - ".goreleaser.yml"
      - ".github/workflows/reproduce-binary.yml"

  workflow_dispatch: {}

jobs:
  reproduce-binary:
    permissions:
      contents: read
    uses: capnspacehook/go-workflows/.github/workflows/reproduce-binary.yml@master
```
</details>

### [update-go-version](https://github.com/capnspacehook/go-workflows/blob/master/.github/workflows/update-go-version.yml)

Creates a pull request to update the Go version in `go.mod` to the latest stable version.

<details>
    <summary>Example usage:</summary>

```yaml
name: Update Go version

on:
  workflow_dispatch: {}

  schedule:
    - cron: "0 0 * * *"

jobs:
  update-go-toolchain:
    permissions:
      contents: write
      pull-requests: write
    uses: capnspacehook/go-workflows/.github/workflows/update-go-toolchain.yml@master
```
</details>

### [lint-docker](https://github.com/capnspacehook/go-workflows/blob/master/.github/workflows/lint-docker.yml)

Lints Dockerfiles using [hadolint](https://github.com/hadolint/hadolint) and verifies that Docker images build successfully.

<details>
    <summary>Example usage:</summary>

```yaml
name: Lint Docker

on:
  push:
    branches:
      - master
    paths:
      - "**Dockerfile*"
      - ".github/workflows/lint-docker.yml"
  pull_request:
    branches:
      - "*"
    paths:
      - "**Dockerfile*"
      - ".github/workflows/lint-docker.yml"

  workflow_dispatch: {}

jobs:
  lint-docker:
    permissions:
      contents: read
    uses: capnspacehook/go-workflows/.github/workflows/lint-docker.yml@master
```
</details>

### [release-image](https://github.com/capnspacehook/go-workflows/blob/master/.github/workflows/release-image.yml)

Builds and pushes Docker images to GitHub Container Registry (GHCR), with semantic versioning for tags and branch/sha tags for other refs. Also signs the images with [`cosign`](https://github.com/sigstore/cosign).

<details>
    <summary>Example usage:</summary>

```yaml
name: Release image

on:
  push:
    branches:
      - master
    tags:
      - "v*"

  workflow_dispatch: {}

jobs:
  release-image:
    permissions:
      contents: read
      id-token: write
      packages: write
    uses: capnspacehook/go-workflows/.github/workflows/release-image.yml@master
```
</details>

### [lint-actions](https://github.com/capnspacehook/go-workflows/blob/master/.github/workflows/lint-actions.yml)

Lints GitHub Actions workflow files using [`actionlint`](https://github.com/rhysd/actionlint).

<details>
    <summary>Example usage:</summary>

```yaml
name: Lint workflows

on:
  push:
    branches:
      - master
    paths:
      - ".github/workflows/*"
  pull_request:
    branches:
      - "*"
    paths:
      - ".github/workflows/*"

  workflow_dispatch: {}

jobs:
  lint-actions:
    permissions:
      contents: read
    uses: capnspacehook/go-workflows/.github/workflows/lint-actions.yml@master
```
</details>

### [codeql](https://github.com/capnspacehook/go-workflows/blob/master/.github/workflows/codeql.yml)

Runs Github CodeQL analysis on Go code to detect security vulnerabilities and code quality issues. Skips execution for private repositories.

<details>
    <summary>Example usage:</summary>

```yaml
name: Run CodeQL

on:
  push:
    branches:
      - master
  pull_request:
    branches:
      - master

jobs:
  codeql:
    permissions:
      actions: write
      contents: read
      security-events: write
    uses: capnspacehook/go-workflows/.github/workflows/codeql.yml@master
```
</details>

## Verifying release artifacts

Go binaries and Docker images are both signed with [`cosign`](https://github.com/sigstore/cosign). Their signatures can can be verified and the binaries can be reproduced to verify that they were built from unmodified source code, by Github Actions.

### Verifying Docker images

Simply check the signature of the image with `cosign`:

```bash
cosign verify ghcr.io/<user>/<repo>:<version> | jq
```

You can verify the image was built by Github Actions by inspecting the Issuer and Subject fields of the output.

### Verifying binaries

Download the checksums file, certificate, signature and the archive from a Github release to the same directory.

Extract the binary from the archive, verify the checksums file and verify the contents of the binary:

```bash
tar xfs whalewall_<version>_linux_amd64.tar.gz
cosign verify-blob --certificate checksums.txt.crt --signature checksums.txt.sig checksums.txt
sha256sum -c checksums.txt
```

### Reproducing released binaries

You can also reproduce the released binaries to verify that they were built from unmodified source code. Verifying binaries requires [`gorepro`](https://github.com/capnspacehook/gorepro).

First, download the release archive and extract it. Clone the source repro and cd into it.

Install `gorepro` and run it on the extracted release binary. `gorepro` will tell you if reproducing the binary was successful. Don't worry about checking out the correct tag or commit, `gorepro` will handle that for you.

If you don't trust `gorepro` you can run it again additionally passing the -d flag. This will print the commands `gorepro` generated to reproduce the release binary. You can run the printed commands and verify for yourself that the reproduced binary is bit for bit identical to the released one.

This example reproduces [`whalewall`](https://github.com/capnspacehook/whalewall) v0.2.3:

```bash
# attempt to reproduce whalewall
wget https://github.com/capnspacehook/whalewall/releases/download/v0.2.3/whalewall_0.2.3_linux_amd64.tar.gz
tar fxs whalewall_v0.2.3_linux_amd64.tar.gz
git clone https://github.com/capnspacehook/whalewall whalewall-src
cd whalewall-src

# reproduce binary
go install github.com/capnspacehook/gorepro@latest
gorepro -b="-ldflags=-s -w -X main.version v0.2.3" ../whalewall

# reproduce by manually running commands from gorepro
BUILD_CMD="$(gorepro -d -b='-ldflags=-s -w -X main.version v0.2.3' ../whalewall)"
echo "$BUILD_CMD"
"$BUILD_CMD"
sha256sum whalewall whalewall.repro
```
