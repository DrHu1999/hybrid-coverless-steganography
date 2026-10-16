# Upstream components

This archive contains newly written orchestration/codec code, not vendored upstream implementations or weights. The following APIs were consulted and are used by optional adapters:

- [Hugging Face Diffusers ControlNet](https://huggingface.co/docs/diffusers/en/api/pipelines/controlnet)
- [SwinFace](https://github.com/lxq1000/SwinFace), including its public inference entry point and task-output interface
- [RetinaFace Python package](https://github.com/serengil/retinaface)
- [MMPose inference APIs](https://mmpose.readthedocs.io/en/latest/_modules/mmpose/apis/inference.html)
- [OpenPose ControlNet model card](https://huggingface.co/lllyasviel/sd-controlnet-openpose)
- [MagicPoser](https://magicposer.com), an external manual pose-design tool named in the paper

Install/download components from their upstream sources and preserve their applicable notices and model terms. The repository MIT license applies to new repository code, not to those separate materials or the published paper. Exact versions/hashes must be recorded for experimental reproducibility.
