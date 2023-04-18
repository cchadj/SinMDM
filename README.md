# SinMDM: Single Motion Diffusion

Please visit our [project page](https://sinmdm.github.io/SinMDM-page/) for more details.

## Setup

This code has been tested in the following environment:
* Ubuntu 18.04.5 LTS
* Python 3.8
* conda3 or miniconda3
* CUDA capable GPU (one is enough)

Setup conda env:
```shell
conda env create -f environment.yml
conda activate SinMDM
```

Install ganimator-eval-kernel by following [these](https://github.com/PeizhuoLi/ganimator-eval-kernel) instructions,  OR by running: 
```shell 
pip install git+https://github.com/PeizhuoLi/ganimator-eval-kernel.git
```

## Preparations

### Get Data

Data should be under the `./dataset` folder.

<details>
  <summary><b>Mixamo Dataset</b></summary>

Download the motions used for our benchmark:
```shell
bash prepare/download_mixamo_dataset.sh
```

Or download motions directly from [Mixamo](https://www.mixamo.com/#/) and use `utils/fbx2bvh.py` to convert fbx files to bvh files.

</details>

<details>
  <summary><b>HumanML3D Dataset</b></summary>

Clone HumanML3D, then copy the data dir to our repository:

```shell
cd ..
git clone https://github.com/EricGuo5513/HumanML3D.git
unzip ./HumanML3D/HumanML3D/texts.zip -d ./HumanML3D/HumanML3D/
cp -r HumanML3D/HumanML3D sin-mdm/dataset/HumanML3D
cd sin-mdm
```

Then, download the motions used for our benchmark: 
```shell
bash prepare/download_humanml3d_dataset.sh
```
Or download the entire dataset by following the instructions in [HumanML3D](https://github.com/EricGuo5513/HumanML3D.git),
then copy the result dataset to our repository:

```shell
cp -r ../HumanML3D/HumanML3D ./dataset/HumanML3D
```
</details>

<details>
  <summary><b>Truebones zoo Dataset</b></summary>

Download motions used in our pretrained models:
```shell
bash prepare/download_truebones_zoo_dataset.sh
```

Or download the full dataset [here](https://truebones.gumroad.com/l/skZMC).
</details>


## Synthesis
### Preparations
#### Download pretrained models

Download the model(s) you wish to use using the scripts below. The models will be placed under `./save/`.

<details>
  <summary><b>Mixamo Dataset Models</b></summary>

Download pretrained models used for our benchmark:
```shell
bash prepare/download_mixamo_models.sh
```
</details>

<details>
  <summary><b>HumanML3D Dataset Models</b></summary>

Download pretrained models used for our benchmark:
```shell
bash prepare/download_humanml3d_models.sh
```

</details>

<details>
  <summary><b>Truebones Zoo Dataset Models</b></summary>

Download pretrained models:
```shell
bash prepare/download_truebones_models.sh
```

Pretrained model of "Flying Dragon" will be available soon!

</details>

### Run synthesis command

To generate motions using a pretrained model use the following command: 
```shell
python -m sample.generate --model_path ./save/path_to_pretrained_model --num_samples 5 --motion_length 10
```

Where `--num_samples` is the number of motions that will be generated and `--motion_length` is the length in seconds. Use `--seed` to specify a seed.

**Running this will get you:**

* `results.npy` file with xyz positions of the generated animation
* `rows_00_to_##.mp4` - stick figure animations of all generated motions.
* `sample##.bvh` - bvh file for each generated animation that can be visuallized using [blender](https://www.blender.org/download/).

It will look something like this:

<img alt="example" src="assets/example_generate.gif" width="40%"/>


## Training

### Preperations

Download HumanML3D dependencies:

```shell
bash prepare/download_t2m_evaluators.sh
```


### Run training command

```shell
python -m train.train_sinmdm --arch qna --dataset humanml --save_dir <path_to_save_models>  --sin_path <'path to .bvh file for mixamo/bvh_general dataset or .npy file for humanml dataset'>
```

* Specify architecture using `--arch` Options: qna, unet  
* Specify dataset using `--dataset` Options: humanml, mixamo, bvh_general
* Use `--device` to define GPU id.
* Use `--seed` to specify seed.
* Add `--train_platform_type {ClearmlPlatform, TensorboardPlatform}` to track results with either [ClearML](https://clear.ml/) or [Tensorboard](https://www.tensorflow.org/tensorboard).
* Add `--eval_during_training` to run a short evaluation for each saved checkpoint.
* Add `--gen_during_training` to synthesize a motion and save its visualization for each saved checkpoint.

  Evaluation and generation during training will slow it down but will give you better monitoring.

Please refer to file utils/parser_util.py for more arguments.

## Applications
### In-betweening
For in-betweening, the prefix and suffix of a motion are given as input and the model generates the rest according to the motion the network was trained on.
```shell
python -m sample.edit --model_path <path_to_pretrained_model> --edit_mode in_betweening --num_samples 3
```
* To specify the motion to be used for the prefix and suffix use `--ref_motion <path_to_reference_motion>`. If `--ref_motion` is not specified, the original motion the network was trained on will be used.
* Use `--prefix_end` and `--suffix_start` to specify the length of the prefix and suffix
* Use `--seed` to specify seed.
* Use `--num_samples` to specify number of motions to generate.

For example:

<img alt="example" src="assets/inbetween.gif" width="80%"/> 

generated parts are colored in an orange scheme, given input is colored in a blue scheme.


### Motion expansion
For motion expansion, a motion is given as input and new prefix and suffix are generated for it by the model.

```shell
python -m sample.edit --model_path <path_to_pretrained_model> --edit_mode expansion --num_samples 3
```
* To specify the input motion use `--ref_motion <path_to_reference_motion>`. If `--ref_motion` is not specified, the original motion the network was trained on will be used.
* Use `--prefix_length` and `--suffix_length` to specify the length of the generated prefix and suffix
* Use `--seed` to specify seed.
* Use `--num_samples` to specify number of motions to generate.

For example:

<img alt="example" src="assets/expansion.gif" width="80%"/> 

generated parts are colored in an orange scheme, given input is colored in a blue scheme.



### Lower body editing
The model is given a reference motion from which to take the upper body, and generates the lower body according to the motion the model was trained on.

```shell
python -m sample.edit --model_path <path_to_pretrained_model> --edit_mode lower_body --num_samples 3 --ref_motion <path_to_reference_motion>
```
This application is supported for the `mixamo` and `humanml` datasets.
* To specify the reference motion to take upper body from use `--ref_motion <path_to_reference_motion>`. If `--ref_motion` is not specified, the original motion the network was trained on will be used.
* Use `--seed` to specify seed.
* Use `--num_samples` to specify number of motions to generate.

For example: (reference motion is "chicken dance" and lower body is generated with model train on "salsa dancing")

<img alt="example" src="assets/lower_body_example.gif" width="50%"/> 

generated lower body is colored in an orange scheme. Upper body which is given as input is colored in a blue scheme.

### Upper body editing
Similarly to lower body editing, use `--edit_mode upper_body`

```shell
python -m sample.edit --model_path <path_to_pretrained_model> --edit_mode upper_body --num_samples 3 --ref_motion <path_to_reference_motion>
```
This application is supported for the `mixamo` and `humanml` datasets.

For example: (reference motion is "salsa dancing" and upper body is generated with model trained on "punching")

<img alt="example" src="assets/upper_body_example.gif" width="50%"/> 

generated upper body is colored in an orange scheme. Lower body which is given as input is colored in a blue scheme.


### Harmonization
You can use harmonization for style transfer. The model is trained on the style motion. The content motion `--ref_motion`, unseen by the network, is given as input and adjusted such that it matches the style motion's motifs.

```shell
python -m sample.edit --model_path <path_to_pretrained_model> --edit_mode harmonization --num_samples 3 --ref_motion <path_to_reference_motion>
```

* To specify the reference motion use `--ref_motion <path_to_reference_motion>`
* Use `--seed` to specify seed.
* Use `--num_samples` to specify number of motions to generate.

For example, here the model was trained on "happy" walk, and we transfer the "happy" style to the input motion:

<img alt="example" src="assets/harmonization_example.gif" width="90%"/> 



## Evaluation


### Preparations

**HumanML3D**

```shell
bash prepare/download_t2m_evaluators.sh
bash prepare/download_humanml3d_dataset.sh
bash prepare/download_humanml3d_models.sh
```

**Mixamo**

```shell
bash prepare/download_mixamo_dataset.sh
bash prepare/download_mixamo_models.sh
```

### Run evaluation command
To evaluate a single model (trained on a single sequence), run:

**HumanML3D**

```shell
python -m eval.eval_humanml --model_path ./save/humanml/0000/model000019999.pt
```

**Mixamo**

```shell
python -m eval.eval_mixamo --model_path ./save/mixamo/0000/model000019999.pt
```

### Run evaluation benchmark
Coming soon.

## Acknowledgments

Our code partially uses each of the following works. We thank the authors of these works for their outstanding contributions and for sharing their code.

[MDM](https://github.com/GuyTevet/motion-diffusion-model), [QnA](https://github.com/moabarar/qna), [Ganimator](https://github.com/PeizhuoLi/ganimator), [Guided Diffusion](https://github.com/openai/guided-diffusion), [A Deep Learning Framework For Character Motion Synthesis and Editing](http://theorangeduck.com/page/deep-learning-framework-character-motion-synthesis-and-editing).

## License
This code is distributed under the [MIT LICENSE](LICENSE).

