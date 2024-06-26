# Docker ECR Sync

![](https://img.shields.io/pypi/v/ecr_sync.svg)
![](https://img.shields.io/pypi/l/ecr_sync.svg)
![](https://img.shields.io/pypi/pyversions/ecr_sync.svg)

Mirror public docker images to ECR, automagically. This requires [Skopeo](https://github.com/containers/skopeo) to be installed.

`pip install ecr-sync`

## Usage

```
$ ecr-sync
Usage: ecr-sync [OPTIONS] COMMAND [ARGS]...

Options:
  --registry-id, --reg TEXT       The registry ID. This is usually your AWS
                                  account ID.  [required]
  --role-name TEXT                Assume a specific role to push to AWS
  --override-os TEXT              Specify the OS of images, default to "linux"
  --override-arch [amd64|arm64|windows-amd64|all]
                                  Specify the ARCH of images, default to
                                  "amd64". If set to "all" - all architectures
                                  will be synced
  --profile-name TEXT             The name of the AWS profile to use
  --verbose                       Enable verbose output
  --dry-run                       Enable dry run
  --debug                         Enable debug output
  --public                        Use ECR Public instead of ECR
  --docker-username TEXT          The username to use for docker login
  --docker-password TEXT          The password to use for docker login
  --threads INTEGER               The number of threads to use for copying
                                  images
  --help                          Show this message and exit.
```

Create an ECR repository with the following two tags set:

* `upstream-image` set to a public Docker hub image, i.e `nginx` or `istio/proxyv2`
* `upstream-tags` set to a `/`-separated list of tag **globs**, i.e `1.6.*` or just `1.2-alpine`. ECR does not allow the
  use of the `*` character in tag values, so you should use `+` as a replacement.

Optional:
* `ignore-tags` set to a `/`-separated list of tag **globs** to ignore. ECR does not allow the
  use of the `*` character in tag values, so you should use `+` as a replacement.

Terraform example:

```hcl
resource "aws_ecr_repository" "repo" {
  name = "nginx"
  tags = {
    upstream-image = "nginx",
    // Mirror 1.16* and 1.17*
    upstream-tags = "1.16+/1.17+"
    ignore-tags = "+-gpu"
  }
}
```

Running `ecr-sync sync` will begin concurrently fetching matched images tags and pushing them to ECR.

You can run `ecr-sync list-repos` to see all repositories that will be mirrored.

You can also manually copy specific image patterns using `ecr-sync copy`:

`ecr-sync copy "istio/proxyv2:1.6.*" ACCOUNT_ID.dkr.ecr.eu-west-1.amazonaws.com/istio-proxyv2`
