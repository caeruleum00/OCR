# OCR
'OCR' is an abbreviation for 'Optimal Character Recognition'. Our project is to apply OCR to 'traditional Hangul'.
<br><br>

## 1. Introduction
‘OCR’ 이란 ‘Optimal Character Recognition’ 의 줄임말로, 광학 문자 인식 기술이라고도 불린다.<br> 
이는 사람이 직접 쓰거나 이미지 속에 있는 문자를 컴퓨터가 인식할 수 있도록 디지털화하는 기술이다. OCR의 기본 구조는 이미지에서 문자가 있는 부분을 찾아내는 detection과 찾아낸 문자가 무엇인지 판단하는 classification으로 구성된다. <br>
해당 과제는 ‘옛한글’ 이미지를 입력했을 때 이미지에서 문자를 인식하여 분류하는 것을 목표로 한다. 

<br>
## 2. Process
옛한글의 경우 현재 가지고 있는 데이터 양이 적고 annotation이 되어있지 않아 옛한글과 비슷한 고서 한자 데이터(AIHub)를 이용하여 우선적으로 실험을 진행했다.<br> 
실험과 옛한글 labeling을 병렬적으로 진행하며 고서 한자 데이터에 대한 성능을 확인하고 이를 옛한글에 적용하는 방법으로 모델링 하였다.<br>
 고서 한자 데이터: 해서체 이미지 1,456장, 행서체 49,918장 <br>
 옛한글 데이터: 판본 이미지 10장, 필사본 이미지 47장<br>
<br>

![1](https://user-images.githubusercontent.com/49026215/125572438-7e3b2df2-e3ac-434e-bc90-915ee2406f4e.jpg){: width="100" height="100"}
