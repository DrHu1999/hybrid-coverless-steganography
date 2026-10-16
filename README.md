# Hybrid Coverless Image Steganography with Full-Body Human Images

[中文使用说明](README_zh.md) · [Method correspondence](docs/REPRODUCIBILITY.md) · [Model setup](docs/MODELS.md)

A paper-based research implementation of **A hybrid generative and mapping method for coverless image steganography with full-body human images**, Bo Hu, Yanhui Xiao, Qiyao Deng and Huawei Tian, *Neurocomputing* 693 (2026), 133842. [DOI](https://doi.org/10.1016/j.neucom.2026.133842).

This repository was newly implemented from the published method. It is not a release of the code used in the original experiments. The original 500-image dataset, trained weights, prompts and experiment logs are not included. No claim is made that the reported paper results have been reproduced here.

## Implemented

- Table 1 face attribute codebook, including categorical hairstyle and hair color.
- COCO-17 coordinate normalization, elbow-angle bins, wrist/ankle ratios and optional nose-distance comparison, Eqs. (1)-(3).
- Message segmentation, a separate padding-count image, image-only feature extraction and at most 10 validation attempts.
- Local Stable Diffusion + OpenPose ControlNet generation adapter; RetinaFace + official SwinFace and MMPose ViTPose adapters.
- Human-in-the-loop pose bank, consistent with the manual pose construction in Algorithm 1.
- All 14 attack families / 32 parameter configurations from Table 5, whole-segment accuracy from Eq. (7), erasure accounting and greedy attribute selection.
- Unit tests, a model-free mathematical demo, citation metadata and GitHub Actions.

## Quick start without models

Python 3.10 or later:

```bash
python -m venv .venv
# Activate .venv using the command appropriate for your operating system.
python -m pip install -e .
python -m unittest discover -s tests -v
python -m hcs demo --output runs/demo
```

The demo checks ideal attribute/geometry coding and framing only. It creates a skeleton diagram and explicitly labeled ideal observations. It does **not** generate a person image or demonstrate visual information recovery.

## Real image workflow

1. Install and configure the real model backends using [MODELS.md](docs/MODELS.md). Weights must be obtained separately from upstream sources.
2. Choose `configs/paper28.json` (the paper's explicit 15+13 allocation) or `configs/example24.json` (a configurable 12+12 example, not a recovered optimal subset).
3. Create a private sender plan:

```bash
python -m hcs plan --config configs/paper28.json --input examples/message.txt --output private/plan.json
```

4. For each distinct pose bit pattern, construct a pose in MagicPoser or another pose editor. Record its COCO-17 keypoints on the same canvas as its control image. Register it:

```bash
python -m hcs add-pose --config configs/paper28.json --bank pose_bank --keypoints private/pose-keypoints.json --width 512 --height 768 --control private/control.png
```

Omit `--control` to render an OpenPose-style skeleton directly from measured keypoints. This renderer is an implementation choice; it does not reproduce a particular MagicPoser rendering. See the [pose format](docs/POSE_BANK.md).

5. Generate and verify each image, including the final padding-count image:

```bash
python -m hcs send --config configs/paper28.json --input examples/message.txt --pose-bank pose_bank --output runs/transmission --seed 2026
```

6. Recover the message in a receiver process:

```bash
python -m hcs receive --config configs/paper28.json --images runs/transmission --output runs/recovered.txt
```

The receiver takes only an ordered image package, a shared codec and model backends. It never reads the sender plan, pose bank, prompts or ground-truth labels. `manifest.json` contains ordering and a codec fingerprint, not the secret payload. Models and their preprocessing must also be matched by sender and receiver; the codec fingerprint alone does not verify model equivalence.

To inspect a real image with the configured recognizers, use `python -m hcs observe --config configs/paper28.json --image path/to/person.png --output private/observation.json`. This saves actual probabilities and COCO keypoints for debugging; it is private analysis material, not part of a transmitted image package.

An unsuccessful generation aborts; no complete manifest is emitted. Ten attempts do not guarantee that every input segment can be generated successfully. Missing pose templates are explicit errors; no artificial carrier is substituted.

## Evaluation

Prepare a **private** JSON list pairing each test image with its expected L-bit segment:

```json
[{"image": "000000.png", "bits": "0000000000000000000000000000"}]
```

```bash
python -m hcs evaluate --config configs/paper28.json --images runs/transmission --ground-truth private/ground-truth.json --output runs/evaluation.json
```

This computes actual image inference after each attack. All failed detections remain in the whole-segment denominator. Bit accuracy on valid decodes and erasure rate are reported separately. Attack conventions that the paper leaves unspecified are documented in [REPRODUCIBILITY.md](docs/REPRODUCIBILITY.md). This repository does not contain invented benchmark results or an RS-analysis implementation.

## Repository layout

```text
hcs/          codecs, model adapters, pipeline, attacks, metrics and CLI
configs/      explicit 28-bit allocation and an illustrative 24-bit allocation
tests/        mathematical, protocol and image-processing tests
scripts/      isolated real-model worker
docs/         reproducibility notes, model setup and pose-bank format
examples/     sample public message
```

## Citation

```bibtex
@article{hu2026hybrid,
  title={A hybrid generative and mapping method for coverless image steganography with full-body human images},
  author={Hu, Bo and Xiao, Yanhui and Deng, Qiyao and Tian, Huawei},
  journal={Neurocomputing},
  volume={693},
  pages={133842},
  year={2026},
  doi={10.1016/j.neucom.2026.133842}
}
```

New repository code is provided under the MIT license. Third-party code and weights retain their own terms and are not bundled. See [THIRD_PARTY.md](THIRD_PARTY.md). This implementation provides information mapping, not cryptographic confidentiality or a guarantee of undetectability.
