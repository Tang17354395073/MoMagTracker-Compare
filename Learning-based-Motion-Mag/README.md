### Environment setup
```
# pytorch installation
   pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
   pip install pillow tqdm matplotlib scipy tensorboard opencv-python
```

### Training
1. Download the dataset at https://drive.google.com/drive/folders/17b4aanuXwtdlIFMdfY1xynd4h47HDVS7?usp=sharing and unzip.
2. Enter the following command.
```
   python main.py --phase="train" --checkpoint_path="Path to the model.tar" --data_path="Path to the directory where the training data is located"
```

There are various modes for inference in the motion magnification method. Each mode can branch as follows:

    ├── Inference
    │   ├── Without a temporal Filter
    │   │   ├── Static
    │   │   ├── Dynamic
    │   ├── With a temporal filter   
    │   │   ├── differenceOfIIR
    │   │   ├── butter
    │   │   ├── fir

### For the inference without a temporal filter
the static mode amplifies small motion based on the first frame, while the dynamic mode amplifies small motion by comparing the current frame to the previous frame.

1) get the baby video and split into multi frames. When using a custom video, also split it into multiple frames.

       mkdir babyframes 2>nul & ffmpeg -i baby.mp4 -f image2 babyframes/%06d.png

2) And then run the static mode. Add "--velocity_mag" for dynamic mode.

       python main.py  --checkpoint_path "./model/epoch50.tar" --phase="play" --amplification_factor 20 --vid_dir="babyframes" --is_single_gpu_trained
       python main.py  --checkpoint_path "./model/epoch50.tar" --phase="play" --velocity_mag --amplification_factor 20 --vid_dir="babyframes" --is_single_gpu_trained

**The amplification level can be adjusted by changing the <amplification_factor>.** 

### For the inference with a temporal filter
the amplification is applied by utilizing the temporal filter. This method effectively amplifies small motions of specific frequencies while reducing noise that may arise in the motion magnification results.

1) And then run the temporal filter mode with differenceOfIIR. This code supports three types of <filter_type>, {"differenceOfIIR", "butter", and "fir"}.
      
       python main.py --checkpoint_path "./model/epoch50.tar" --phase="play_temporal" --vid_dir="babyframes" --amplification_factor 20 --fs 30 --freq 0.04 0.4 --filter_type differenceOfIIR --is_single_gpu_trained

**When applying a temporal filter, it is crucial to accurately specify the frame rate <fs> and the frequency band <freq> to ensure optimal performance and effectiveness.** 

**We highly recommend using a temporal filter for real videos, as they are likely to contain the photometric noise.**

### Quantitative evaluation

Many motion magnification methods train their models using the training data proposed by  but the evaluation data for quantitative assessment presented in that paper has not been made publicly available.

As a workaround, you can use the evaluation dataset provided in ["Learning-based Axial Video Motion Magnification, ECCV 2024"](https://axial-momag.github.io/axial-momag/), which is made by strictly following the evaluation dataset generation method proposed in ["Oh, Tae-Hyun, et al., "Learning-based video motion magnification, ECCV 2018"](https://arxiv.org/abs/1804.02684), Numerical results for various motion magnification methods are also available, making it easy to compare your model quantitatively.

To utilize the evaluation dataset, please refer to https://github.com/postech-ami/Axial-mm/tree/main/script.
