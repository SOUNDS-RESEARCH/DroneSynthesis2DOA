# DroneSynthesis2DOA (Drone Signal Synthesis and Localization)

This project focuses on **Drone Localization and Tracking using Acoustic Arrays and Machine Learning Algorithms**.  
Due to the scarcity of real-world drone audio data, we simulate drone flight audio signals for training ML models, and later validate them using actual drone recordings.

---

## 📁 Project Structure

- `drone_generator/`: Generate synthetic drone audio
- `mic_array/`: Define and simulate microphone array configurations
- `data_generator/`: Generate microphone array data using drone + background signals
- `neural_doa/`: Neural DOA models' training and evaluation
- `doa_estimator/`: Traditional signal processing DOA estimators (e.g., SRP-PHAT)
- `evaluation/`: Performance evaluation and visualization for signal processing methods

---
## 🛠 Requirements

- Python 3.x  
- `numpy`, `scipy`, `matplotlib`, `soundfile`, `tqdm`, `pyyaml` 

```bash
pip install numpy scipy matplotlib soundfile tqdm pyyaml
```

## 🚀 Getting Started
Create a new experiment folder (such as exp1):
```bash
mkdir -p exps/exp1
```

## 1️⃣ Drone Signal Generation
Before running, copy the config file from drone_generator/exp_config.yaml to exps/exp1/
Edit exps/exp1/exp_config.yaml to adjust parameters such as flight duration, state, SNR, etc.
Run the following to generate synthetic drone signals:
```bash
python drone_generator/drone_generator_main.py --num 900 --exp exp1 --drone_data DroneAudioData
```
Output:
Synthetic drone mechanical sounds will be saved at args.drone_data (such as exps/exp1/DroneAudioData). Each sample contains:
- .wav file
- .json file describing the drone's flight trajectory and parameters

## 2️⃣ Microphone Array Configuration
Define your microphone arrays:
``` bash
python mic_array/mic_generator_main.py --yaml_file reusev301.yaml --exp exp1
```
Each microphone array configuration includes:
- Array type: tetra, octahedron, or individual
- Center position
- Rotation angles (x, y, z in degrees)
- White noise intensity
- Side length (for tetra)
Example configs: mic_config1.yaml, mic_config2.yaml, etc. Reusev301.yaml is an individual configuration with 8 microphones.  

Output:
exps/exp1/mic_config.json with microphone coordinates.

## 3️⃣ Data Generation
To generate the training Dataset:
``` bash
python data_generator/data_generator_main.py \
  --exp exp1 \
  --env_path env/Reusev301_train \
  --output MicArrayData \
  --drone_data DroneAudioData
```
To generate the testing Dataset, using different environment files and drone data (similar):
``` bash
python data_generator/data_generator_main.py \
  --exp exp1 \
  --env_path env/Reusev301_test \
  --output MicArrayDataTest \
  --drone_data DroneAudioDataTest
```
Please put your background noise folder in data_generator/env. The wav files must correspond to the microphone array you choose (e.g., 8 channels for octahedron and with the correct spatial coherence for this microphone array, best are real recordings with an actual array).
If --env_path is not provided, background noise is not included.
The output are multi-channel signals received by the simulated microphone array, saved at exps/args.exp/args.output.

## 4️⃣ Neural Network DOA Estimation
Train a DOA model, such as SELD_ACCDOA:
``` bash
python neural_doa/neural_main.py \
  --exp exp1 \
  --input_feature GCC_PHAT \
  --nn_model SELD_ACCDOA \
  --data_file MicArrayData \
  --data_file_test MicArrayDataTest \
  --is_training
```
**`input_feature` Options**
In addition to standard features, `input_feature` can also be:
- **`GCC_PHAT_mask`**  
  Applies a binary frequency mask (250 Hz–7000 Hz) in GCC-PHAT to retain relevant components and suppress noise.
- **`GCC_PHAT_beta`**  
  Adds a weighting factor **β** to GCC-PHAT to retain partial magnitude information and improve robustness in low-SNR conditions.  
  Default: **0.7**.
- **`GCC_PHAT_beta_mask`**  
  Combines both enhancements for improved performance.
  
Evaluate on real-world drone recordings. MicArrayDataReal30s is a real dataset provided by Fraunhofer IDMT (preprocessed).
[Insert download link here]. 
Please put it at exps/args.exp and run:
``` bash
python neural_doa/neural_main.py \
  --exp exp1 \
  --input_feature GCC_PHAT \
  --nn_model SELD_ACCDOA \
  --data_file_test MicArrayDataReal30s \
  --only_azi
```
PS: MicArrayDataReal30s only contains azimuth (azi) angle, no elevation.


## 5️⃣ Traditional DOA Estimation (SRP-PHAT)
Use signal processing methods for comparison:
```bash
python doa_estimator/doa_runner_main.py \
  --exp exp1 \
  --algorithm srp_phat \
  --dataset MicArrayDataTest
```
Optional flags, which are explained in the corresponding paper and 4️⃣ Neural Network DOA Estimation: 
```bash
--beta
--mask
```
`beta` means using the techique `GCC_PHAT_beta` in srp-phat and `mask` means using `GCC_PHAT_mask`.
Both options can also be used together for enhanced performance.

## 6️⃣ Evaluation for Traditional DOA Estimation Methods
Evaluate predictions vs ground truth on signal processing methods:
```bash
python evaluation/eval_runner_main.py \
  --exp exp1 \
  --eval_alg srp_phat \
  --dataset MicArrayDataTest
```

## 📂 Example Directory Structure
``` plaintext
exps/
└── exp1/
    ├── exp_config.yaml
    ├── mic_config.json
    ├── DroneAudioData/
    ├── DroneAudioDataTest/
    ├── MicArrayData/
    ├── MicArrayDataTest/
    └── MicArrayDataReal30s/
```

There is a Jupyter Notebook `exp_runner.ipynb` file so you can run the command codes easily.

## 📫 Contact
For any questions or contributions, feel free to reach out.
Maintainer: SherryYang1122

## 📝 Citation
If you use any files of this repository, please cite:

X. Yang, P. Naylor, S. Doclo, J. Bitzer, Neural Drone Localization Exploiting Signal Synthesis of Real-World Audio Data, in Proc. European Signal Processing Conference (EUSIPCO), Palermo, Italy, Sep. 2025, pp. 256-260.
