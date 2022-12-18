# Crop Classification

## Setting the Environment
```
conda create --name aidea python=3.9.15
conda activate aidea
pip install -r requirements.txt
```

## Download the Checkpoint 
You can download the checkpoints from Google Drive: https://drive.google.com/drive/folders/1qCnjAqN5TmdW-kP0tgxRGA4cRyJAJgT7?usp=share_link

## How to Train the Model
* There are some default models in the file `net.py`
* Example: run `source train.sh ../config/regnet`
  * Note: You need to prepare datalist and  data first, and modify the path in the config file


### The Structure of YAML
```
name: swin_s_1080 (Related to folder name of log and checkpoint)
savepath: v4 (Related to folder name of log and checkpoint )
epoches: 150
strategy: ddp
accumulate_batches: 1
precision: 16
gradient_clip_val: 5.0
data_config:
  batch_size: 16
  val_batch_size: 64 (If you have a small gpu memory, you can reduce the number of batch size when you are doing the inference)
  dataroot: /neodata/pathology_breast/aidea/dataset/ (Dataroot is expected to be linked to the front of the image path in datalist)
  datalist: /neodata/pathology_breast/aidea/crop_code/datalist/fold_0.json
  cache: false （true if your ram > 700GB）
  transforms_config:
    img_size: 1080
    random_transform: true （auto augmentation）
    random_rotation: false
model_config:
  model:
    num_classes: 33
    backbone: swin_s
    backbone_num_features: 1000
  loss: FL (Focal loss)
  use_additional_loss: false (Augular Loss. It is not used in current process)
  optimizer: adamw
  lr: 0.0001
  scheduler:
    name: cosine
    T_max: 50
    eta_min: 0
ckpt: ../checkpoint/swin_s_1080.ckpt (checkpoint/model weight path)
```
### The Structure of Datalist
Datalist is a JSON file, which is a `dict` format. The `dict` has 3 keys: "training", "validation" and "test". The value corresponding to each key is `list`, which has many `dict`, and each `dict` contains the image path and label.
```python
{
    "training": [
        {
            "image": "asparagus/000b43a3-d331-47ad-99a4-4c0fa9b48298.jpg",
            "label": 0
        },
        {
            "image": "asparagus/00172189-3156-48d7-bb1e-f0a922bc54b8.jpg",
            "label": 0
        },
        ...
```

## How to Make Inference (Reproduce)
* Setup the environment

* Prepare the config file to be used during training (`.yaml`).
  * Give the config file path in `infer_public.py`(Note that there is a line in infer_public that needs to be modified for usage. `data_list = json.load(open("../datalist/public_private.json"))` Please give the correct JSON file for the data to be inferred)
  * The JSON file is `list` structure. Each element is `dict` structure in list in the format {"image": image_path}.
    * image_path should be replaced by the **absolute path**.
    * The example datalist is in the `datalist` folder. (public_private.json)
* Please specify the checkpoint path you want in the config file
* Example: run `source reproduce.sh` (You can adjust which GPU to use in `infer_public.sh`)
* The inference time required for each model varies, with estimates ranging from approximately 10 to 30 minutes.
* All of the results (the results of the **four** models on the public and private data) will be stored in a `private` folder. For this competition, we used an ensemble method, which requires these four CSV files.  

## Contact
Questions about the actions of this code can be directed to: rex19981002@gmail.com

## Citation
```
@misc{
    title  = {crop_classification},
    author = {Jia-Wei Liao, Yen-Jia Chen, Yi-Cheng Hung, Jing-En Hung, Shang-Yen Lee},
    url    = {https://github.com/yenjia/AIdea_crops},
    year   = {2022}
}
```