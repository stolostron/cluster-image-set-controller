# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Build and Development
- `make build` - Build the main binary (`bin/clusterimageset`)
- `make fmt` - Run go fmt on all code
- `make vet` - Run go vet on all code
- `make vendor` - Update vendor dependencies
- `make run` - Run the controller locally

### Testing
- `make test` - Run unit tests with coverage
- `make build-e2e` - Build end-to-end test binary
- `make test-e2e` - Run full e2e tests (requires OCM cluster)

### Docker/Container Operations
- `make docker-build` - Build container image
- `make docker-push` - Push container image
- `IMG=quay.io/stolostron/cluster-imageset-controller:tag make docker-build` - Build with custom image tag

### Running the Controller
- `./bin/clusterimageset sync --help` - Show all available options
- `./bin/clusterimageset sync --sync-interval=30` - Run with 30-second sync interval
- `./bin/clusterimageset sync --git-configmap=my-config --git-secret=my-secret` - Run with custom ConfigMap/Secret names

## Architecture

This is a Kubernetes controller that synchronizes OpenShift clusterImageSets from a Git repository to the cluster. The controller is designed for Red Hat Advanced Cluster Management (ACM) and Multicluster Engine (MCE) environments.

### Core Components

**Main Binary (`cmd/main.go`)**
- Entry point using Cobra CLI framework
- Sets up logging with zap/logr
- Delegates to the sync command in the controller package

**Controller (`pkg/controller/`)**
- `clusterimageset.go` - Main controller logic, command setup, and manager configuration
- `gitrepo.go` - Git repository interactions, authentication, and clusterImageSet parsing
- Uses controller-runtime framework for Kubernetes operations

**Git Repository Integration**
- Default repository: `https://github.com/stolostron/acm-hive-openshift-releases.git`
- Default branch: `backplane-2.9`
- Default path: `clusterImageSets`
- Default channel: `fast`
- Supports basic auth, token auth, and client certificate authentication

**Configuration**
- ConfigMap `cluster-image-set-git-repo` for repository settings (URL, branch, path, channel, TLS config)
- Secret `cluster-image-set-git-repo` for authentication (user/token or client cert/key)
- Both ConfigMap and Secret names are configurable via CLI flags

**Sync Process**
- Polls Git repository at configurable intervals (default 60 seconds)
- Checks for new commits since last sync
- Parses YAML files from the configured path/channel
- Creates/updates clusterImageSets in the cluster using Hive APIs

### Dependencies
- Uses OpenShift Hive APIs for clusterImageSet CRDs
- Built on controller-runtime for Kubernetes controller patterns
- Go-git library for Git repository operations
- Supports Kubernetes 1.27+ (see go.mod)

### Testing Structure
- Unit tests alongside source files (*_test.go)
- Integration tests in `test/integration/`
- Uses Ginkgo/Gomega testing framework
- E2E tests require an actual cluster with OCM components