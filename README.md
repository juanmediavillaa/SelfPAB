# SelfPAB
Implementations of the SelfPAB, MonoSelfPAB, and LTA2V methods presented in our papers: "[SelfPAB: large-scale pre-training on accelerometer data for human activity recognition](https://doi.org/10.1007/s10489-024-05322-3)", "[Self-supervised Learning with Randomized Cross-sensor Masked Reconstruction for Human Activity Recognition](https://www.sciencedirect.com/science/article/pii/S0952197623016627)", and "[Long-term self-supervised learning for accelerometer-based sleep–wake recognition](https://doi.org/10.1016/j.engappai.2024.109758)", respectively.

## Preface: Access to HUNT4 pre-training data
The HUNT4 subset used for pre-training can be requested by contacting [kontakt\@hunt.ntnu.no](mailto:kontakt@hunt.ntnu.no?subject=HUNT4%20accelerometer%20snippets). Ask for the data used in the paper "Self-supervised Learning with Randomized Cross-sensor Masked Reconstruction for Human Activity Recognition".

## Requirements
[![git-lfs 3.4.1](https://img.shields.io/badge/Git_LFS-3.4.1-green)](https://git-lfs.com)
[![Python 3.8.10](https://img.shields.io/badge/Python_Versions-3.8_%7C_3.9_%7C_3.10-blue)](https://www.python.org/downloads/release/python-3810/)
```bash
pip install -r requirements.txt
```
__Note__: Per default the script tries to use a GPU to run the trainings. If no supported GPU can be allocated, a MisconfigurationException will occur. All the training scripts can be executed on the CPU instead by setting the "NUM_GPUS" parameter in the corresponding config.yml files to an empty list:
```bash
NUM_GPUS: []
```
## Download Datasets
Download the required datasets. Currently, the downstream datasets [HARTH](https://archive.ics.uci.edu/dataset/779/harth), [HAR70+](https://archive.ics.uci.edu/dataset/780/har70), [PAMAP2](https://archive.ics.uci.edu/dataset/231/pamap2+physical+activity+monitoring), [USC-HAD](https://sipi.usc.edu/had/) and [DualSleep](https://doi.org/10.18710/UGNIFE) are supported. The HUNT4 pre-training data can be requested by contacting [kontakt\@hunt.ntnu.no](mailto:kontakt@hunt.ntnu.no?subject=HUNT4%20accelerometer%20snippets), see Preface.
```bash
python download_dataset.py <dataset_name>
# Example (HARTH): python download_dataset.py harth
# Example (HAR70+): python download_dataset.py har70plus
# Example (PAMAP2): python download_dataset.py pamap2
# Example (USC-HAD): python download_dataset.py uschad
# Example (DualSleep): python download_dataset.py dualsleep
```
This command will download the given dataset into the [data/](https://github.com/ntnu-ai-lab/SelfPAB/tree/main/data) folder.

## Upstream Pre-training
After unzipping the pre-training HUNT4 data and storing it in `data/hunt4`, the upstream training can be started with:
```bash
python upstream.py -p <path/to/config.yml> -d <path/to/dataset>
# Example (SelfPAB): python upstream.py -p params/selfPAB_upstream/config.yml -d data/hunt4/
# Example (MonoSelfPAB): python upstream.py -p params/MonoSelfPAB_upstream/config.yml -d data/hunt4/
```
The [params/selfPAB_upstream/config.yml](https://github.com/ntnu-ai-lab/SelfPAB/blob/main/params/selfPAB_upstream/config.yml), [params/MonoSelfPAB_upstream/config.yml](https://github.com/ntnu-ai-lab/SelfPAB/blob/main/params/MonoSelfPAB_upstream/config.yml), and [params/LTA2V_upstream/config.yml](https://github.com/ntnu-ai-lab/SelfPAB/blob/main/params/LTA2V_upstream/config.yml) files contain all upstream model hyperparameters, presented in our papers, for SelfPAB, MonoSelfPAB, and LTA2V, respectively.

After upstream pre-training, the model with the best validation loss is stored as .ckpt file. This saved model can now be used as feature extractor for downstream training.
- The [parmas/selfPAB_upstream/upstream_model.ckpt](https://github.com/ntnu-ai-lab/SelfPAB/blob/main/params/selfPAB_upstream/upstream_model.ckpt) is the transformer encoder, which we pre-trained on 100,000 hours of unlabeled HUNT4 data using the "masked reconstruction" auxiliary task, as presented in our paper: Large-Scale Pre-Training for Dual-Accelerometer Human Activity Recognition.
- The [parmas/MonoSelfPAB_upstream/upstream_model.ckpt](https://github.com/ntnu-ai-lab/SelfPAB/blob/main/params/MonoSelfPAB_upstream/upstream_model.ckpt) is the transformer encoder, which we pre-trained on 100,000 hours of unlabeled HUNT4 data using the "randomized cross-sensor masked reconstruction (RCSMR)" auxiliary task, as presented in our paper: Self-supervised Learning with Randomized Cross-sensor Masked Reconstruction for Human Activity Recognition.
- The [parmas/LTA2V_upstream/upstream_model.ckpt](https://github.com/ntnu-ai-lab/SelfPAB/blob/main/params/LTA2V_upstream/upstream_model.ckpt) is the transformer encoder, which we pre-trained on 821,700 hours of unlabeled HUNT4 data using the "masked reconstruction" auxiliary task with long-term (30 hours) spectrograms and global positional encoding, as presented in our paper: Long-term self-supervised learning for accelerometer-based sleep–wake recognition.


## Downstream Training
A downstream leave-one-subject-out cross-validation (LOSO) / 5-fold cross-validation can be started with:
```bash
python loo_cross_validation.py -p <path/to/config.yml> -d <path/to/dataset>
# Example (SelfPAB, HARTH): python loo_cross_validation.py -p params/selfPAB_downstream_experiments/selfPAB_downstream_harth/config.yml -d data/harth/
# Example (MonoSelfPAB, HAR70+[Back]): python loo_cross_validation.py -p params/MonoSelfPAB_downstream_experiments/MonoSelfPAB_downstream_har70_B/config.yml -d data/har70plus/
# Example (MonoSelfPAB, PAMAP2): python loo_cross_validation.py -p params/MonoSelfPAB_downstream_experiments/MonoSelfPAB_downstream_pamap2/config.yml -d data/pamap2/
# Example (LTA2V, DualSleep): python loo_cross_validation.py -p params/LTA2V_downstream_experiments/LTA2V_downstream_DualSleep/config.yml -d data/dualsleep/
```
For HARTH, HAR70+, and DualSleep LOSOs are performed. For PAMAP2 and USC-HAD 5-fold cross-validations are performed with this command.
The downstream config has an argument called "upstream_model_path", defining the upstream model to use. In the [first example above](https://github.com/ntnu-ai-lab/SelfPAB/blob/main/params/selfPAB_downstream_experiments/selfPAB_downstream_harth/config.yml) the upstream model is set to be our pre-trained SelfPAB: [parmas/selfPAB_upstream/upstream_model.ckpt](https://github.com/ntnu-ai-lab/SelfPAB/blob/main/params/selfPAB_upstream/upstream_model.ckpt). In the [second example](https://github.com/ntnu-ai-lab/SelfPAB/blob/main/params/MonoSelfPAB_downstream_experiments/MonoSelfPAB_downstream_har70_B/config.yml) the upstream model is set to be our pre-trained MonoSelfPAB: [parmas/MonoSelfPAB_upstream/upstream_model.ckpt](https://github.com/ntnu-ai-lab/SelfPAB/blob/main/params/MonoSelfPAB_upstream/upstream_model.ckpt). In the [fourth example](https://github.com/ntnu-ai-lab/SelfPAB/blob/main/params/LTA2V_downstream_experiments/LTA2V_downstream_DualSleep/config.yml) the upstream model is set to be our pre-trained LTA2V: [parmas/LTA2V_upstream/upstream_model.ckpt](https://github.com/ntnu-ai-lab/SelfPAB/blob/main/params/LTA2V_upstream/upstream_model.ckpt).

## Results Visualization
The training results/progress can be logged in [Weights and Biases](https://wandb.ai/) by setting the WANDB argument in the config.yml files to True. For downstream training, the WANDB_KEY has to be provided in the config.yml file as well.

The LOSO results are stored as .pkl files in a params subfolder called loso_cmats. These can be visualized using the [loso_viz.ipynb](https://github.com/ntnu-ai-lab/SelfPAB/blob/main/loso_viz.ipynb) script.

## Embedding Generation
In case you are interested in investigating the latent representations generated by the respective upstream model, you can use the feature_extraction.py script:
```bash
python feature_extraction.py -p <path/to/downstream_config.yml> -d <path/to/dataset>
# Example (SelfPAB, HARTH): python feature_extraction.py -p params/selfPAB_downstream_experiments/selfPAB_downstream_harth/config.yml -d data/harth/
```
It will create the feature vectors of the given dataset batch-wise in a features.h5 file.

## Citation
If you use the [pre-trained SelfPAB upstream model](https://github.com/ntnu-ai-lab/SelfPAB/blob/main/params/selfPAB_upstream/upstream_model.ckpt) for your research, please cite the following paper:
```bibtex
@article{logacjovSelfPABLargescalePretraining2024,
  title = {{{SelfPAB}}: Large-Scale Pre-Training on Accelerometer Data for Human Activity Recognition},
  shorttitle = {{{SelfPAB}}},
  author = {Logacjov, Aleksej and Herland, Sverre and Ustad, Astrid and Bach, Kerstin},
  year = {2024},
  month = mar,
  journal = {Applied Intelligence},
  issn = {1573-7497},
  doi = {10.1007/s10489-024-05322-3},
  urldate = {2024-03-29},
  keywords = {Accelerometer,Human activity recognition,Machine learning,Physical activity behavior,Self-supervised learning,Transformer},
}
```

If you use the [pre-trained MonoSelfPAB upstream model](https://github.com/ntnu-ai-lab/SelfPAB/blob/main/params/MonoSelfPAB_upstream/upstream_model.ckpt) for your research, please cite the following paper:
```bibtex
@article{logacjovSelfsupervisedLearningRandomized2024,
  title = {Self-Supervised Learning with Randomized Cross-Sensor Masked Reconstruction for Human Activity Recognition},
  author = {Logacjov, Aleksej and Bach, Kerstin},
  year = {2024},
  month = feb,
  journal = {Engineering Applications of Artificial Intelligence},
  volume = {128},
  pages = {107478},
  issn = {0952-1976},
  doi = {10.1016/j.engappai.2023.107478},
  urldate = {2023-11-15},
  keywords = {Accelerometer,Human activity recognition,Machine learning,Representation learning,Self-supervised learning,Transformer}
}
```

If you use the [pre-trained LTA2V upstream model](https://github.com/ntnu-ai-lab/SelfPAB/blob/main/params/LTA2V_upstream/upstream_model.ckpt) for your research, please cite the following paper:
```bibtex
@article{logacjovLongtermSelfsupervisedLearning2025,
  title = {Long-Term Self-Supervised Learning for Accelerometer-Based Sleep--Wake Recognition},
  author = {Logacjov, Aleksej and Bach, Kerstin and Mork, Paul Jarle},
  year = {2025},
  month = feb,
  journal = {Engineering Applications of Artificial Intelligence},
  volume = {141},
  pages = {109758},
  issn = {0952-1976},
  doi = {10.1016/j.engappai.2024.109758},
  urldate = {2024-12-09},
  keywords = {Accelerometer,Machine learning,Representation learning,Self-supervised learning,Sleep-wake recognition,Transformer}
}
```
