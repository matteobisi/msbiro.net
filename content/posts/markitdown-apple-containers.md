---
title: "MarkItDown: An AI-Boosting Tool Tested on Apple Containers"
date: 2025-11-04T01:30:00+01:00
tags: ["MarkItDown", "Apple", "Containers", "AI", "Podman"]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "A hands-on test of Microsoft's MarkItDown, a powerful tool for AI workflows, and a first look at Apple's new container technology on an M4 MacBook."
canonicalURL: "https://msbiro.net/posts/markitdown-apple-containers/"
disableHLJS: true
disableShare: true
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "https://www.msbiro.net/social-image.png"
    alt: "MarkItDown and Apple Containers"
    caption: ""
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

## Introduction

As everyone, we are evolving and we are including AI into several workflows, so it's essential having a way to pass data to the AI from various types of files. This is where Microsoft's [MarkItDown](https://github.com/microsoft/markitdown) comes in as a powerful tool. It's a lightweight Python utility that converts numerous file formats into Markdown, a format easily consumable by AI models. Whether you want to use it with an AI assistant like Claude through its MCP server, as a CLI tool, with Python code, or run it in a container, MarkItDown offers a lot of flexibility.

This article explores MarkItDown and my first experience with Apple's new container technology on an M4 MacBook. I'll walk you through the process of building and running a MarkItDown container, comparing the experience with Podman, and sharing my thoughts on this new containerization approach.

## What is MarkItDown?

MarkItDown is a versatile tool that can convert a plethora of file formats, including:

- PDF
- PowerPoint
- Word
- Excel
- Images (with EXIF metadata and OCR)
- Audio (with EXIF metadata and speech transcription)
- HTML
- and many more.

This makes it an invaluable asset for anyone working with AI, as it simplifies the process of preparing diverse data sources for model consumption.

## Getting Started with Apple Containers

My journey began with setting up Apple's new container technology on my M4 MacBook. The installation process is straightforward, starting with downloading the package from the official repository.

- **Apple Container repository:** [https://github.com/apple/container/](https://github.com/apple/container/)
I did this tests with the current "latest" relase: 0.6.0.
Upon the first run of `container system start`, I was prompted to download the default Linux kernel from the Kata Containers project, which is a key component of Apple's container solution.

```bash
❯ container system start
Verifying apiserver is running...
No default kernel configured.
Install the recommended default kernel from [https://github.com/kata-containers/kata-containers/releases/download/3.17.0/kata-static-3.17.0-arm64.tar.xz]? [Y/n]:
Installing kernel...
```

This marks a significant architectural shift compared to Docker Desktop and Podman on macOS, which run Linux containers inside a single shared lightweight virtual machine. In contrast, Apple Containers employs a security-first design by running each container within its own dedicated, lightweight virtual machine, each with an independent Linux kernel instance. This approach significantly improves isolation by containing potential kernel vulnerabilities within individual VMs, thereby reducing the attack surface and providing stronger security guarantees. Additionally, it offers a more native Linux experience optimized for Apple silicon, combining enhanced performance with robust container isolation.

## Building and Running the MarkItDown Container

With the environment set up, I proceeded to build the MarkItDown container image using the provided Dockerfile from the [MarkItDown repository](https://github.com/microsoft/markitdown).

The command is simple:
```bash
container build -t markitdown:latest .
```

The build process was successful, and here is a snippet of the output:
```bash
❯ container build -t markitdown:latest .
[+] Building 68.0s (11/11) FINISHED
 => [resolver] fetching image...docker.io/library/python:3.13-slim-bullseye                                                                                        0.0s
 => [internal] load .dockerignore                                                                                                                                  0.1s
 => => transferring context: 51B                                                                                                                                   0.0s
 => oci-layout://docker.io/library/python:3.13-slim-bullseye@sha256:e98b521460ee75bca92175c16247bdf7275637a8faaeb2bcfa19d879ae5c4b9a                               0.8s
 => => resolve docker.io/library/python:3.13-slim-bullseye@sha256:e98b521460ee75bca92175c16247bdf7275637a8faaeb2bcfa19d879ae5c4b9a                                 0.1s
 => => sha256:c2a297c67e79845c78ac7e44c42637dffd664b856396a7557b7674d006236421 250B / 250B                                                                         0.0s
 => => sha256:428dc70e028d9ec43c6f1a0344141dfd09d36e58f19ba57957eaafa9e3622f2a 1.07MB / 1.07MB                                                                     0.0s
 => => sha256:003c2c873a951e19476da4ce58c3cfd40117ca97c2f3290472eb24b39d57cfeb 12.65MB / 12.65MB                                                                   0.1s
 => => sha256:c9dcb8c12911e83d609f6f21bfb1cb64bd506f6a38c2eceb94fd680d4fb376cd 28.74MB / 28.74MB                                                                   0.2s
 => => extracting sha256:c9dcb8c12911e83d609f6f21bfb1cb64bd506f6a38c2eceb94fd680d4fb376cd                                                                          0.3s
 => => extracting sha256:428dc70e028d9ec43c6f1a0344141dfd09d36e58f19ba57957eaafa9e3622f2a                                                                          0.0s
 => => extracting sha256:003c2c873a951e19476da4ce58c3cfd40117ca97c2f3290472eb24b39d57cfeb                                                                          0.1s
 => => extracting sha256:c2a297c67e79845c78ac7e44c42637dffd664b856396a7557b7674d006236421                                                                          0.0s
 => [internal] load build context                                                                                                                                  0.2s
 => => transferring context: 8.48MB                                                                                                                                0.2s
 => [linux/arm64 1/7] RUN apt-get update && apt-get install -y --no-install-recommends     ffmpeg     exiftool                                                    27.1s
 => [linux/arm64 2/7] RUN if [ "false" = "true" ]; then     apt-get install -y --no-install-recommends     git;     fi                                             0.1s
 => [linux/arm64 3/7] RUN rm -rf /var/lib/apt/lists/*                                                                                                              0.0s
 => [linux/arm64/v8 4/7] WORKDIR /app                                                                                                                              0.0s
 => [linux/arm64/v8 5/7] COPY . /app                                                                                                                               0.0s
 => [linux/arm64 6/7] RUN pip --no-cache-dir install     /app/packages/markitdown[all]     /app/packages/markitdown-sample-plugin                                 25.9s
 => exporting to oci image format                                                                                                                                  8.5s
 => => exporting layers                                                                                                                                            6.7s
 => => exporting manifest sha256:e4f86b1c307a048bfe23e7b9f997e313aae69df25e91d40b6a91a1848032ec5c                                                                  0.0s
 => => exporting config sha256:c5f31ce49ace05b613a236ad71a7742e72888fabd83b68072b5b9e05fca88399                                                                    0.0s
 => => exporting manifest list sha256:614e06454ec35995a9223782fb41367b6df274451e266df3b19a6ca0078bd07c                                                             0.0s
 => => sending tarball                                                                                                                                             1.8s
Successfully built markitdown:latest
```

Next, I tested the container by converting a PowerPoint presentation to Markdown:

```bash
container run --rm -i markitdown:latest < ~/presentation.pptx > presentation.md
```

I received a warning (`onnxruntime cpuid_info warning: Unknown CPU vendor. cpuinfo_vendor value: 0`), but the conversion worked perfectly, and the `presentation.md` file was created as expected.

## Comparison with Podman

To get a better sense of Apple Containers, I performed the same steps with Podman. The build process was similar, and I encountered the same `onnxruntime` warning. The final Markdown output was identical.

However, a notable difference was the reported image size.

**Podman:**
```bash
❯ podman images
REPOSITORY                     TAG                 IMAGE ID      CREATED        SIZE
localhost/markitdown           latest              597b85b7c566  6 seconds ago  930 MB
docker.io/library/python       3.13-slim-bullseye  b2b87c0f2bbe  2 months ago   119 MB
```

**Apple Containers:**
```bash
❯ container image list -v
NAME        TAG                 INDEX DIGEST                 OS     ARCH   VARIANT  SIZE      CREATED               MANIFEST DIGEST
python      3.13-slim-bullseye  e98b521460ee75bca92175c1...  linux  arm64  v8       42.5 MB   2025-08-06T21:20:23Z  0eca45d3951845d038115591...
markitdown  latest              614e06454ec35995a9223782...  linux  arm64           356.7 MB  2025-08-06T21:20:23Z  e4f86b1c307a048bfe23e7b9...
```

The size difference is because Apple Containers reports the compressed size of the image, while Podman shows the uncompressed size. This is an important distinction to keep in mind when comparing the two.

## OCI Compliance and Interoperability

A key feature of Apple Containers is that it builds OCI-compliant images. This means that an image built with Apple Containers can be used by other container runtimes like Podman or Docker. I tested this by tagging and pushing the `markitdown` image to my company's registry:

```bash
container registry login registry.sighup.io --username matteo.bisi@reevo.it
container image tag markitdown:latest registry.sighup.io/devsecops/tools/markitdown:0.1.3
container image push registry.sighup.io/devsecops/tools/markitdown:0.1.3
```

Then, I pulled and ran the image with Podman:

```bash
podman run --rm -i registry.sighup.io/devsecops/tools/markitdown:0.1.3 < presentation.pptx > presentation-podman-appleimage.md
```

It worked flawlessly, producing the exact same output. This demonstrates the interoperability of Apple Containers and its adherence to open standards.

## Conclusion

MarkItDown is an incredibly useful tool for anyone working with AI, and Apple's new container technology provides a promising new way to work with containers on macOS. While it hasn't reached version 1.0 yet, its native performance, OCI compliance, and ease of use make it a compelling alternative to traditional container runtimes on Mac. I'm excited to see how Apple Containers evolves and what new capabilities it will bring to the developer community.
