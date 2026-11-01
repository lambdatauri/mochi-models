# Mochi models

Model files that [Mochi](https://github.com/lambdatauri) downloads for its opt-in vision packs, when no upstream
publishes them in a form Mochi can run. Each file is converted from a pinned upstream release, is byte-for-byte
reproducible, and is verified by Mochi against the size and SHA-256 below before use.

Files are published as release assets; releases are immutable.

## Animal pack v1 (`animal-pack-v1`)

| File | Bytes | SHA-256 |
|---|---|---|
| `detector.onnx` | 80,351,225 | `c0eaebcd8d83ee15b7e0a5d64644216c77546597a31316c49158e1beabb025f5` |
| `bioclip2-image.onnx` | 608,746,749 | `d00c15a5899ca963953d9659900433c180e7cdff93bf5d003c8c2539cffa025c` |
| `species-v1.json` | 8,243,110 | `d226920cb96b04e36e4968cd907b64400aaa17d4320874b22984c04cf7c9f06f` |
| `species-v1.f16` | 122,088,960 | `63445c13aabb5ad5003a6589ecbb9fb4e9741695f45a928e652853f1dd85e0fe` |

- **`detector.onnx`**: MegaDetector v6 `MDV6-apa-rtdetr-c` (Microsoft AI for Good Lab), exported to ONNX (opset 17) with its
  RT-DETRv2 post-processing. Inputs `images` (N×3×640×640, RGB in 0..1) and `orig_target_sizes` (N×2, width and height);
  outputs `labels` (0 animal, 1 person, 2 vehicle), `boxes` (xyxy in original pixels), and `scores`.
  Source weights: [Zenodo record 15398270](https://zenodo.org/records/15398270), `MDV6-apa-rtdetr-c.pth`, SHA-256
  `4814cdbc9a0dbf20e4b04495f060176d8340b00d182fc98ce808a4d14a2a6fce`; model code: `rtdetrv2_pytorch` from
  [microsoft/Biodiversity](https://github.com/microsoft/Biodiversity) at `d3e0a0697167bb859a6f1c97b2e81344ff86c28a`.
- **`bioclip2-image.onnx`**: the image encoder of [BioCLIP 2](https://huggingface.co/imageomics/bioclip-2) at revision
  `2957b322090f9cb17ae72c71981c7218a28d81e0`, exported to ONNX (opset 17). Large weights are stored as float16 and cast to
  float32 when loaded, so results match the float32 model (cosine 1.00000). Input `pixels` (N×3×224×224, squashed and
  normalised with the CLIP mean and deviation); output `embedding` (N×768, L2-normalised).
- **`species-v1.f16`, `species-v1.json`**: 79,485 animal species from the BioCLIP 2 text embeddings in
  [TreeOfLife-200M](https://huggingface.co/datasets/imageomics/TreeOfLife-200M) at revision
  `5f2dc493b3dc0e544438a04038ab15faa646b749` (867,455 species), trimmed to animals people photograph: every vertebrate,
  other animals with a common name, without microscopic groups. The `.f16` file holds N×768 little-endian float16 rows;
  the JSON holds each row's lineage and common name and BioCLIP 2's logit scale.

Built by `scripts/build_animal_pack.py` in the Mochi repository. Mochi's decision record (ADR 0029) has the validation:
parity with PyTorch, timing on Windows and Linux CPUs, and agreement with the full table.

## Licences and attribution

| File | Licence | Copyright and attribution |
|---|---|---|
| `detector.onnx` | Apache-2.0 (model, per the [MegaDetector model zoo](https://microsoft.github.io/MegaDetector/model_zoo/)); weights distributed on Zenodo under CC-BY-4.0; RT-DETRv2 code Apache-2.0 | MegaDetector v6 by the Microsoft AI for Good Lab; RT-DETRv2 by Wenyu Lv et al. ("RT-DETRv2: Improved Baseline with Bag-of-Freebies for Real-Time Detection Transformer", 2024). Converted to ONNX by Mochi. |
| `bioclip2-image.onnx` | MIT | Copyright (c) 2023-2026 Imageomics Institute. BioCLIP 2 by Jianyang Gu, Samuel Stevens, et al., doi:10.57967/hf/5765. Converted to ONNX with float16 weights by Mochi. |
| `species-v1.*` | CC0-1.0 (TreeOfLife-200M compilation); embeddings from BioCLIP 2 (MIT) | Imageomics Institute. Only taxonomy, common names, and text embeddings are included, no images. Trimmed by Mochi. |

Licence texts are in [`LICENSES/`](LICENSES). See [`NOTICE`](NOTICE).
