# Progressive Object-Token Split Inference for Deadline-Aware Underwater Debris Reporting over Acoustic Links

> **Submission status.** This repository currently contains the project overview, evaluation summary, and dataset links. We plan to publicly release the detector code, acoustic-link simulator, evaluation scripts, and experiment configurations here upon acceptance. Additional materials can be made available upon request.

## Overview

An autonomous underwater vehicle (AUV) can detect debris onboard and still fail to report it to the surface in time. Over a low-rate acoustic link, a compressed image or crop may be unusable if a required packet is lost or arrives late. Sending only a class prediction and bounding box uses less airtime, but gives the receiver no visual features to refine the detection.

Our method splits detection between the AUV and a surface gateway. The AUV sends a compact representation of each object proposal so the gateway can make an initial detection from a single packet and refine it as more evidence arrives.

## How it works

1. **Encode proposals onboard.** Each selected object proposal is represented by a quantized base token and four independently packetized enhancement groups. Including packet overhead, the base packet is 44 bytes and each enhancement packet is 28 bytes.
2. **Send base packets first.** The transmitter prioritizes base packets and uses the remaining airtime for eligible enhancement packets.
3. **Refine at the gateway.** The gateway produces a provisional detection from the base token alone. It can refine that detection using any subset of enhancements processed before the cutoff, without reconstructing the image or filling in missing features.

## Datasets

| Dataset | Role | Images |
| --- | --- | ---: |
| [SeaClear Marine Debris Detection & Segmentation Dataset](https://doi.org/10.4121/4f1dff25-e157-4399-a5d4-478055461689) | Training, validation, and held-out testing | 5,770 train / 1,080 validation / 1,760 test |
| [TrashCan 1.0](https://doi.org/10.13020/g1gx-y834) | External testing only | 1,147 test |

Download the datasets from their original sources. The SeaClear test images are held out at the image level and may share locations or cameras with the training images. TrashCan 1.0 was not used for training, threshold tuning, or checkpoint selection.

## Evaluation setup

The experiments replay real underwater images through a **discrete-event acoustic-link simulator**. The results are from simulation, not at-sea modem measurements.

| Reference setting | Value |
| --- | ---: |
| Link rate | 8 kbit/s |
| Transmission range | 1,500 m |
| Frame interval | 5 s |
| Reporting deadline per frame | 15 s |
| Airtime ceiling per 900 s run | 450 s |
| Channel seeds | 20 (0–19) |

The primary metric is **on-time recall**, the fraction of annotated debris objects matched by a detection with an intersection-over-union of at least 0.50 and reported by the frame deadline. Detection thresholds were selected on validation data for a target of 0.25 false alarms per image. Actual test rates are reported below.

The study compares progressive object tokens with six baselines: metadata-only reporting, JPEG crops, JPEG frames, progressive crops, a monolithic latent representation, and a packet-tolerant latent representation.

## Key results

The table shows the reference operating point from Table IV of the manuscript. The packet-tolerant latent was the strongest baseline by on-time recall on both test sets.

| Test set | Method | On-time recall | False alarms per image | Transmitted data per full run |
| --- | --- | ---: | ---: | ---: |
| SeaClear | Packet-tolerant latent | 42.3% | 0.26 | 168.9 KiB |
| SeaClear | **Progressive object tokens** | **46.8%** | 0.25 | 132.4 KiB |
| TrashCan 1.0 | Packet-tolerant latent | 32.6% | 0.33 | 168.7 KiB |
| TrashCan 1.0 | **Progressive object tokens** | **35.4%** | 0.32 | 134.3 KiB |

Compared with the strongest baseline, on-time recall improved by **4.5 percentage points** on SeaClear (95% CI 3.6–5.4) and **2.8 percentage points** on TrashCan 1.0 (95% CI 1.7–3.9). On SeaClear, progressive object tokens transmitted **22% fewer bytes** at nearly the same modem airtime. The confidence intervals reflect variability across channel seeds on the same test images.

## Planned release

- Split-inference detector code, including the object-token encoder and gateway refinement head
- Discrete-event simulator for packet scheduling, acoustic transmission, and gateway processing
- Experiment configurations and the SeaClear split lists used in the study
- Evaluation scripts and instructions for reproducing the reported comparisons
