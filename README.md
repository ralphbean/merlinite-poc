# Model Upload PoC

This is a proof of concept of using [konflux](https://konflux-ci.dev) to upload [instructlab/merlinite-7b-lab-GGUF](https://huggingface.co/instructlab/merlinite-7b-lab-GGUF) to quay.

The configuration of what to pull and upload is driven by `oci.yaml`.

The process is achieved by the tekton pipeline in `.tekton/`.

You can examine the output builds at quay.io/redhat-user-workloads/rhel-ai-poc-tenant/models/merlinite-poc:99232c2355efd09ef5d0f4aca4fe3e22a7849a92.

There are two files stored in the registry: the gguf model, and the huggingface logo. There's no point in distributing the huggingface logo like this - it's just there to demonstrate what distributing more than one file looks like.

```bash
❯ cosign tree quay.io/redhat-user-workloads/rhel-ai-poc-tenant/models/merlinite-poc:99232c2355efd09ef5d0f4aca4fe3e22a7849a92
📦 Supply Chain Security Related artifacts for an image: quay.io/redhat-user-workloads/rhel-ai-poc-tenant/models/merlinite-poc:99232c2355efd09ef5d0f4aca4fe3e22a7849a92
└── 💾 Attestations for an image tag: quay.io/redhat-user-workloads/rhel-ai-poc-tenant/models/merlinite-poc:sha256-57d759c7b811e575d6cbfe08b233b16e55c4e894e6faa20c8e7ed5d9bf3c9d8c.att
   └── 🍒 sha256:411d171c8330666195b60fed9c879973ce7eb7a2c59ed3df4b51ed0f89a87287
└── 🔐 Signatures for an image tag: quay.io/redhat-user-workloads/rhel-ai-poc-tenant/models/merlinite-poc:sha256-57d759c7b811e575d6cbfe08b233b16e55c4e894e6faa20c8e7ed5d9bf3c9d8c.sig
   └── 🍒 sha256:eaf6f72cfc268c7a05398fcf10a5846b50674efe9005dc03995ae419bd436e9b
└── 📦 SBOMs for an image tag: quay.io/redhat-user-workloads/rhel-ai-poc-tenant/models/merlinite-poc:sha256-57d759c7b811e575d6cbfe08b233b16e55c4e894e6faa20c8e7ed5d9bf3c9d8c.sbom
   └── 🍒 sha256:e8ce2ea22b156a7e53114a5ff622a7cecc717ff781596c788fc97cbf93587ba5
```

```bash
❯ skopeo inspect --raw docker://quay.io/redhat-user-workloads/rhel-ai-poc-tenant/models/merlinite-poc:99232c2355efd09ef5d0f4aca4fe3e22a7849a92 | jq
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.index.v1+json",
  "manifests": [
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "digest": "sha256:9eded4b501f27ede70107f46941f43cd6d4d4fb00859a8b1b49d512a1dbf859d",
      "size": 475,
      "artifactType": "image/svg+xml"
    },
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "digest": "sha256:ce1195719ed41d4820ea5cabc672c352b22786d471a271780522f2ad3ccddf4a",
      "size": 512,
      "artifactType": "application/vnd.gguf"
    }
  ]
}
```

```bash
❯ oras pull quay.io/redhat-user-workloads/rhel-ai-poc-tenant/models/merlinite-poc:99232c2355efd09ef5d0f4aca4fe3e22a7849a92
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
❯ cosign download sbom quay.io/redhat-user-workloads/rhel-ai-poc-tenant/models/merlinite-poc:99232c2355efd09ef5d0f4aca4fe3e22a7849a92 
{
  "$schema": "http://cyclonedx.org/schema/bom-1.5.schema.json",
  "bomFormat": "CycloneDX",
  "specVersion": "1.5",
  "version": 1,
  "components": [
    {
      "purl": "pkg:generic/huggingface.svg?download_url=https://huggingface.co/front/assets/huggingface_logo-noborder.svg&checksum=sha256:3613c73f07ccae19118bfe6d2f8cd127183d08cf99468a708e090953e116ed0a",
      "type": "file",
      "name": "huggingface.svg"
    },
    {
      "purl": "pkg:generic/merlinite-7b-lab-Q4_K_M.gguf?download_url=https://huggingface.co/instructlab/merlinite-7b-lab-GGUF/resolve/4bb27da133fc4888d687ab731ac7faf0ed804c6d/merlinite-7b-lab-Q4_K_M.gguf&checksum=sha256:9ca044d727db34750e1aeb04e3b18c3cf4a8c064a9ac96cf00448c506631d16c",
      "type": "file",
      "name": "merlinite-7b-lab-Q4_K_M.gguf"
    }
  ]
}
```
