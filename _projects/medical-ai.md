---

layout: page
title: Prognostic Modeling for COVID-19 Patients from Longitudinal Blood Tests
description: Developed a temporal deep-learning pipeline for COVID-19 prognosis prediction from longitudinal blood-test data, addressing limited sample size, label inconsistencies, and clinically informed feature design.
importance: 4
category: industry
related_publications: false
---------------------------

**Past Industry Research · Medical AI / Clinical Machine Learning**

## Overview

I worked on predicting the clinical prognosis of COVID-19 patients from **longitudinal blood-test measurements**.

The project involved a relatively small clinical dataset, making generalization and data quality particularly important. I therefore focused not only on model architecture, but also on label consistency, auxiliary learning objectives, and the incorporation of clinical knowledge into the modeling pipeline.

---

## Problem

Given a sequence of laboratory measurements for each patient,

$$
x_1, x_2, \ldots, x_T,
$$

the goal was to predict the patient's subsequent clinical outcome.

The main challenges included:

* limited training data,
* heterogeneous longitudinal measurements,
* inconsistencies between provided labels and actual clinical severity, and
* the need to generalize reliably to unseen patients.

Because the data naturally contained temporal structure and the available cohort was limited, I used an **LSTM-based model with attention** to incorporate temporal inductive bias while allowing the model to emphasize informative observations.

---

## Data Quality and Label Validation

One of the most important issues I identified was a mismatch between some provided outcome labels and the patients' actual clinical severity.

Rather than treating the supplied labels as ground truth without further inspection, I reviewed these inconsistencies and revised the dataset based on the available clinical information.

My contributions at the data level included:

* identifying inconsistencies between outcome labels and clinical status,
* validating problematic samples, and
* revising the dataset before model training and evaluation.

This step was particularly important because label errors can directly limit downstream model performance regardless of model architecture.

---

## Multi-Task Learning for Generalization

To reduce overfitting under limited training data, I introduced an additional prediction task that was clinically correlated with the primary prognosis objective.

The model was trained using a combined objective,

$$
\mathcal{L}
=
\mathcal{L}_{\mathrm{main}}
+
\lambda \mathcal{L}_{\mathrm{aux}},
$$

where the auxiliary task provided an additional learning signal to the shared representation.

The multi-task formulation acted as an **inductive regularizer**, encouraging the model to learn representations useful across related clinical objectives rather than fitting only the primary outcome labels.

The goal was to improve generalization to unseen patients under severe data limitations.

---

## Clinician-Guided Feature Development

A substantial part of the project involved close communication with medical experts to understand the clinical meaning of the available measurements.

Through these discussions, I worked to:

* identify clinically relevant prognostic factors,
* validate the interpretation of laboratory variables,
* distinguish potentially informative measurements from less relevant ones, and
* design additional derived features based on clinical knowledge.

This collaboration allowed domain knowledge to inform both data analysis and model development rather than treating the clinical dataset purely as a collection of numerical features.

---

## My Contributions

My primary contributions included:

* designing and evaluating an LSTM-attention model for longitudinal clinical data,
* identifying and correcting inconsistencies in outcome labels,
* introducing a correlated auxiliary task for multi-task learning,
* refining the training pipeline to improve generalization under data scarcity,
* collaborating with clinicians to identify important prognostic factors, and
* developing clinically informed features for model improvement.

---

## What I Learned

This project reinforced that real-world machine learning performance depends on much more than model architecture alone.

Data integrity, learning-objective design, domain knowledge, and careful validation can be as important as increasing model complexity, particularly when working with small and noisy clinical datasets.

It also gave me practical experience developing machine-learning systems under real-world data and engineering constraints.

---

## Availability

This project was conducted as part of proprietary industry research. Source code, internal datasets, and detailed experimental results are not publicly available.

## Research Topics

`Medical AI` · `Clinical Machine Learning` · `Time-Series Modeling` · `LSTM` · `Attention` · `Multi-Task Learning` · `Data Quality` · `Feature Engineering`
