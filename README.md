# OCR
'OCR' is an abbreviation for 'Optimal Character Recognition'. Our project is to apply OCR to 'traditional Hangul'.
<br><br>

## Quick Guide
<b>환경(Environment)</b>
 ```
 Ubuntu 18.04
 cuda 10.1
 cudnn 7.6.5
 opencv
 ```
1. Git clone darknet(AlexeyAB)
 ```
 git clone https://github.com/AlexeyAB/darknet.git
 ```

2. darknet 실행
```
cd darknet
```
Makefile 수정: GPU = 1, CUDNN = 1, OPENCV=1
```
make
```

3. custom 폴더 생성
파일들을 다운로드 받고 custom 폴더를 생성  
custom 폴더에 'custom.data', 'custom.names', 'custom_yolov3.cfg', 'custom_yolov3_test.cfg'파일 넣기
<br>

4. weights
backup 폴더에 custom_yolov3_best.weights 넣기
<br>

5. 실행
```
./darknet detector test custom/custom.data custom/custom_yolov3_test.cfg backup/custom_yolov3_best.weights
Enter Image: 
```


## 1. Introduction
‘OCR’ 이란 ‘Optimal Character Recognition’ 의 줄임말로, 광학 문자 인식 기술이라고도 불린다.<br> 
이는 사람이 직접 쓰거나 이미지 속에 있는 문자를 컴퓨터가 인식할 수 있도록 디지털화하는 기술이다. OCR의 기본 구조는 이미지에서 문자가 있는 부분을 찾아내는 detection과 찾아낸 문자가 무엇인지 판단하는 classification으로 구성된다. <br>
해당 과제는 ‘옛한글’ 이미지를 입력했을 때 이미지에서 문자를 인식하여 분류하는 것을 목표로 한다. 
(이 repo에서는 detection에 관한 부분만 다룬다.)  

<br>

## 2. Process
옛한글의 경우 현재 가지고 있는 데이터 양이 적고 annotation이 되어있지 않아 옛한글과 비슷한 고서 한자 데이터(AIHub)를 이용하여 우선적으로 실험을 진행했다.<br> 
실험과 옛한글 labeling을 병렬적으로 진행하며 고서 한자 데이터에 대한 성능을 확인하고 이를 옛한글에 적용하는 방법으로 모델링 하였다.<br>
- 고서 한자 데이터: 해서체 이미지 1,456장, 행서체 49,918장 <br>
- 옛한글 데이터: 판본 이미지 10장, 필사본 이미지 47장<br>

고서 한자 데이터             |  옛한글 데이터
:-------------------------:|:-------------------------:
<img src="https://user-images.githubusercontent.com/49026215/125572438-7e3b2df2-e3ac-434e-bc90-915ee2406f4e.jpg"  width="300" height="400"> | <img src="https://user-images.githubusercontent.com/49026215/125573883-b94031ec-0e7a-4a07-87f5-23dac7443cb0.jpg"  width="300" height="400">

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;< Table1. 고서 한자 데이터와 옛한글 데이터 비교> <br>

### 2-1. Kaggle Kuzushiji Recognition
‘Kaggle kuzushiji recognition’은 일본 고대 문자인 kuzushiji가 표기된 이미지에서 문자를 찾아 분류하는 모델을 구축하는 대회로 해당 과제와 유사하다고 생각되어 대회에서 우수한 성능을 낸 코드를 위주로 살펴보았다. </n>
그 중 대회에서 ‘score 0.934’를 기록한 K_mat의 base code 흐름을 참고하였으며, base code는 다음과 같이 두 단계로 구성되었다.</n>

 1) Detection Model  
 모든 문자의 class를 0으로 라벨링하여 글자가 있는 부분을 detection하도록 학습시켰으며 network는 CenterNet을 사용하였다.  
 2) Classification Model  
 input image annotation의 좌표 값을 이용하여 문자별로 crop image를 저장하고 crop된 이미지를 활용하여 CNN으로 학습시켰다.  
<br>
우리는 기존과 같이 모델을 detection과 classification으로 나눠 두 가지로 구축하는 방법과 detection과 classification을 한 번에 할 수 있는 Large class detection에 대한 방법도 추가로 고민해봤다.   
<br>

### 2-2. 고서 한자 데이터에 base model 적용
고서 한자 데이터에 대한 base model의 성능을 알아보기 위한 실험을 진행했다.  

구분            |  Network   |   결과   |    결과
:-------------------------:|:-------------------------:|:-------------------------:|:-------------------------:
Detection | CenterNet | <img src="https://user-images.githubusercontent.com/49026215/125577334-614270ad-04b9-4c0f-abbe-8c33c0bf6743.png"  width="200" height="300"> | <img src="https://user-images.githubusercontent.com/49026215/125577342-f731ca5f-6660-4077-a167-33f47ceeea1c.png"  width="200" height="300"> 
Classification | CNN | <img src="https://user-images.githubusercontent.com/49026215/125577345-85b6b93c-2e4b-48c9-9267-b8b5019e6a37.png"  width="200" height="300"> | <img src="https://user-images.githubusercontent.com/49026215/125577346-4ff87df1-5ff4-4006-90a1-6d80cf588ac8.png"  width="200" height="300">

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;< Table2. 고서 한자 데이터에 대한 base model 성능 > <br><br>

CenterNet을 이용한 detection의 경우 mAP가 0.736으로 나쁘지 않은 성능을 보여줬다.   
classification은 비슷한 문자가 별로 없거나 단순한 획으로 구성된 경우에는 잘 구별하였지만 *< Table2 >* 의 왼쪽 아래 그림과 같이 뒷면에 비치는 글자까지 포함하여 인식하는 경우가 발생했다.   
즉, 우리는 다음과 같은 3가지 부분에서 개선점을 발견하였다.  
1. detection model의 성능 향상을 위해 CenterNet대신 YOLO를 사용   
2. classification model의 성능 향상을 위해 CNN대신 ResNet 등 일반적으로 더 높은 성능을 보이는 network를 사용  
3. 이미지 전면의 글자를 강조하는 image Preprocessing 및 augmentation 진행  
 
실험을 통해 찾은 개선 사항을 반영하여 우선적으로 라벨링이 완료된 판본과 필사본의 일부를 이용하여 모델을 구현하였다.  
<br><br>
## 3. 옛한글 데이터에 대한 Detection 구현
판본 이미지 10장과 필사본 이미지 22장에 대한 라벨링이 완료되어 해당 데이터들을 YOLOv3로 학습시켜 성능을 살펴보았다.  
또한 판본과 필사본의 글씨체를 구분하여 detecting 할 수 있는지 알아보기 위해 판본과 필사본의 class를 구별하여 labeling을 진행했다.    
### 3-1. Labeling
판본의 글자를 word, 필사본의 글자를 word_hw(handwriting)로 구분하여 라벨링을 진행하였다.  
<br>

### 3-2. Data Set  
- 판본: 이미지 10장, 총 2836 자  
- 필사본: 이미지 22장, 총 4635자   
Train과 Test를 7: 3으로 나눠 train으로 22장(판본 7장, 필사본 15장)의 이미지를, test로 10장(판본 3장, 필사본 7장)의 이미지를 사용했다.  
<br>

### 3-3. Training
초기 가중치는 darknet53.conv.74를 사용했으며 cfg파일은 yolov3.cfg파일을 기반으로 하였다.   
성능 향상을 위해 width와 height를 608로 증가시켰고, class가 한 개이므로 max_batches를 기존 500200에서 2200으로 낮춰주었다. 이에 따라 steps 역시 1600, 1800으로 수정해주었다.   
[yolo] 부분의 classes를 1, filters를 18로 수정해서 학습을 진행했다.   
<br>
<b>[ 정리 ]</b>  
초기 가중치: darknet53.conv.74    
cfg: yolov3.cfg   
구분   |  변경 전  |  변경 후
:-------------------------:|:-------------------------:|:-------------------------:
width, height | 416 | 608  
max_batches | 500200 | 4200
steps | 400000, 450000 | 3200, 2600  

[yolo]의 classes = 80 → 2, [yolo] 바로 위의 [convolutional]의 filter=255 → 21  
*ctrl+f로 [yolo]를 검색하면 세 개가 검색되는데 세 부분 모두 변경해줘야함.*  


max_batches = 2200             |  max_batches = 4200
:-------------------------:|:-------------------------:
<img src="https://user-images.githubusercontent.com/49026215/125578608-d8a673c5-caeb-4139-abf6-27e7f55f0170.png"  width="300" height="300"> | <img src="https://user-images.githubusercontent.com/49026215/125578610-7f8ca87b-54a0-47ed-9a7b-bc50cbce493f.png"  width="300" height="300">

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;< Figure1. 옛한글 학습 chart (max_batches = 좌: 2200, 우: 4200)  >

<br>

### 3-4. Test 결과
class_id = 0 (word, 판본)의 ap = 29.23%  
class_id = 1 (word_hw, 필사본)의 ap = 92.83%  
precision |  recall  |  F1-score | Average IoU | mAP@0.50 | Total Detection Time
:------------:|:------------:|:------------:|:------------:|:------------: |:------------:
0.94 | 0.41 | 0.57 | 77.14% | 61.03% | 1s

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;< Table3. 판본과 필사본 일부 데이터 학습 성능(YOLOv3)  >

<br>

판본            |  필사본   |   필사본   
:-------------------------:|:-------------------------:|:-------------------------:|
<img src="https://user-images.githubusercontent.com/49026215/125579173-85219fbd-07d8-4d24-8dbd-6cec85abc180.png"  width="300" height="300"> | <img src="https://user-images.githubusercontent.com/49026215/125578734-f1de0996-d7f5-4bf0-828e-c4fdb537651d.jpg"  width="300" height="300"> | <img src="https://user-images.githubusercontent.com/49026215/125578746-7229de6e-fbae-4e24-bcc3-066d01a549dc.jpg"  width="300" height="300">
8오륜행실도 | 26새 신부가 시아버지에게 보낸 안부 편지  | 44어떤 여인이 숙주라는 여인에게 보낸 편지


필사본은 Train과 Test 데이터의 글자 크기가 거의 일정하였고 그 결과 ap가 92.83%로 아주 좋은 성능을 보였다.  
<br>
YOLOv3를 사용하여 옛한글 데이터에 대한 학습을 진행한 결과 데이터 양이 적었음에도 불구하고 우수한 성능을 보였다.  
YOLO가 detection에서 우수한 성능을 보인 만큼 classification모델을 따로 만들지 않고 YOLO 자체로 Large class detection을 할 수 있는 방법에 대해 알아보았다.   

<br>

## 4. Detection model을 사용하여 Large Class Detection (YOLO9000)  
고서 한자 데이터를 이용해 YOLOv3 자체를 그대로 사용하여 detection과 classification을 동시에 시도할 경우 out of memory 문제가 발생하였다. (행서 데이터의 class 갯수: 5911개)   
옛한글은 지금의 한글보다 자모음의 조합이 훨씬 많기 때문에 한자 데이터와 비슷한 수준정도로 많은 class가 생성될 것이고 이를 현재 YOLOv3로 해결하기에는 무리가 있다고 판단하였다.  
이에 따라 large class detection에 대한 연구 사례 research를 진행한 결과 YOLO9000 모델과 이 모델을 현재 우리 데이터에 접목시킬 수 있는 가능성에 대해 살펴보았다.   
YOLO9000은 large class에 대한 Hierarchincal tree인 WordTree를 구성하여 detection과 classification을 joint training하는 방법을 제안한다.   
논문에서는 COCO dataset과 ImageNet을 결합하여 9418개의 클래스를 지닌 WordTree를 생성하였다.    
YOLO9000의 성능은 19.7mAP로 다소 떨어지는 것을 확인할 수 있다.    
WordTree를 구성하여 detection과 classification의 loss를 따로 계산하는 등 다양한 시도를 하였지만 역시나 클래스가 많아짐에 따라 성능이 낮아진 것으로 보인다.   

Figure2            |  Figure3  
:-------------------------:|:-------------------------:
<img src="https://user-images.githubusercontent.com/49026215/125578753-03f7bf2b-01a9-4e85-a954-4133defe9e35.png"  width="300" height="300"> | <img src="https://user-images.githubusercontent.com/49026215/125578759-98340e9b-b25a-4168-8363-134a15f2dd49.png"  width="300" height="300"> 

우리가 YOLO9000을 벤치마킹할 경우 학습에 사용되는 문자에 대해 WordTree를 구성하는 작업이 필요한데 이때 무엇을 기준으로 상위와 하위 범주를 구분할 것인지에 대한 고민이 필요하다. 또한 논문에 나온 YOLO9000의 성능을 보았을 때 class가 많은 옛한글 데이터에 대한 classification 성능 역시 좋지 못할 것으로 예상된다.

<br>


