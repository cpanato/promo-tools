# Creating Promotion Pull Requests

> [!TIP]
> Disclaimer: This document was originally written for Release Managers, but can be adapted to be used for any subproject.

- [Preparing Environment](#preparing-environment)
- [Promoting Images](#promoting-images)
- [Completing the Image Promotion](#completing-the-image-promotion)

When cutting a new Kubernetes release, we need to publish images to the `k8s-staging-kubernetes` GCS Bucket and then promote them to the production.

**The Image Promotion should be done after `Official Stage` is completed and there are no errors.**

## Preparing Environment

First, take the following steps to prepare your environment for promoting images:

- Install the [promotion tooling](/README.md#installation):

  ```shell
  go install sigs.k8s.io/promo-tools/v4/cmd/kpromo@latest
  ```

  > Note: If kpromo has not yet published a [minor release](https://github.com/kubernetes-sigs/promo-tools/releases) and you desire to run the actual latest version. Specify it explicitly with the full version, for example:  `go install sigs.k8s.io/promo-tools/v4/cmd/kpromo@v3.3.0-beta.3`.

- Promoting images will require a GitHub Personal Access Token in order to
  create a PR on your behalf.

  If you have not already created a token, you can do so by following these
  [instructions](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
  and using the appropriate token scopes:
  - `public_repo`

  Once you have a personal access token, make it available to your environment:

  ```shell
   export GITHUB_TOKEN="ghp-xxxxxxxxxxxxxxxxxxx"
  ```

  _Note the whitespace preceding the `export` command (which will prevent the
  token from being logged in your terminal's history.)_

## Promoting Images

The images are promoted by using the `kpromo pr` command.

```console
Usage:
  kpromo pr [flags]

Flags:
      --fork string        the user's fork of kubernetes/k8s.io
  -h, --help               help for pr
  -i, --interactive        interactive mode, asks before every step
      --project string     the name of the project to promote images for (default "kubernetes")
      --reviewers string   the list of GitHub users or teams to assign to the PR (default "@kubernetes/release-engineering")
  -t, --tag strings        version tag of the images we will promote
```

Example:

```shell
kpromo pr -i --fork=<your-github-username> --tag=v1.20.0-rc.0 --tag=v1.21.0-alpha.0
```

> [!NOTE]  
> The images that are promoted depend on the release you're cutting. In case of multiple cuts needed, due to resource restriction on Prow, please open two separate PRs (which means running two separate `kpromo` commands).

> [!CAUTION]  
> For Stable Kubernetes releases (1.x) **DO NOT** cut the next patch `1.x.1-rc.0`

- Alpha or Beta release: promote the images for the release you're cutting (e.g. `v1.20.0-beta.1`)
- The first RC (e.g. `v1.20.0-rc.0`): promote the images for the RC and for the next minor alpha release (e.g. `v1.21.0-alpha.0`)
- The subsequent RCs (e.g. `v1.20.0-rc.1`): promote the images for the RC you're cutting (e.g. `v1.20.0-rc.1`)

- A stable Kubernetes release (e.g. `v1.20.0`): only promote the images for the release you're cutting (e.g. `v1.20.0`)
- A stable release of other projects (e.g. `v1.20.0`): promote the images for the release you're cutting and for the RC of the next patch release (e.g. `v1.20.1-rc.0`)

The following steps are taken by the `kpromo pr` command:

- Clone and update your `kubernetes/k8s.io` fork
- Update the images manifest (`k8s.gcr.io/images/k8s-staging-kubernetes/images.yaml`) to add the image digests for specified releases/tags
- Create a branch and push it to your fork
- Create a PR in the `kubernetes/k8s.io` repository with an explicit `/hold`

Example PRs:

- [PR example 1](https://github.com/kubernetes/k8s.io/pull/1386)
- [PR example 2](https://github.com/kubernetes/k8s.io/pull/1348)

## Completing the Image Promotion

Once the `kpromo pr` command is done take the following steps to complete image promotion and continue the release process:

- Edit the PR description to add links for GCB jobs for `Mock Stage`, `Mock Release`, and `Official Stage` steps, or a link to the release tracking issue which includes the needed links
- Once the PR is approved by Release Managers lift the hold and proceed with the release process
- After the Pull Request is merged and **before** starting the `Official Release` step, we need to watch the following [Prow Job](https://prow.k8s.io/?job=post-k8sio-image-promo) to succeed. When the latest master ran without errors, then we can continue with `Official Release`.
