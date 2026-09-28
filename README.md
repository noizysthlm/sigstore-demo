## App Deployment
### Set up a virtual machine
The container in [docker.io/noizysthlm/sigstore-demo](docker.io/noizysthlm/sigstore-demo) runs on amd64 linux.
- Set up a virtual machine on AWS or any other environment.
- Install Docker in rootless mode, Minikube, HELM, and kubectl if not installed by Minikube.
- Make sure that port 8080 (TCP) is accessible.
### Deploy the app
`kubectl apply -f ./<webapp-deployment.yml>` and then `kubectl apply -f ./<webapp-service.yml>`.

Port forward with `kubectl port-forward --address 0.0.0.0 service/dummy-webapp-service 8080:8080` and access the webpage should be accessible on port 8080.

## The Kyverno Policy
The policy will `fail` any deployment where the image do not meet the signature requirements.
The policy enforces signature verification with cosign where:
- The the subject is `https://github.com/noizysthlm/sigstore-demo/.github/workflows/docker-release.yml@refs/heads/main"`
-  The oicd-issuer is `https://token.actions.githubusercontent.com`

The policy can be applied by running `kubectl apply -f ./validate-container-signatures.yml`. Any new pods and deployments will fail unless they comply with the policy.

## Using GitSign
[Gitsig](https://github.com/sigstore/gitsign) is used to complete and verify commit signatures.

### Signing commits
After configuring the local Repository to use `gitsign` for signatures, we can sign our commits by including the `-S` option with the `commit` command.

### Verify a commit
`git cat-file commit HEAD` to shows the complete commit message for the `HEAD` commit. If signed, there should be a gpg signature there. Signatures recorded in Rekor can be viewed in [https://search.sigstore.dev/](https://search.sigstore.dev/).

`gitsign verify --certificate-identity=<the-signers-identity> --certificate-oidc-issuer=<the-oicd-provider> HEAD` verifies the latest commit.

## The structure of this repo

### Workflows
There are currently two workflows in this repo.

#### verify commit signatures
This workflow traverses through all commits that are to be mergerd by the PR and checks their signatures with `gitsign verify`. `git rev-list origin/${BASE_REF}..HEAD)` gives us a set of all commits reachable from `HEAD` but not `BASE_REF`. See [GitHub Context Reference](https://docs.github.com/en/actions/reference/workflows-and-actions/contexts#github-context) and [git-rev-list](https://git-scm.com/docs/git-rev-list)

#### container-build-push-sign
Builds, pushes, and signs the image to `ghcr.io`. The oicd-issuer will be `https://token.actions.githubusercontent.com` and the certificate identity: `https://github.com/USERNAME/REPOSITORY_NAME/.github/workflows/WORKFLOW_NAME@refs/heads/BRANCH_NAME` as described in [OIDC Usage in Fulcio](https://docs.sigstore.dev/certificate_authority/oidc-in-fulcio/#github).


### main branch protection
The repository have some active rules which can be viewed on [https://github.com/noizy-sthlm/sigstore-demo/rules/](https://github.com/noizy-sthlm/sigstore-demo/rules/)

Branch protection can be enabled by creating a `Ruleset` in the repository's settings
- `Require a pull request before merging` prevents pushes directly to main
	- There should be at least 1 Approver to allow merging
	- `Dismiss stale pull request approvals when new commits are pushed` to protect against ["bait and switch" attacks'](https://youtu.be/UbfhVXJn6fk?si=hP34D22sINGE8Yro)
	- `Require approval of the most recent reviewable push` to protect from reviewers mergin their own code
	- `Allowed merge methods` should be set to `merge` only to protect the commit history for future validation. Rebasing and Squashing breaks the signature validity.
- `Require linear history` could be good to ease commit history reviews but may not be possible for larger projects. 
- We set `Require status checks to pass` and add the `verify` action in the required list. This will block PRs from merging unless the action succeeds
