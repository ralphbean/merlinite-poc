# Model Upload PoC

This is a proof of concept of using [konflux](https://konflux-ci.dev) to upload [instructlab/merlinite-7b-lab-GGUF](https://huggingface.co/instructlab/merlinite-7b-lab-GGUF) to quay.

The configuration of what to pull and upload is driven by `oci.yaml`.

The process is achieved by the tekton pipeline in `.tekton/`.

You can examine the output builds at quay.io/redhat-user-workloads/rhel-ai-poc-tenant/models/merlinite-poc:99232c2355efd09ef5d0f4aca4fe3e22a7849a92.

There are two files stored in the registry: the gguf model, and the huggingface logo. There's no point in distributing the huggingface logo like this - it's just there to demonstrate what distributing more than one file looks like.

```bash
❯ cosign tree quay.io/redhat-user-workloads/rhel-ai-poc-tenant/models/merlinite-poc:82c0d0ab382f17a8567da12f34640626d207c868
📦 Supply Chain Security Related artifacts for an image: quay.io/redhat-user-workloads/rhel-ai-poc-tenant/models/merlinite-poc:82c0d0ab382f17a8567da12f34640626d207c868
└── 💾 Attestations for an image tag: quay.io/redhat-user-workloads/rhel-ai-poc-tenant/models/merlinite-poc:sha256-e6072aa3763c0b88b33084f001cafa5ef5bac11058fd27dbca7a3e5a0bfba91f.att
   └── 🍒 sha256:7f9659bad9776e16ca2a7c5d439f84a2494d4dc24aa6786b2335df680cdc9192
└── 🔐 Signatures for an image tag: quay.io/redhat-user-workloads/rhel-ai-poc-tenant/models/merlinite-poc:sha256-e6072aa3763c0b88b33084f001cafa5ef5bac11058fd27dbca7a3e5a0bfba91f.sig
   └── 🍒 sha256:48bc14bdf03968c83c61842b7bf81604a9fef7c8f8165a4abe6b0077ef97fe44
└── 📦 SBOMs for an image tag: quay.io/redhat-user-workloads/rhel-ai-poc-tenant/models/merlinite-poc:sha256-e6072aa3763c0b88b33084f001cafa5ef5bac11058fd27dbca7a3e5a0bfba91f.sbom
   └── 🍒 sha256:8f33660ec9145ef91369707c7420c7974bd0e13dcc170820fb3f3e93838f9a86
```

```bash
❯ skopeo inspect --raw docker://quay.io/redhat-user-workloads/rhel-ai-poc-tenant/models/merlinite-poc:82c0d0ab382f17a8567da12f34640626d207c868 | jq
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "artifactType": "application/x-mlmodel",
  "config": {
    "mediaType": "application/vnd.oci.empty.v1+json",
    "digest": "sha256:44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a",
    "size": 2,
    "data": "e30="
  },
  "layers": [
    {
      "mediaType": "image/svg+xml",
      "digest": "sha256:3613c73f07ccae19118bfe6d2f8cd127183d08cf99468a708e090953e116ed0a",
      "size": 4634,
      "annotations": {
        "org.opencontainers.image.title": "huggingface.svg"
      }
    },
    {
      "mediaType": "application/vnd.gguf",
      "digest": "sha256:9ca044d727db34750e1aeb04e3b18c3cf4a8c064a9ac96cf00448c506631d16c",
      "size": 4368484992,
      "annotations": {
        "org.opencontainers.image.title": "merlinite-7b-lab-Q4_K_M.gguf"
      }
    }
  ],
  "annotations": {
    "org.opencontainers.image.created": "2024-06-24T17:00:49Z"
  }
}
```

```bash
❯ oras pull quay.io/redhat-user-workloads/rhel-ai-poc-tenant/models/merlinite-poc:82c0d0ab382f17a8567da12f34640626d207c868
✓ Pulled      merlinite-7b-lab-Q4_K_M.gguf                                     4.07/4.07 GB 100.00%  2m31s
  └─ sha256:9ca044d727db34750e1aeb04e3b18c3cf4a8c064a9ac96cf00448c506631d16c
✓ Pulled      huggingface.svg                                                  4.53/4.53 kB 100.00%   14ms
  └─ sha256:3613c73f07ccae19118bfe6d2f8cd127183d08cf99468a708e090953e116ed0a
✓ Pulled      application/vnd.oci.image.manifest.v1+json                         475/475  B 100.00%     0s
  └─ sha256:9eded4b501f27ede70107f46941f43cd6d4d4fb00859a8b1b49d512a1dbf859d
✓ Pulled      application/vnd.oci.image.manifest.v1+json                         512/512  B 100.00%   75µs
  └─ sha256:ce1195719ed41d4820ea5cabc672c352b22786d471a271780522f2ad3ccddf4a
✓ Pulled      application/vnd.oci.image.index.v1+json                            462/462  B 100.00%     0s
  └─ sha256:57d759c7b811e575d6cbfe08b233b16e55c4e894e6faa20c8e7ed5d9bf3c9d8c
```

```bash
❯ ls
huggingface.svg  merlinite-7b-lab-Q4_K_M.gguf
```

```bash
❯ cosign download sbom quay.io/redhat-user-workloads/rhel-ai-poc-tenant/models/merlinite-poc:82c0d0ab382f17a8567da12f34640626d207c868
{
  "$schema": "http://cyclonedx.org/schema/bom-1.5.schema.json",
  "bomFormat": "CycloneDX",
  "specVersion": "1.5",
  "version": 1,
  "components": [
    {
      "purl": "pkg:generic/huggingface.svg?download_url=https://huggingface.co/front/assets/huggingface_logo-noborder.svg&checksum=sha256:3613c73f07ccae19118bfe6d2f8cd127183d08cf99468a708e090953e116ed0a",
      "type": "file",
      "name": "huggingface.svg",
      "hashes": [
        {
          "alg": "SHA-256",
          "content": "3613c73f07ccae19118bfe6d2f8cd127183d08cf99468a708e090953e116ed0a"
        }
      ],
      "externalReferences": [
        {
          "type": "distribution",
          "url": "https://huggingface.co/front/assets/huggingface_logo-noborder.svg"
        }
      ]
    },
    {
      "purl": "pkg:generic/merlinite-7b-lab-Q4_K_M.gguf?download_url=https://huggingface.co/instructlab/merlinite-7b-lab-GGUF/resolve/4bb27da133fc4888d687ab731ac7faf0ed804c6d/merlinite-7b-lab-Q4_K_M.gguf&checksum=sha256:9ca044d727db34750e1aeb04e3b18c3cf4a8c064a9ac96cf00448c506631d16c",
      "type": "file",
      "name": "merlinite-7b-lab-Q4_K_M.gguf",
      "hashes": [
        {
          "alg": "SHA-256",
          "content": "9ca044d727db34750e1aeb04e3b18c3cf4a8c064a9ac96cf00448c506631d16c"
        }
      ],
      "externalReferences": [
        {
          "type": "distribution",
          "url": "https://huggingface.co/instructlab/merlinite-7b-lab-GGUF/resolve/4bb27da133fc4888d687ab731ac7faf0ed804c6d/merlinite-7b-lab-Q4_K_M.gguf"
        }
      ]
    }
  ]
}
```
