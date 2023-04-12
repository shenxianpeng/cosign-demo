# cosign-demo

[Cosign docs](https://docs.sigstore.dev/cosign/overview/)

## prepare demo image

```bash
docker build -t cosign-demo .
docker login
docker tag cosign-demo:latest xianpengshen/cosign-demo:latest
docker push xianpengshen/cosign-demo:latest
```

## generating keys

```bash
$ cosign generate-key-pair
Enter password for private key: 
Enter password for private key again: 
Private key written to cosign.key
Public key written to cosign.pub
```

## sign with a local key pair

```bash
$ cosign sign --key cosign.key xianpengshen/cosign-demo:latest
Enter password for private key: 
WARNING: Image reference xianpengshen/cosign-demo:latest uses a tag, not a digest, to identify the image to sign.
    This can lead you to sign a different image than the intended one. Please use a
    digest (example.com/ubuntu@sha256:abc123...) rather than tag
    (example.com/ubuntu:latest) for the input to cosign. The ability to refer to
    images by tag will be removed in a future release.


        The sigstore service, hosted by sigstore a Series of LF Projects, LLC, is provided pursuant to the Hosted Project Tools Terms of Use, available at https://lfprojects.org/policies/hosted-project-tools-terms-of-use/.
        Note that if your submission includes personal data associated with this signed artifact, it will be part of an immutable record.
        This may include the email address associated with the account with which you authenticate your contractual Agreement.
        This information will be used for signing this artifact and will be stored in public transparency logs and cannot be removed later, and is subject to the Immutable Record notice at https://lfprojects.org/policies/hosted-project-tools-immutable-records/.

By typing 'y', you attest that (1) you are not submitting the personal data of any other person; and (2) you understand and agree to the statement and the Agreement terms at the URLs listed above.
Are you sure you would like to continue? [y/N] y
tlog entry created with index: 17742600
Pushing signature to: index.docker.io/xianpengshen/cosign-demo
```

## json payload is printed to stdout

```bash
$ cosign generate xianpengshen/cosign-demo:latest
{"critical":{"identity":{"docker-reference":"index.docker.io/xianpengshen/cosign-demo"},"image":{"docker-manifest-digest":"sha256:d61c17b6aa49def5e3a654258592829a7fdec3387436c4f93cd0dcfb4c7f01e6"},"type":"cosign container image signature"},"optional":null}
```

```bash
$ cosign triangulate xianpengshen/cosign-demo:latest
index.docker.io/xianpengshen/cosign-demo:sha256-d61c17b6aa49def5e3a654258592829a7fdec3387436c4f93cd0dcfb4c7f01e6.sig
```

## sign blobs

```bash
#  install cosign
$ go install github.com/sigstore/cosign/v2/cmd/cosign@latest

$ echo "my first artifact" > artifact

cosign upload blob -f artifact xianpengshen/cosign-demo:latest
Uploading file from [artifact] to [index.docker.io/xianpengshen/cosign-demo:latest] with media type [text/plain]
File [artifact] is available directly at [index.docker.io/v2/xianpengshen/cosign-demo/blobs/sha256:c69d72c98b55258f9026f984e4656f0e9fd3ef024ea3fac1d7e5c7e6249f1626]
Uploaded image to:
index.docker.io/xianpengshen/cosign-demo@sha256:11ec09099cdc5e0460148b4eb35b46c0577b649be4485e3f4d447cc2c91cde48

$ curl -L index.docker.io/v2/xianpengshen/cosign-demo/blobs/sha256:c69d72c98b55258f9026f984e4656f0e9fd3ef024ea3fac1d7e5c7e6249f1626 | jq .
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100   167  100   167    0     0    426      0 --:--:-- --:--:-- --:--:--   426
{
  "errors": [
    {
      "code": "UNAUTHORIZED",
      "message": "authentication required",
      "detail": [
        {
          "Type": "repository",
          "Class": "",
          "Name": "xianpengshen/cosign-demo",
          "Action": "pull"
        }
      ]
    }
  ]
}
```