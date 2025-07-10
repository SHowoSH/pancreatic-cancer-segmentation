# pancreatic-cancer-segmentation
3D Pancreatic Cancer Segmentation from CT scans using PyTorch &amp; MONAI.
CT 기반 3D 췌장암 분할 (Pancreatic Cancer Segmentation)
Medical Segmentation Decathlon (MSD) 데이터를 활용하여, 췌장(Pancreas)과 종양(Tumor) 영역을 분할하는 3D U-Net 모델을 개발하고 3D 시각화로 결과를 확인하는 프로젝트입니다.

1. 프로젝트 개요
목표: CT 영상에서 췌장과 종양 영역을 정밀하게 분할하는 3D Semantic Segmentation 모델 구축

데이터:

췌장암 (Cancer): Medical Segmentation Decathlon (MSD) - Task07_Pancreas (281건)

정상 (Normal): 자체 확보 데이터 (t1.zip, 385건)

핵심 성과:

최종 검증 데이터셋에서 평균 Dice Score 0.7062 달성 (at 96 epoch)

Plotly를 이용해 분할된 췌장과 종양 영역의 인터랙티브 3D Mesh 시각화 구현 및 HTML 저장 기능 개발

2. 모델 아키텍처 및 학습 전략
2.1. Custom 3D U-Net
본 프로젝트에서는 의료 영상 분할에 특화된 3D U-Net 아키텍처를 PyTorch와 MONAI를 기반으로 직접 구현했습니다.

구조: 3D Convolution 기반의 Encoder-Decoder 구조와 Skip-connection을 통해 다중 스케일의 공간적 특징을 효과적으로 학습합니다.

활성화 함수: LeakyReLU를 사용하여 Dying ReLU 문제를 방지했습니다.

손실 함수: DiceCELoss를 사용하여, Dice Loss와 Cross-Entropy Loss의 장점을 결합해 영역 분할의 정확도와 안정성을 동시에 확보했습니다.

2.2. 데이터 전처리 및 증강
MONAI의 강력한 Transform 파이프라인을 활용하여 데이터 처리의 효율성과 모델의 강건성을 높였습니다.

주요 전처리:

LoadImaged, EnsureChannelFirstd: NIfTI 데이터 로드 및 채널 차원 정리

Orientationd: 모든 영상의 방향을 "RAS"로 통일

ScaleIntensityRanged: CT 영상의 HU 값을 (-100, 240) 범위로 윈도잉 후 정규화

CropForegroundd: 관심 영역(Foreground) 중심으로 불필요한 배경 제거

Resized: 모든 영상의 해상도를 (64, 96, 96)으로 통일

데이터 증강 (Augmentation):

학습 데이터에 RandFlipd, RandRotate90d, RandShiftIntensityd 등을 적용하여 모델이 다양한 영상 패턴에 일반화될 수 있도록 학습했습니다.

클래스 불균형 해소:

암 데이터(281건)가 정상 데이터(385건)보다 적은 문제를 해결하기 위해, 암 데이터를 오버샘플링(Oversampling) 하여 클래스 간의 데이터 수를 동일하게 맞췄습니다.

3. 결과 및 시각화
3.1. 학습 결과
총 100 Epoch 학습 과정에서 96번째 Epoch에서 최고 평균 Dice Score 0.7062를 기록하며, Early Stopping (Patience=15) 및 ReduceLROnPlateau 스케줄러를 통해 최적의 모델을 찾았습니다.

3.2. 3D 분할 결과 시각화
추론된 마스크 결과를 skimage의 marching_cubes 알고리즘으로 3D 메쉬(Mesh)로 변환하고, Plotly를 이용해 인터랙티브 3D 시각화를 구현했습니다.

사용자는 마우스로 췌장(파란색)과 종양(빨간색)의 3D 모델을 회전하고 확대하며 직관적으로 분할 결과를 확인할 수 있으며, 이 결과는 독립적인 HTML 파일로 저장하여 공유할 수 있습니다.

4. 사용 방법
Repository 클론

Bash

git clone https://github.com/SHowoSH/pancreatic-cancer-segmentation.git
cd pancreatic-cancer-segmentation
필요 라이브러리 설치

Bash

pip install -r requirements.txt
학습 실행

췌장암_로컬 (1).ipynb 노트북 파일의 DRIVE_BASE_PATH를 실제 데이터가 저장된 경로로 수정한 후 실행합니다.

추론 및 3D 시각화

new_모델_시각화.ipynb 노트북 파일의 MODEL_PATH와 INPUT_CT_PATH를 학습된 모델과 분석할 CT 영상 경로로 수정한 후 실행하면, pancreas_3d_visualization.html 파일이 생성됩니다.