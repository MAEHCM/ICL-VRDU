## ICL-D3IE: In-Context Learning with Diverse Demonstrations Updating for Document Information Extraction

## Table of contents
* [Overview](#overview)
* [Installation](#installation)
* [Datasets](#datasets)
* [Tuning and Testing](#tuning-and-testing)
* [Results](#results)


We propose a simple but effective in-context learning framework called ICL-D3IE, which enables LLMs to perform DIE with different types of demonstration examples. 

![](https://user-images.githubusercontent.com/111342294/223765044-5fcfc41b-0b5f-4b56-bd64-9aeefba39791.png)


# Installation

Installation for Project，if you need to study the robustness of the model to text shift, you need to install [Textattack](https://github.com/QData/TextAttack)

```
git clone https://anonymous.4open.science/r/ICL-D3IE-B1EE && cd ICL-D3IE
```

# Datasets

| Dataset | Link      |
|:--------:| :------------:|
| FUNSD | [download](https://www.kaggle.com/datasets/aravindram11/funsdform-understanding-noisy-scanned-documents)|
| CORD | [download](https://github.com/clovaai/cord)|
| SROIE | [download](https://www.kaggle.com/datasets/urbikn/sroie-datasetv2)|


### Preprocess datasets

First generate strong and weak semantic entities and get the following files , `/weak_other_map` , `/strong_answer_map` , `/strong_question_map` , `/weak_Q_map`   , `/weak_A_map`,We provide five strong and weak semantic entity libraries extracted from our shuffle layout method on the FUNSD test set for five different pre-training models ,You can choose to fill in `v3`, `v2`, `v1`, `bros` or `lilt` in { } and execute the following code
```
python map_{ }_funsd_L.py
```

Then modify the file path to generate FUNSD-L test data , which is saved in the `mix_test.txt` , you can modify the number of rows and columns generated by the layout, the size of the bounding box, the probability of random filling, and the number of documents generated

```
generate_ood_data("mix_test.txt", "/strong_question_map",
                  "/strong_answer_map", "/weak_Q_map",
                  "/weak_A_map", "/weak_other_map",50)
```

```
python gen_ood_mix.py
```

![](https://user-images.githubusercontent.com/111342294/202719602-47a09c21-0226-4221-9652-6d714b4a4a46.png)


### Generate CDIP-L

To facilitate use, we separately place it in the main directory, and adjust two parameters: lamda1 controls the horizontal distance, and lamda2 controls the vertical distance. We use the priority order of consolidation: horizontal first and then vertical

```
python merge_layout.py
```

![](https://user-images.githubusercontent.com/111342294/202724209-b915d944-dd62-4e77-a66e-781bc4b4a707.png)


### Generate CDIP-I<sub>1</sub>

Separate text pixels and non text pixels in the document, and then overlay them into the natural scene [MSCOCO](https://cocodataset.org/#home)

```
python python mixup_image.py
```

![](https://user-images.githubusercontent.com/111342294/202724449-f8ee8ffd-c8aa-4dd7-b665-1a6558b5e7aa.png)

### Generate CDIP-I<sub>2</sub>

Using pre-trained [DocGeoNet](https://github.com/fh2019ustc/DocGeoNet)(specific process reference), a forward propagation calculation of the normal document image is performed to get the distorted image, and then OCR again

```
python inference.py
```

![](https://user-images.githubusercontent.com/111342294/202726278-e0a89790-494e-46a6-8a2e-42009f2dfce4.png)


# Tuning and Testing

### Tuning

Select the model used and the task fill, the first { } select `v3`, `v2`, `v1`, `bros` or `lilt`, the second {} select`funsd` or `cdip`

Finetune your own LayoutLMv3 model or download our finetuned model [download](https://pan.baidu.com/s/1zwlTvQsJfQDVOo2UDMRgRA)，Select models and tasks to use

```javascript
python -m torch.distributed.launch --nproc_per_node --use_env finetune_{ }_{ }.py --config config.yaml --output_dir
```

For VQA tasks, use the command line alone，fill in the selected model at { }

```javascript
python docvqa_{}_main.py
```

### Testing

Select the model used and the task fill, the first { } select `v3`, `v2`, `v1`, `bros` or `lilt`, the second { } select`funsd` or `cdip` ， modify the following parameters to perform a shift operation on a mode.`--text_aug`,`--image_aug`,`--aut_layout`
```
--text_aug={'WordSwapMaskedLM','WordSwapEmbedding','WordSwapHomoglyphSwap','WordSwapChangeNumber','WordSwapRandomCharacterDeletion'} , --image_aug=True/False , --aug_layout=True/False
```

```
python demo_{ }_ood_{ }.py
```

# Results

### The ID and OOD performance of the existing models
![image](https://user-images.githubusercontent.com/111342294/202836388-7e251a9c-ad73-4d16-bfc3-a0d6ba11f9dc.png)

### Visualize the results

