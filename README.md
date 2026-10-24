# Construction Safety Detection — Computer Vision Study

An academic notebook exploring **hard-hat violation detection** with a fine-tuned YOLOv8n detector, open-vocabulary YOLO-World, Florence-2 and Moondream2.

Developed for the Digital Image Processing and Computer Vision course at Universidad Alberto Hurtado. The notebook and demonstrations are in Spanish.

**[Read the notebook](proyecto.ipynb)** · [Detector demonstration](https://youtu.be/HCQj_n97s-8) · [Deployment discussion](https://youtu.be/XaC0UP39Vbw)

## Research question

How do a task-specific detector, an open-vocabulary detector and vision-language models behave when asked whether a construction worker is **not wearing a hard hat**?

The notebook uses the Construction Site Safety dataset from Roboflow, version 30, and samples a separate demonstration video at approximately one frame per second. It explores localization, event-level classification, inference time and sensitivity to prompt wording.

## Approaches

| Approach | Model | Role in the experiment |
| --- | --- | --- |
| Closed vocabulary | Fine-tuned YOLOv8n | Detects dataset classes, including `NO-Hardhat`. |
| Open vocabulary | YOLOv8s-World | Uses text classes such as `hard hat`, `head` and `person`. |
| Grounded vision-language model | Florence-2-base | Demonstrates bounding-box grounding for a text query. |
| Generative vision-language model | Moondream2, revision `2024-05-20` | Answers questions about missing hard hats and is tested with prompt variations. |

## Recorded event-level results

The final evaluation cell contains **14 manually labeled video frames**, including six positive hard-hat violations. These values come from the notebook's saved outputs; they have not been independently reproduced here.

| Model | True positives | False positives | False negatives | Event F1 |
| --- | ---: | ---: | ---: | ---: |
| YOLOv8n | 2 | 1 | 4 | 0.444 |
| YOLO-World | 0 | 0 | 6 | 0.000 |
| Moondream2 | 6 | 8 | 0 | 0.600 |

Moondream2 predicted a violation in every evaluated frame. Its higher F1 in this small sample therefore does not establish useful discrimination between safe and unsafe frames. YOLOv8n also missed four of six positive frames. These results do **not** validate a production safety-monitoring system.

The YOLO-World event heuristic treats a detected `head` as evidence of a missing hard hat; it does not establish a person-to-head-to-helmet association. Florence-2 is demonstrated separately and has no final event-level F1 result in the notebook.

## Reproduce the study

The notebook is designed around a **Google Colab-style Python environment with a CUDA GPU** and `/content/` paths. The recorded runs used a Tesla T4.

1. Download or clone this repository and open `proyecto.ipynb` in Colab or a compatible notebook environment.
2. Review its installation cells and use an isolated runtime. The notebook installs Ultralytics, Roboflow and a pinned Transformers version for its VLM experiments.
3. Download Construction Site Safety version 30 through Roboflow using your own credentials, entered at runtime. Never commit a key.
4. Supply your own input video and update the `/content/videoplayback 2.mp4` path and dataset paths.
5. Run frame extraction and training before cells that load the generated `best.pt` checkpoint.
6. Review each sampled frame and align its label with the numerical filename order before running event evaluation. Do not reuse the included label list for a different video.

The repository contains the notebook, not the dataset, input video or trained weights. It is not a ready-to-run inference application.

## Limitations and next steps

- The 14-frame event sample is too small to support generalization or a deployment claim.
- The evaluation truncates predictions and labels to their minimum length. A reproducible evaluation should reject mismatched lengths instead.
- Box-level metrics and binary event F1 answer different questions. YOLO-World's saved box validation requires review of class-ID mapping before comparison with the fine-tuned detector.
- Saved timing outputs use different execution paths. Moondream's timed answer excludes image encoding, so these figures are not an end-to-end speed benchmark across models.
- Spanish and English prompt variants produce different answers; parsing an answer by checking for `yes` is a limited classifier.
- The notebook contains environment-specific dependency workarounds, including a dummy `flash_attn` module, and cells that load remote model code. It needs environment cleanup and pinned revisions before reliable reproduction.

A stronger follow-up would use a larger held-out event dataset, verified class mappings, precision/recall and confusion matrices, repeated latency measurements and consistent prompts. Human review remains necessary when assessing safety-related outputs.

## Dependencies and attribution

The experiment uses Ultralytics YOLO, Roboflow, PyTorch, OpenCV, Hugging Face Transformers, Florence-2 and Moondream2. Consult the licenses and model cards for the exact dataset, package and model revisions you use before redistribution. No repository-wide license is declared here.
