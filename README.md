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

A forward transition means: event A occurred before event B within a prespecified temporal window

## Limitations

This repository is intended for methodological development and retrospective validation, not for direct clinical deployment. Several limitations should be considered when interpreting the notebooks and model outputs.

First, EHR timestamps reflect both disease biology and clinical behavior. Diagnosis codes and laboratory measurements may be influenced by visit frequency, documentation practices, billing workflows, referral patterns, and preoperative workup intensity. Although the workflow includes utilization and documentation-intensity adjustment, residual confounding by clinical observation remains possible.

Second, STDP-inspired transition features represent temporally ordered associations, not causal effects. A forward transition indicates that one event was documented before another within a specified time window, but it does not prove that the earlier event caused the later event or the outcome. Bidirectional sensitivity features should be interpreted only as local temporal proximity or co-documentation patterns.

Third, lag and tau are empirical hyperparameters. They define the maximum transition window and temporal decay strength, but they should not be interpreted as fixed biological constants. Different clinical settings, institutions, event types, and prediction horizons may require different lag/tau choices.

Fourth, condition mapping depends on source-code quality. ICD-to-Phecode mapping improves standardization, but not all source codes map cleanly. The workflow retains unmapped conditions for burden adjustment, but unmapped tokens are less standardized and should not be over-interpreted as biological transition states.

Fifth, this PDAC recurrence task uses a single-institution retrospective cohort. Model performance may vary across institutions because of differences in coding practices, surgical workflows, follow-up intensity, recurrence ascertainment, patient mix, and treatment patterns. External validation is required before clinical generalization.

Sixth, the current condition-transition notebooks do not yet include all clinically relevant modalities. Medications, procedures, chemotherapy timing, radiology reports, pathology details, imaging features, tumor markers, and laboratory trajectories may provide additional recurrence-relevant signal. Later notebooks will extend this framework to condition-to-laboratory and laboratory-to-condition transitions.

Finally, model discrimination remains modest in the PDAC recurrence setting. These models should be interpreted as tools for studying temporal EHR representations and generating hypotheses about recurrence-associated trajectories, not as standalone surgical decision models.
