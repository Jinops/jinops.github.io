---
layout: post
title: "FreeCAD Addons 오픈소스 기여"
subtitle: CAD 수업에서 오픈소스 기여라니..
date: 2024-03-07 02:00:00 +0900
background: 
categories: [Opensource]
---

## Link
[https://github.com/dubstar-04/TurningAddon/pull/33](https://github.com/dubstar-04/TurningAddon/pull/33)

## 개요
대단하진 않지만, 작년에 처음으로 오픈소스에 기여했던 경험을 기록한다.

학과 CAD/CAM 수업에서 3D experiences, Solid edge와 같은 상용 SW를 거쳐, 사양 등의 이유로 FreeCAD라는 오픈소스 SW를 사용하게 되었다.

팀프로젝트로 마트 쇼핑카트를 베이스로 한 컨베이어 벨트, 리프트 등이 포함된 카트를 설계하고 CAD로 3D 도면을 만들었다. 프로젝트에는 VR skatch, Generative Design, CAM(one part), 3D Printing 등이 포함되어 있었다.

## CAM
FreeCAD는 CAM 기능을 제공한다. 즉, CNC 머신의 작업경로 설정을 위한 G-code를 만들 수 있다.

교수님께서 우리 팀의 CAM 파트를 바퀴로 선정해주셨다. 참고로 바퀴 모델은 다음과 같다.

![wheel_model](img/posts/2024-03-07-free-cad-contribution/model.png)

바퀴의 둥근 테두리와 움푹 파인 부분은 선반(lathe) 가공이 필요했다. 그러나 FreeCAD의 기본 기능에는 이를 제공하지 않았다.

팀원 모두가 CAD/CAM이 처음이었고, 이 파트는 나의 담당이었다. 가공 방식을 다르게 시도해보는 등 여러 시행 착오 뒤에, 선반 가공을 가능하도록 하는 확장프로그램 [Turning Addon](https://github.com/dubstar-04/TurningAddon)을 찾을 수 있었다.

## 문제 발생

설치를 잘 끝내고 적용 및 실행을 하였으나, G-code가 생성되지 않았다. 여러 번 시도해도 마찬가지였다.

이 시도들에 대한 보고서를 작성을 하고 마무리 하기 전, 디버깅을 해보고자 하였다. 특히 오픈소스 SW의 오픈소스 Addon을 다루다 발생한 오류로, 파고들고자 한다면 소스코드를 모두 볼 수 있는 상황이었다. 또한 아무리 내 역량 밖의 문제라도, 맡을 일을 이렇게 끝내기에는 너무 아쉬웠다. 따라서 끝까지 책임을 다 하고자 하였다.
 

## 해결 과정

### 기본 정의
우선 use case를 다음과 같이 순서대로 정의하고, 문제를 야기하는 잠재적인 상황으로 생각했다.

  1. Addon을 추가
  2. Job 생성
  3. Addon에서 추가된 작업(Turn Face) 설정
  4. 작업 실행 - 실제 에러 발생 시점
  5. Path 및 Face 생성 완료
  6. G Code 조회

또한 (1) 해당 Addon 내부의 로직 결함, (2) 프로그램과 Addon 사이 데이터를 주고 받는 것에서 발생하는 결함, 두 경우를 중심으로 접근했다.



위 기준을 바탕으로, 발생한 에러를 확인하였다. 함수에 전달된 `parentJob`이라는 argument가 정의되지 않은 것이었다. 만약 필자 본인이 작성한 코드였다면, 당연히 이를 상단에 정의하면 끝났겠지만, 해당 코드에서 `parentJob`이 어느 쓰임이며, 어떻게 정의되는지 알 수 없었기에 확실한 근거가 필요했다.

이를 파라미터로 받는 함수는 `Create`였는데, 위 4-5번에서 결과를 만드는 상황에 호출되는 함수임을 짐작할 수는 있었으나, Addon의 이 함수가 FreeCAD의 로직 중 어디에서 어떻게 호출되는지 찾을 수 없어  `parentJob`의 출처를 찾는데 어려움이 있었다.

### 시야 확장

`parentJob`이 Addon 내부 로직이 아닌 프로그램으로부터 전달받는 값이라는건 금방 알 수 있었다. 이를 통해 FreeCAD 원본 코드에서, 선반이 아닌 밀링(Milling), 드릴링(Drilling) 등 다른 작업을 실행하는 함수를 찾았다.

해당 작업들의 코드 구조는 Addon의 작업들 코드와 매우 유사했다. 즉, `Create`와 같은 함수가 동일하게 존재했다. 해당 함수를 호출하는 상위 코드는 당장 알 수 없으나 기존 기능들에는 `parentJob`을 인자로 전달하고 있다는 것을 확인, `parentJob`이 `Create` 호출 시 항상 인자로 넘겨준다는 것을 확인할 수 있었다.

에러 발생에 대한 이전 상황을 추적하려는 방식에서, 단순한 코드 대조를 통해 훨씬 쉽게 에러를 수정할 수 있었다. 물론 상위 코드 추적은 이에 대한 확신을 얻기 위한 과정으로 큰 도움이 되었다.

![path](img/posts/2024-03-07-free-cad-contribution/path.png)
![gcode](img/posts/2024-03-07-free-cad-contribution/gcode.png)
(생성된 path와 G Code)

## PR

위에서 발견 및 수정한 내용을 바탕으로, 원본 레포지토리에 PR을 올려 Merge되었다. ([https://github.com/dubstar-04/TurningAddon/pull/33](https://github.com/dubstar-04/TurningAddon/pull/33))

![PR](img/posts/2024-03-07-free-cad-contribution/PR.png)


## 여담

사실 수정한 코드는 매우 단순하고, 디버깅의 난이도도 높지는 않다. 그러나 내 코드가 아닌 다른 사람의 코드를 수정하기 위해 더 꼼꼼히 살펴봐야 했으며, 현재 상황이 원본 코드 작성자가 의도하지 않은 버그임을 확신하고 수정하기 위해 더 신중해야 했다.

이렇게 첫 기여를 하고 난 뒤, 오픈소스 사용 및 수정, 기여에 대한 부담이 훨씬 덜어졌다. 이 경험을 통해 얻은 자신감이 최근 프로젝트에도 이어졌는데, 관련 내용은 [오픈소스 모델과 로봇 개발 프로젝트](https://jinops.github.io/%ED%9A%8C%EA%B3%A0/%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/2024/01/16/conveyor-alignment-robot.html) 게시글에 짧게 기록하였다.
