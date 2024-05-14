# FreSh - Video experiments

This repository is a modified version of [ResFields](https://github.com/markomih/ResFields),
containing Video experiments testing the FreSh method. If you wany to do something else than
reproducing our experiments, please refer to the original repository. 
The main FreSh repository can be found \[here\](TODO).

## Setup

### Code

See [the original repository](https://github.com/markomih/ResFields). 

## Experiments

Below you can find the commands needed to run the experiments.

Save model outputs at initialisation (commands below assume you are using slurm array jobs):
```bash
export sequence="skvideo.datasets.bikes"
export sequence="../DATA_ROOT/Video/cat.mp4"

# Siren
OMEGA=$((SLURM_ARRAY_TASK_ID * 10))
python launch.py --config ./configs/video/base.yaml --train --predict \
 dataset.video_path=$sequence --exp_dir ../model_outputs  model.resfield_layers=[1,2,3] \
 model.omega=$OMEGA tag="siren_{$OMEGA}"   \
 save_outputs=True model.disable_time=True

# Fourier
python launch.py --config ./configs/video/base_relu.yaml --train --predict \
 dataset.video_path=$sequence --exp_dir ../model_outputs/ \
 model.resfield_layers=[1,2,3] \
  model.sigma=$SLURM_ARRAY_TASK_ID \
 tag="fourier_{$SLURM_ARRAY_TASK_ID}" save_outputs=True \
  model.uniform_init=True \
 model.positional_encoding=False  model.disable_time=True
```

Run the FreSh method (you need the script from the main FreSh repository):
```bash
python <path_to_fresh>/scripts/find_optimal_config.py \
  --dataset model_outputs/<dataset_name>.npy  \
  --model_output model_outputs/...  \
  --results_root wasserstein_results/example
```
You will find the configurations selected by FreSh in `wasserstein_results/example/wasserstein_best.csv`.
For an additional description of using the script see the main FreSh repository.

Train a model:
```bash
export sequence="../DATA_ROOT/Video/cat.mp4"
#export sequence="skvideo.datasets.bikes"

python launch.py --config ./configs/video/base_relu.yaml --train --predict \
 dataset.video_path=$sequence --exp_dir ../results/ \
 model.resfield_layers=[1,2,3] seed=$SLURM_ARRAY_TASK_ID \
 tag="positional_encoding"  model.uniform_init=False \
  model.positional_encoding=True  model.disable_time=True

export OMEGA=30
python launch.py --config ./configs/video/base.yaml --train --predict \
 dataset.video_path=$sequence \
 --exp_dir ../results \
 model.resfield_layers=[1,2,3] \
 model.omega=$OMEGA seed=$SLURM_ARRAY_TASK_ID  \
 tag="siren"  model.disable_time=True

export sigma=1
python launch.py --config ./configs/video/base_relu.yaml --train --predict \
 dataset.video_path=$sequence  --exp_dir ../results  model.resfield_layers=[1,2,3] seed=$SLURM_ARRAY_TASK_ID \
 model.sigma=$sigma  tag="fourier" model.hidden_features=$rff model.uniform_init=True \
  model.disable_time=True
```

