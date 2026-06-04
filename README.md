# GHOST: Hierarchical Sub-Goal Policies for Generalizing Robot Manipulation

[Project page](https://ghost-human-demo.github.io/) · Robotics: Science and Systems (RSS) 2026

<p align="center">
  <img src="ghost-rss26.gif" width="100%" alt="GHOST overview" />
</p>

This repository contains the **high-level sub-goal policy** (training and evaluation). The low-level Diffusion Policy controller and real-robot deployment live in the separate [r-pad/lerobot](https://github.com/r-pad/lerobot/) repository.

## Installation

This project uses [pixi](https://pixi.sh/latest/) for dependency management.

```bash
# avoid LeRobot install errors
export CPPFLAGS="-I/usr/include"
export CFLAGS="-DHAVE_LINUX_INPUT_H"

pixi install
pixi run install-deps
pixi shell
```

Download the [MANO models](https://mano.is.tue.mpg.de/) and place them in `mano/`.

## Data preparation

This repo depends on data collected from the [r-pad/lerobot](https://github.com/r-pad/lerobot/) repo, stored as a `LeRobotDataset` and postprocessed with `upgrade_dataset.py` (from that repo) to produce a goal-conditioned dataset for training.

## Training

Train the high-level sub-goal policy (GHOST uses the `dino_3dgp` model):

```bash
python scripts/train.py model=dino_3dgp dataset=rpadLerobot \
    dataset.repo_id='["sriramsk/fold_onesie_human_MV_20260119_ss_hg"]' \
    dataset.cache_dir=/path/to/cache \
    resources.num_workers=32 \
    training.check_val_every_n_epochs=3
```

> **Note:** The first epoch is typically slow because each sample is processed and cached to `dataset.cache_dir`. Later epochs — and any reruns pointing at the same `cache_dir` — are much faster.

Pass multiple repos in the `dataset.repo_id` list to train on several tasks at once. `model=mimicplay` and `model=articubot` are also available.

The best checkpoint is uploaded to WandB at the end of training. To upload an intermediate checkpoint stored in `logs/`:

```bash
cd scripts/
python upload_wandb.py --run_id <wandb-run-id> --checkpoint_path <path/to/checkpoint>
```

## Evaluation

```bash
python scripts/eval.py checkpoint.run_id=<wandb-run-id> dataset.data_dir=<path/to/dataset/>
```

If the model was trained on the cluster, override `dataset.cache_dir` and set it to `null`.

Per-episode evaluation on LeRobot:

```bash
python scripts/eval_lerobot_episode.py checkpoint.run_id=<wandb-run-id> \
    dataset=rpadLerobot dataset.repo_id="[<your/lerobot_repo_goal>]" \
    model=dino_heatmap checkpoint.type=pix_dist
```


<details>
<summary><b>MuJoCo calibration</b> (only needed for rendering robot-gripper trajectories)</summary>

- The default path to a MuJoCo robot model is `~/.cache/robot_descriptions/mujoco_menagerie/{ROBOT}` (e.g. `.../aloha`).
- Copy the robot directory from that cache into the local `robot_descriptions/` folder.
- Edit the robot description XML (e.g. `aloha.xml`) to match your real-world robot's geometry, joint limits, and transforms — for example, adjust `"left/base_link"` and `"right/base_link"` to match your setup.

</details>

## Citation

```bibtex
@inproceedings{krishna2026ghost,
  title     = {GHOST: Hierarchical Sub-Goal Policies for Generalizing Robot Manipulation},
  author    = {Krishna, Sriram and Eisner, Ben and Zhan, Haotian and Yuan, Ying and
               Zhen, Haoyu and Gan, Chuang and Tulsiani, Shubham and Held, David},
  booktitle = {Robotics: Science and Systems (RSS)},
  year      = {2026}
}
```

## Acknowledgements

This codebase was adapted from [TAX3D](https://github.com/ey-cai/non-rigid/) and [python-ml-project-template](https://github.com/r-pad/python_ml_project_template/).
