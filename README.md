# STDP-inspired-clinical-events-modeling-# STDP-inspired clinical event modeling

This repository contains notebooks for building and evaluating interpretable temporal transition features from longitudinal electronic health record (EHR) data. The current release focuses on two prediction tasks, acute kidney injury (AKI) and postoperative recurrence prediction in pancreatic ductal adenocarcinoma (PDAC), with a condition-based transition modeling workflow.

The modeling framework is inspired by spike-timing-dependent plasticity (STDP): clinical events are represented as sparse temporal pairs, where events that occur closer together contribute more strongly than events farther apart. In this implementation, STDP is used as a computational analogy for temporally ordered EHR events, not as a neurobiological or causal model.

# Data availability

Institutional EHR data are not included in this repository because they contain protected health information. The notebooks are structured so that users can adapt the workflow to their own de-identified event-day tables with the required columns.
## Current focus

The current GitHub section covers condition-based transition modeling for PDAC recurrence prediction. Later sections will add condition-to-laboratory and laboratory-to-condition transition tuning notebooks.

The condition-transition workflow includes:

1. Raw condition event-day modeling.
2. ICD-to-Phecode mapping.
3. Hybrid Phecode + unmapped condition burden features.
4. Forward-only Phecode and Phecode-group transition features.
5. Bidirectional proximity sensitivity analysis.
6. Clinical utilization/documentation-intensity adjustment.
7. Prediction-head tuning under a VVUQ framework.

## Why Phecodes?

Raw diagnosis labels can be sparse, institution-specific, and difficult to interpret consistently. To make condition transitions more standardized, the workflow maps ICD-9/ICD-10 source codes to Phecodes when possible.

Mapped Phecode tokens are used for standardized clinical-category modeling. Unmapped condition tokens are retained for burden adjustment so that patients are not excluded only because a condition code did not map to a Phecode.

## Forward-only vs bidirectional transitions

Forward-only transitions are the primary biologically interpretable representation.

A forward transition means:

```text
event A occurred before event B within a prespecified temporal window

