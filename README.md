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

## Condition-to-condition and condition-to-laboratory transition pairs

This workflow evaluates two main types of temporally ordered transition pairs.

Condition-to-condition transitions capture longitudinal clinical-state trajectories. These features ask whether one condition/Phecode-derived clinical state was documented before another condition state within a prespecified lag/tau window. Because diagnosis-code trajectories may evolve over days to weeks, condition-to-condition transitions were evaluated with broader temporal windows.

Condition-to-laboratory transitions capture cross-modal physiologic patterns. These features ask whether a condition/Phecode-derived clinical state was followed by an abnormal laboratory state or laboratory axis within a prespecified lag/tau window. Because laboratory testing often occurs during the same or near-adjacent clinical workup episode, condition-to-laboratory transitions were evaluated primarily with shorter temporal windows.

In the combined model, both transition families are used together. Conceptually, the patient timeline is represented as an ordered sequence of condition and laboratory events. The model then constructs decayed transition scores for selected event pairs, allowing it to capture both longitudinal diagnosis trajectories and short-range condition-to-physiology relationships.

For example:

condition A → condition B
condition B → abnormal laboratory axis C

may represent a clinical trajectory followed by a physiologic abnormality. These features preserve directionality and timing, but they should be interpreted as temporally ordered associations rather than causal pathways.

Lag and tau were treated as empirical hyperparameters. The lag parameter defines the maximum allowable time gap between two events, while tau controls how quickly more distant event pairs are down-weighted. Because different event families may operate on different clinical time scales, condition-to-condition and condition-to-laboratory transitions were tuned separately before being evaluated in combined models.

## ## Rolling risk prediction

The rolling risk prediction workflow evaluates the same STDP-inspired feature construction across multiple preoperative prediction horizons. Instead of building one fixed patient-level feature vector using the full preoperative record, the patient timeline is progressively truncated at clinically relevant cutoff points. At each cutoff, only events available up to that time are used to reconstruct the feature vector and estimate recurrence risk.

For the PDAC recurrence task, rolling prediction horizons include:

```text
Day -90, Day -60, Day -30, Day -14, Day -7, Day -3, and Day 0 (7days before recorded surgery date to avoid data leakage and documentation delay)
```

where each day is defined relative to the surgical/index date. Day -30 is treated as the primary preoperative decision horizon, while later horizons evaluate how risk estimates update as additional clinical information becomes available closer to surgery.

At each rolling horizon, the workflow performs the following steps:

```text
1. Truncate each patient's event timeline at the horizon-specific cutoff.
2. Rebuild condition-frequency, laboratory-frequency, and utilization features using only visible pre-cutoff data.
3. Recompute STDP-inspired transition features using only event pairs available before the cutoff.
4. Fit and evaluate prediction models using patient-level cross-validation.
5. Compare risk estimates across horizons to assess dynamic risk evolution.
```

This design simulates a surveillance setting in which a patient’s risk estimate is updated as new diagnoses, laboratory abnormalities, and clinical encounters accumulate over time. For example, a patient may have an initial risk estimate at Day -30, followed by updated estimates at Day -14, Day -7, and Day 0 as additional preoperative events become available.

The rolling prediction analysis is intended to evaluate both absolute risk at each horizon and changes in risk over time. Earlier horizons may have weaker discrimination because less proximal clinical information is available. Later horizons may show stronger performance if recurrence-associated patterns become more visible during the final preoperative workup period. Therefore, this workflow should be interpreted as a dynamic risk-surveillance framework rather than a single fixed-time prediction model.


## Limitations

This repository is intended for methodological development and retrospective validation, not for direct clinical deployment. Several limitations should be considered when interpreting the notebooks and model outputs.

First, EHR timestamps reflect both disease biology and clinical behavior. Diagnosis codes and laboratory measurements may be influenced by visit frequency, documentation practices, billing workflows, referral patterns, and preoperative workup intensity. Although the workflow includes utilization and documentation-intensity adjustment, residual confounding by clinical observation remains possible.

Second, STDP-inspired transition features represent temporally ordered associations, not causal effects. A forward transition indicates that one event was documented before another within a specified time window, but it does not prove that the earlier event caused the later event or the outcome. Bidirectional sensitivity features should be interpreted only as local temporal proximity or co-documentation patterns.

Third, lag and tau are empirical hyperparameters. They define the maximum transition window and temporal decay strength, but they should not be interpreted as fixed biological constants. Different clinical settings, institutions, event types, and prediction horizons may require different lag/tau choices.

Fourth, condition mapping depends on source-code quality. ICD-to-Phecode mapping improves standardization, but not all source codes map cleanly. The workflow retains unmapped conditions for burden adjustment, but unmapped tokens are less standardized and should not be over-interpreted as biological transition states.

Fifth, this PDAC recurrence task uses a single-institution retrospective cohort. Model performance may vary across institutions because of differences in coding practices, surgical workflows, follow-up intensity, recurrence ascertainment, patient mix, and treatment patterns. External validation is required before clinical generalization.

Sixth, the current condition-transition notebooks do not yet include all clinically relevant modalities. Medications, procedures, chemotherapy timing, radiology reports, pathology details, imaging features, tumor markers, and laboratory trajectories may provide additional recurrence-relevant signal. Later notebooks will extend this framework to condition-to-laboratory and laboratory-to-condition transitions.

Finally, model discrimination remains modest in the PDAC recurrence setting. These models should be interpreted as tools for studying temporal EHR representations and generating hypotheses about recurrence-associated trajectories, not as standalone surgical decision models.

## Acknowledgements

The authors gratefully acknowledge Benjamin May for his guidance and support with OMOP data access, data curation, and structured EHR data interpretation. The authors also thank Harry Reyes Nieva for his guidance on clinical informatics, AI governance, reporting considerations, and OMOP-based data workflows. The authors acknowledge PhysioNet and the MIMIC-IV data contributors for making the MIMIC-IV dataset available to credentialed researchers, enabling external benchmarking of the STDP-inspired temporal transition modeling framework.
