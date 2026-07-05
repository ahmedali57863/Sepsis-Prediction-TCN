# Early Sepsis Prediction using Temporal Convolutional Networks (TCN) 🩺📊

This repository contains the codebase and experimental framework for predicting the early onset of sepsis using deep learning. By leveraging Temporal Convolutional Networks (TCNs), this project analyzes complex, multi-variate time-series clinical data to identify early physiological indicators of sepsis.

## Project Overview

Sepsis is a life-threatening condition that requires rapid intervention. Traditional detection methods often rely on lagging indicators. This research project focuses on extracting temporal patterns from Electronic Health Records (EHR) to predict sepsis hours before clinical onset, improving patient outcomes through early warning systems.

## Key Features

* **Time-Series Analysis:** Designed to handle irregular and sparse clinical data streams.
* **TCN Architecture:** Utilizes causal and dilated convolutions for superior long-term memory retention compared to standard recurrent models.
* **Clinical Dataset:** Built and validated using the restricted-access MIMIC-IV database.

## Tech Stack

* **Language:** Python
* **Deep Learning Framework:** PyTorch
* **Data Processing:** Pandas, NumPy
* **Environment:** Google Colab (GPU Acceleration)
* **Version Control:** Git & GitHub

## Dataset Acknowledgment

This project utilizes the **MIMIC-IV** (Medical Information Mart for Intensive Care) dataset. Due to the strict Data Use Agreement (DUA) and patient privacy regulations, the raw clinical data is **not** hosted in this repository. Access must be independently requested and approved via PhysioNet.

## Project Team

* **Muhammad Ahmed** - Co-Developer & Researcher
* **Umama Khalid** - Co-Developer & Researcher

---
*Note: This repository is actively under development as part of an academic Final Year Project.*
