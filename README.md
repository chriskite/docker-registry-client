我是光年实验室高级招聘经理。
我在github上访问了你的开源项目，你的代码超赞。你最近有没有在看工作机会，我们在招软件开发工程师，拉钩和BOSS等招聘网站也发布了相关岗位，有公司和职位的详细信息。
我们公司在杭州，业务主要做流量增长，是很多大型互联网公司的流量顾问。公司弹性工作制，福利齐全，发展潜力大，良好的办公环境和学习氛围。
公司官网是http://www.gnlab.com,公司地址是杭州市西湖区古墩路紫金广场B座，若你感兴趣，欢迎与我联系，
电话是0571-88839161，手机号：18668131388，微信号：echo 'bGhsaGxoMTEyNAo='|base64 -D ,静待佳音。如有打扰，还请见谅，祝生活愉快工作顺利。

# Docker Registry Client

An API client for the [V2 Docker Registry
API](http://docs.docker.com/registry/spec/api/), for Go applications.

## Imports

```go
import (
    "github.com/heroku/docker-registry-client/registry"
    "github.com/docker/distribution/digest"
    "github.com/docker/distribution/manifest"
    "github.com/docker/libtrust"
)
```

## Creating A Client

```go
url      := "https://registry-1.docker.io/"
username := "" // anonymous
password := "" // anonymous
hub, err := registry.New(url, username, password)
```

Creating a registry will also ping it to verify that it supports the registry
API, which may fail. Failures return non-`nil` err values.

Authentication supports both HTTP Basic authentication and OAuth2 token
negotiation.

## Listing Repositories

```go
repositories, err := hub.Repositories()
```

The repositories will be returned as a slice of `string`s.

## Listing Tags

Each Docker repository has a set of tags -- named images that can be downloaded.

```go
tags, err := hub.Tags("heroku/cedar")
```

The tags will be returned as a slice of `string`s.

## Downloading Manifests

Each tag has a corresponding manifest, which lists the layers and image
configuration for that tag.

```go
manifest, err := hub.Manifest("heroku/cedar", "14")
```

Schema V2

```go
manifest, err := hub.ManifestV2("heroku/cedar", "14")
```

The returned manifest will be a `manifest.SignedManifest` pointer. For details,
see the `github.com/docker/distribution/manifest` library.

## Retrieving Manifest Digest

A manifest is identified by a digest.

```go
digest, err := hub.ManifestDigest("heroku/cedar", "14")
```

The returned digest will be a `digest.Digest`. See `github.com/docker/distribution/digest`.

## Deleting Manifest

To delete a manifest

```go
digest, err := hub.ManifestDigest("heroku/cedar", "14")
err = hub.DeleteManifest("heroku/cedar", digest)
```

Please notice that, as specified by the Registry v2 API, this call doesn't actually remove the fs layers used by the image.

## Downloading Layers

Each manifest contains a list of layers, filesystem images that Docker will
compose to create containers.

```go
// or obtain the digest from an existing manifest's FSLayer list
digest := digest.NewDigestFromHex(
    "sha256",
    "a3ed95caeb02ffe68cdd9fd84406680ae93d633cb16422d00e8a7c22955b46d4",
)
reader, err := hub.DownloadLayer("heroku/cedar", digest)
)
if reader != nil {
    defer reader.Close()
}
if err != nil {
    return err
}
```

## Uploading Layers

This library can also publish new layers:

```go
digest := digest.NewDigestFromHex(
    "sha256",
    "a3ed95caeb02ffe68cdd9fd84406680ae93d633cb16422d00e8a7c22955b46d4",
)
exists, err := hub.HasLayer("example/repo", digest)
)
if err != nil {
    // …
}
if !exists {
    stream := …;
    hub.UploadLayer("example/repo", digest, stream)
}
```

## Uploading Manifests

First, create a signed manifest:

```go
manifest := &manifest.Manifest{
    Versioned: manifest.Versioned{
        SchemaVersion: 1,
    },
    Tag: "latest",
    // …
}

key, err := libtrust.GenerateECP256PrivateKey()
if err != nil {
    // …
}

signedManifest := manifest.Sign(manifest, key)
if err != nil {
    // …
}
```

Production applications should probably reuse keys, rather than generating
ephemeral keys. See the libtrust documentation for details.

Then, upload the signed manifest:

```go
err := hub.PutManifest("example/repo", "latest", signedManifest)
if err != nil {
    // …
}
```

This will also create or update tags, as necessary.
