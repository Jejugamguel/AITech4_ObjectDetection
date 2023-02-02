![header](https://capsule-render.vercel.app/api?type=rect&&color=0:00FF7F,100:87CEEB&text=재활용%20품목%20분류를%20위한%20Object%20Detection&fontSize=32)
<div align="left">
	<img src="https://img.shields.io/badge/Python-3776AB?style=flat&logo=Python&logoColor=white" />
	<img src="https://img.shields.io/badge/Pytorch-EE4C2C?style=flat&logo=Pytorch&logoColor=white" />
	<img src="https://img.shields.io/badge/OpenMMLab-181717?style=flat&logo=Github&logoColor=white" />
</div>

&nbsp;

# Members
- **김도윤**  : 2stage model 실험(faster, cascade, htc 등), Hyperparameter Tuning, Weighted Boxes Fusion 실험
- **김윤호**  : Data Augmentation 실험(Auto-aug, Mosaic, Multi-Scale), 2stage model 실험(ATSS-Dyhead, cascade-rcnn), Stratified Group Kfold 구현
- **김종해**  : 1stage model 실험(RetinaNet, Yolo v7), Stratified Group Kfold 구현, Weighted Boxes Fusion 실험
- **조재효**  : Data Augmentation 실험(TTA, Albumentation), 1stage model 실험(Yolo v7), 2stage model 실험 (cascade-SwinB), Hyperparameter Tuning
- **허진녕**  : EDA, 1stage model 실험(Yolo v3, Yolof, Yolox), Hyperparameter Tuning

&nbsp;

# 프로젝트 개요
> 대량 생산, 대량 소비의 시대에 접어들면서 '쓰레기 대란' 문제가 함께 수면 위로 떠올랐습니다. 많은 쓰레기가 배출되면서 환경 오염 문제가 대두되었고, 이를 해결하기 위해 올바른 분리수거 습관을 함양해야 한다는 목소리가 강해졌습니다. 잘 분리된 쓰레기는 다시 자원으로서의 가치를 인정받기에, 재활용 품목을 분류하는 Object Detection 모델을 설계하여 환경 부담을 줄이는 과제에 앞장설 것입니다.

&nbsp;

# Repository 구조
```
├─ data
│  ├─ train
│  │  └─ 0000.jpg
│  ├─ test
│  │  └─ 0000.jpg
│  ├─ train.json
│  └─ test.json
│
└─ Repo
   └─ mmdetection
      ...
      └─ configs
         ...
         └─ cv03
            ├─ model1
            ├─ model2
            └─ utils
```	


&nbsp;

# 프로젝트 수행 절차
<h3> 1. EDA  </h3>
<h3> 2. 모델 및 Augmentation 기법 탐색  </h3>
<h3> 3. 모델 선정 및 최적 Augmentation 기법 선정  </h3>
<h3> 4. Hyperparameter Tuning  </h3>
<h3> 5. K-fold Cross Validation </h3>
<h3> 6. Model Ensemble  </h3>

&nbsp;

# 문제정의
<h3> 1. mAP에 대한 의구심   </h3>  

- Test set에 대하여 Inference를 거친 결과, Bounding Box가 과도하게 많이 그려진다는 특징을 발견했다.
- 그럼에도 Test set에 대한 mAP50은 0.6 근처로 낮지 않게 나왔는데, 이로부터 mAP50이 모델 성능을 대변하는 지표로 적합한가에 대한 의구심이 생겼다.
- 결론적으로 성능의 '경향성'을 대변할 수 있다고 결론을 내었고, 과도하게 많은 Bounding Box는 서로 인근의 것과 합쳐지거나 threshold를 높임으로써 문제상황을 완화할 수 있다고 판단하였다.

&nbsp;

# 모델 및 Data Augmentation
- Cascade SwinT
    - Resize
    - Albu
        - HorizontalFlip
        - VerticalFlip
        - OneOf (ShiftScaleRotate x3)
        - OneOf (Blur, MedianBlur)
        - OneOf (RGBShift, HueSaturationValue,
                 RandomBrightnessContrast)
        - ChannelShuffle
    - Normalize
    - Pad
   
- Yolo v7

- ATSS SwinT Dyhead
    - Resize
    - Albu
        - HorizontalFlip
        - OneOf (Blur, MedianBlur)
        - OneOf (RGBShift, HueSaturationValue,
                 RandomBrightnessContrast)
        - ChannelShuffle
    - Normalize
    - Pad

&nbsp;

# Advanced Techniques
<h3> 1. Stratified Group K-fold   </h3>  

- 교차 검증을 위해 데이터를 분할하기로 결정하였다.
- 한 이미지 내에 다수의 카테고리 품목이 포함되고, 해당 Annotation은 서로 분리되지 않아야 하므로, 지정된 Group을 유지시켜주는 Stratified Group K-fold를 적용하였다.

<h3> 2. Weighted Boxes Fusion   </h3>  

- 예측한 Bounding Box를 적절히 합쳐 새로운 Bounding Box를 생성하는 Weighted Boxes Fusion 기법을 활용하여 Model Ensemble을 진행하였다.
- K-fold Cross Validation으로 도출된 5개의 Inference 결과에 적용하여 더 높은 mAP50을 기록할 수 있었다.
- 서로 다른 모델의 Inference 결과에 역시 적용하여, 더 높은 mAP50을 기록할 수 있었다.