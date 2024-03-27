## RIP-AV: Joint Representative Instance Pre-training and Context Aware Network for Retinal Artery/Vein Segmentation

> ### :o: Contribution:
>
> :arrow_forward:<em>The first attempt to joint the <b>Representative Instance Pre-training</b> (RIP) with the contextual analysis of local patch for <b>retinal A/V segmentation</b>.</em>
>
> :arrow_forward:<em>The RIP task is proposed to learn <b>latent arteriovenous features</b> from diverse spatial regions.</em>
>
> :arrow_forward:<em><b>Distance Aware</b> and <b>Patch Context Fusion</b> modules are desgined to explore the relationship of patch and its context base on the sense of vascular structures.</em>
>
> :arrow_forward:<em>Validated against three datasets, outperforming state-of-the-art methods. </em>

<img align='center' src='fig/Fig_1_v10_300dpi.png' height=80% width=80%/>

## &#x1F527; Usage

###  :zap: Dependencies

[![dada](https://img.shields.io/badge/python-==3.10-blue.svg)]()

[![dada](https://img.shields.io/badge/torch-==1.13.1-red.svg)]()

[![dada](https://img.shields.io/badge/tensorflow-==2.9.1-yellow.svg)]()

[![dada](https://img.shields.io/badge/CUDA->=11.7-green.svg)]()

Conda environment settings:

 ```shell
 conda create -n rip_av python=3.10 -y
 conda activate rip_av
 pip install poetry 
 poetry install
 pip install tensorflow==2.9.1
 pip install torch==1.13.1 torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu117 --force
 ```

## :rocket: Preparing Segmentation Datasets

Download the public datasets: [AV-DRIVE](https://drive.google.com/drive/folders/1SdOyBnN91hOPMNuw69_r21yJaXgsiohT?usp=drive_link), [LES-AV](https://drive.google.com/drive/folders/1VMugPO4PwtrJtjdqYJKwcM1Zozc-CsDV?usp=drive_link), HRF ([v1](https://drive.google.com/drive/folders/1A_OFBTydx5y_XuazaypUp4C9ezS3R-gR?usp=drive_link), [v2](https://drive.google.com/drive/folders/11ZSXwxxhJ02KuKWdEg_PhAocu2UqZ5NR?usp=drive_link)) and put in the `./data`

```
├─dataset

​	├─test 
​	│  ├─av
​	│  └─images
​	└─training
​    	├─av
​    	└─images
```

## :sunny: Training stage

There are two stage: RIP-stage and segmentation stage.

### :one: RIP Stage

Run `Preprocessing/generate_patch_selection_for_RIP.py` to generate <b>representative patches</b> for **RIP stage**, or directly use the pretrained weights.

```
python generate_patch_selection_for_RIP.py 
	--dataset_path <dataset path>
	-- train_or_test <training|test>
    --out <output path>
cd ./RIP/
python train.py
```


$$
{{\mathcal{L}}_{RIP}}=-\frac{1}{N}\sum\limits_{i=1}^{N}{\sum\limits_{j=1}^{C}{\left[ {{y}_{ij}}\cdot \log (m({{p}_{i}}))+(1-{{y}_{ij}})\cdot \log (1-m({{p}_{i}})) \right]}}
$$


| Dataset  | Pretrain weight                                              |
| -------- | ------------------------------------------------------------ |
| AV-DRIVE | [RIP_pretrain_drive](https://drive.google.com/file/d/1ofz2WNXCVfpjXGvoGzneS5PVb7mAhcIG/view?usp=sharing) |
| LES-AV   | [RIP_pretrain_les](https://drive.google.com/file/d/1EB0ZKa3-9yCq9_RgcfFaU7Ut3SPLdXya/view?usp=sharing) |
| HRF      | [RIP_pretrain_hrf](https://drive.google.com/file/d/1eyqpM0p5hmBhk1jze782UxLtETYuCNg6/view?usp=sharing) |

#### :rainbow: Tsne visualization results (W/O RIP)

<img align='center' src='fig/drive_tsne.svg' height=80% width=80%/>

<p align='center' style='margin-top:0px;margin-bottom:0px;'><b>AV-DRIVE</b></p>

<img src='fig/les_tsne.svg' height=80% width=80%/>

<p align='center'><b>LES-AV</b></p>

<img  src='fig/hrf_tsne.svg' height=80% width=80%/>

<p align='center'><b>HRF</b></p>

### :two:Segmentation Stage

```shell
# train and evaluate  
cd AV/
python main config/config_train_general

# change config file and run test with visulization
python test_with_vis.py

```

$$
{{\mathcal{L}}_{da}}(DA({{f}_{d}}),S(y))=\frac{1}{\sum\limits_{i=1}^{N}{y}}\sum\limits_{j=1}^{N}{\frac{1}{S({{y}_{j}})+\varepsilon }}\cdot smoot{{h}_{{{L}_{1}}}}(DA({{f}_{dj}}),S({{y}_{j}})) \\
smoot{{h}_{{{L}_{1}}}}(x,y)=\left\{ \begin{array}{*{35}{l}}
   0.5{{(x-y)}^{2}}, & \text{if }|x-y|<0  \\
   |x-y|-0.5, & \text{otherwise }  \\
\end{array} \right.
$$

$$
\mathcal{L}_{adv}^{D}={{\mathbb{E}}_{x,y}}\left[ -y\log (1-D(x,G(x))) \right]+{{\mathbb{E}}_{x,y}}\left[ -y\log (D(x,y)) \right]\\
\mathcal{L}_{adv}^{G}={{\mathbb{E}}_{x,y}}\left[ -y\log D(x,G(x)) \right]\\
{{\mathcal{L}}_{s}}=\sum\limits_{i=1}^{3}{{{\beta }_{i}}({{\mathcal{L}}_{bce}}(G(x),y)+{{\mathcal{L}}_{dice}}(G(x),y)})\\
\begin{align}
  & {{\mathcal{L}}_{G}}={{\lambda }_{a}}\mathcal{L}_{adv}^{G}+{{\lambda }_{s}}{{\mathcal{L}}_{s}}+{{\lambda }_{d}}{{\mathcal{L}}_{da}} \\ 
 & {{\mathcal{L}}_{D}}={{\lambda }_{a}}\mathcal{L}_{adv}^{D} \\ 
\end{align}
$$

where the hyper-parameters ${{\lambda }_{a}}$, ${{\lambda }_{s}}$, ${{\lambda }_{d}}$ are meticulously calibrated to balance these three losses and set as 0.01, 5 and 1, respectively, see `./AV/config/config_train_general.py`.

| Dataset  | checkpoint                                                   |
| -------- | ------------------------------------------------------------ |
| AV-DRIVE | [RIP_checkpoint_drive](https://drive.google.com/file/d/1AUtWDTS6LutsHpB_hHfopUFqEDa-JUnT/view?usp=sharing) |
| LES-AV   | [RIP_checkpoint_les](https://drive.google.com/file/d/1iFWqhxsnOH3h-j1CKJ7NPL_AmdM7lTjm/view?usp=sharing) |
| HRF      | [RIP_checkpoint_hrf](https://drive.google.com/file/d/1a21dNW92n-YAACtL_blJHcwkSshKSABC/view?usp=sharing) |

#### :rainbow: Segmentation Performance

<img align='center' src='fig/DRIVE_LES_HRF_combine_vis_text_enlarge_v3.png' height=80% width=80%/>

Full prediction results can be available at [AV-DRIVE](https://drive.google.com/drive/folders/1sKliquaxXwesrzlWAsXP5xjNaWpJXAuL?usp=sharing), [LES-AV](https://drive.google.com/drive/folders/1djAWENC-T7C7sCk9UjR0qZG18XuFyrTl?usp=sharing), [HRF](https://drive.google.com/drive/folders/1f3_5LjwGZRaDp4sOEZyQoyzHvolJFkIN?usp=sharing).

### :memo:To-Do List

- [x] training and test code
- [ ] visualization demo
- [ ] test on more retinal dataset

