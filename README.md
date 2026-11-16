Progressive Object-Token Split Inference for Deadline-Aware Underwater Debris Reporting over Acoustic Links
Submission-stage repository. This repository currently contains the project overview, evaluation summary, and dataset links. The detector code, acoustic-link simulator, evaluation scripts, and experiment configurations are planned for public release here upon acceptance. Editors and reviewers may request specific materials needed for peer review through the journal’s submission system.

Authors: Ahmad Junaid, Fatma S. Alrayes, Abuzar Khan, Muhammad Attique Khan, Jamel Baili, Anwer Al-Dulaimi, Ghassan Husnain, and Sangsoon Lim.
Overview
An autonomous underwater vehicle (AUV) may detect debris onboard but still fail to report it to the surface in time. Over a low-rate acoustic link, a compressed image or crop may be unusable if a required packet is lost or arrives late. Sending only a class prediction and bounding box is cheaper, but gives the receiver no visual features to refine the detection.

This work splits detection between the AUV and a surface gateway. The AUV sends a compact representation for each object proposal so the gateway can make an initial detection from a single packet and improve it as more evidence arrives.
How the method works
1. Encode proposals onboard. Each selected proposal becomes a quantized base token and four separately packetized enhancement groups. The base packet is 44 bytes and each enhancement packet is 28 bytes, including packet overhead.
2. Schedule the base first. The transmitter prioritizes base packets and uses remaining airtime for eligible enhancement packets under a shared airtime ceiling.
3. Refine at the gateway. The gateway produces a provisional detection from the base alone. Its masked fusion head refines the prediction using any subset of enhancements processed before the cutoff, without reconstructing pixels or filling in missing features.
Datasets
Dataset	Role in the study	Images
SeaClear Marine Debris Detection & Segmentation Dataset	Training, validation, and held-out testing	5,770 train / 1,080 validation / 1,760 test
TrashCan 1.0	External test only, with no training, tuning, or checkpoint selection on this dataset	1,147 test


Download the datasets from their original sources. The SeaClear test images are held out at the image level and may share locations or cameras with the training images.
Evaluation setup
The experiments use real underwater images replayed through a discrete-event acoustic-link simulator. The reported results are simulation results, not at-sea modem measurements. The reference scenario uses an 8 kbit/s link over 1,500 m, one frame every 5 s, a 15 s reporting deadline per frame, and a 450 s modem-airtime ceiling per 900 s run. Each scenario is evaluated across 20 channel seeds (0–19).

The primary metric is on-time recall, the fraction of annotated debris objects matched by a correct detection (IoU ≥ 0.50) reported by the frame deadline. Thresholds are selected on validation data for a target of 0.25 false alarms per image. Realized test rates are reported separately below.

The study compares the method with six baselines: metadata-only reporting, JPEG crops, JPEG frames, progressive crops, a monolithic latent, and a packet-tolerant latent.
Key results
Reference operating point (manuscript Table IV). The packet-tolerant latent was the strongest baseline by on-time recall on both test sets.
Test set	Method	On-time recall	False alarms/image	Transmitted data per full run
SeaClear	Packet-tolerant latent	42.3%	0.26	168.9 KiB
SeaClear	Progressive object tokens	46.8%	0.25	132.4 KiB
TrashCan 1.0	Packet-tolerant latent	32.6%	0.33	168.7 KiB
TrashCan 1.0	Progressive object tokens	35.4%	0.32	134.3 KiB


The recall gain was 4.5 percentage points on SeaClear (95% CI: 3.6–5.4) and 2.8 percentage points on TrashCan 1.0 (95% CI: 1.7–3.9). On SeaClear, progressive object tokens transmitted 22% fewer bytes than the packet-tolerant latent at nearly the same modem airtime. The confidence intervals reflect variability across channel seeds on the fixed test images.
Planned release
- Split-inference detector code, including the token encoder and gateway refinement head.
- Discrete-event simulator for packet scheduling, acoustic transmission, and gateway processing.
- Experiment configurations and SeaClear split lists used in the study.
- Evaluation scripts and instructions for reproducing the reported comparisons.
The implementation and experiment files are not included in the current repository. The dataset links above are available now.
