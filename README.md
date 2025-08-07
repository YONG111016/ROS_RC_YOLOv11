# ROS2_RC_YOLOv11
1. Jetson 기본 설정
시간 설정: Asia/Seoul 시간대로 변경해서 한국 시간 기준으로 맞춥니다.
시스템 상태 확인:
free -m: RAM과 스왑 메모리 사용량 확인.
df -h: 디스크 사용량 확인 (현재 / 파티션 거의 다 참).
저전력 모드 설정: 배터리 절약을 위해 sudo nvpmodel -m 1 명령 실행.
 2. 쿨링팬 설정
팬 제어 라이브러리 설치:
GitHub에서 jetson-fan-ctl을 클론하고 install.sh로 설치.
소음 조절:
echo 128 > /sys/devices/pwm-fan/target_pwm 로 팬 속도 조절 가능.
 3. 시스템 모니터링 툴 jtop 설치
설치:
sudo apt install python3-pip
sudo -H pip3 install -U jetson-stats
설치 후 reboot하고 jtop 명령어로 실행.
기능: CPU 온도, 사용률, 메모리 등을 실시간으로 확인 가능.
 4. I2C 장치 확인
PCA9685 서보 제어 보드 연결 확인:
i2cdetect -y -r 1 명령어 사용.
주소 60이 보이면 연결 성공.
 5. ROS2 Galactic 설치
설치 스크립트 사용:
git clone -b galactic https://github.com/zeta0707/installROS2.git
./install-ros2.sh 실행 (약 10~15분 소요)
.bashrc 환경 설정 추가:
자주 쓰는 명령어를 alias로 등록 (예: cba, cbp, testpub, rnl 등)
ROS2 환경 초기화: source /opt/ros/galactic/setup.bash 등
 6. ROS2 워크스페이스 초기화
워크스페이스: ~/ros2_ws
명령어:
cca: 이전 빌드 파일 삭제
cba: 전체 패키지 재빌드
 7. ROS2 동작 확인
테스트 명령어:
터미널 1: ros2 run demo_nodes_cpp talker
터미널 2: ros2 run demo_nodes_py listener
→ 메시지 수신되면 ROS2 정상 작동
 8. turtlesim 설치 및 테스트
jetson@nano:~/ros2_ws$ sudo apt update
sudo apt install ros-foxy-turtlesim
jetson@nano:~/ros2_ws$ sudo apt install ros-foxy-turtlesim
제대로 깔렸는지 확인해줍니다.
jetson@nano:~/ros2_ws$ ros2 pkg list | grep turtlesim
/opt/ros/galactic/bin/ros2:6: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import load_entry_point
turtlesim
jetson@nano:~/ros2_ws$ ros2 pkg executables turtlesim
/opt/ros/galactic/bin/ros2:6: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import load_entry_point
turtlesim draw_square
turtlesim mimic
turtlesim turtle_teleop_key
turtlesim turtlesim_node

아래명령을 하면 위의 내용을 알 수 있습니다
# 2. Start turtlesim
# terminal 1 open
  ros2 run turtlesim turtlesim_node
 #  terminal 2 open
  ros2 run turtlesim turtle_teleop_key
 9. 자율주행용 패키지 설치 (monicar3)
설치:

cd ~/ros2_ws/src

git clone https://github.com/zeta0707/monicar3.git

설정:

./carSelect.sh pca9685Steer

./camSelect.sh usbcam

빌드:
cd ~/ros2_ws && cba

monicar_control, monicar_cv, monicar_teleop 패키지가 성공적으로 빌드됨
 10. 기타 패키지 추가 설치
누락된 ROS 패키지 설치:

sudo apt install libgomp1 ros-galactic-joy-* ros-galactic-ackermann-msgs ros-galactic-image-pipeline -y

6장
1. USB 카메라 연결 및 포트 확인
터미널에서 다음 명령으로 연결된 비디오 장치를 확인합니다:

ls /dev/video*

예시 결과:

/dev/video0  /dev/video1
 → 여러 개의 videoX 장치가 보일 수 있으며, 일반적으로 USB 카메라는 /dev/video1일 가능성이 높습니다.


이 장치 중 어떤 것이 실제 USB 카메라인지를 정확히 알아야 합니다.
 2. camera.yaml에서 포트 번호(camport) 설정
카메라 노드가 사용하는 포트는 camera.yaml 파일에서 camport라는 이름으로 지정되어 있습니다.

위치:

~/ros2_ws/src/monicar3/monicar3_cv/param/camera.yaml

예시 설정:
camport: 0

편한 에디터로 수정 가능:

gedit camera.yaml

수정 후 저장하고 빌드해야 적용됩니다:

cd ~/ros2_ws
cbpa monicar3
3. 카메라 노드 실행 및 영상 확인
3.1 Launch 파일로 카메라 노드 실행
카메라 노드 실행:

ros2 launch monicar3_cv usbcam.launch.py


실행하면 다음과 같은 메시지가 출력됩니다:

 csharp
복사편집
[usbcam_pub-1] Camera Port: 0
[usbcam_pub-1] Camera Node created
 → 여기서 Camera Port: 0은 camera.yaml 설정과 일치해야 합니다.


3.2 이미지 보기 (다른 터미널에서 실행)
다음 명령어로 카메라 영상 스트림 확인:

ros2 run image_view image_view --ros-args --remap /image:=/image_raw

실행되면 이미지 창이 뜨고, USB 카메라 영상이 실시간으로 표시됩니다.
4. 카메라 영상 캡처 (추가 방법)
Jetson에서는 GStreamer 도구인 nvgstcapture-1.0를 사용할 수도 있습니다:
nvgstcapture-1.0 --mode=1 --camsrc=0 --cap-dev-node=0

--camsrc=0: USB 카메라 사용

--cap-dev-node=0: /dev/video0을 지정

실행하면 화면에 라이브 영상이 나타나고 Image Captured 메시지와 함께 이미지 캡처 가능
정리
ls /dev/video*로 카메라 장치 번호 확인

camera.yaml의 camport 값 수정 (예: 0 또는 1)

cbpa monicar3로 패키지 재빌드

ros2 launch로 카메라 노드 실행 후, image_view로 영상 확인

추가로 nvgstcapture-1.0을 활용해 영상 캡처 및 테스트도 가능
1. 키보드 Teleoperation 실행 방법
터미널 1에서 다음 명령어를 입력하여 모터 제어용 노드를 실행합니다:

ros2 launch monicar3_control motor.launch.py
 이 명령어는 ROS 2에서 monicar3_control 패키지 안에 있는 motor.launch.py 파일을 통해 차량의 모터 제어 노드를 실행합니다.

터미널 2에서는 키보드 입력을 받아 차량을 조종할 수 있는 Teleop 노드를 실행합니다:

ros2 run monicar3_teleop teleop_keyboard
 이 명령어는 키보드의 방향키나 지정된 키를 사용하여 차량의 움직임을 제어하는 노드를 시작합니다.

2. 명령 실행 중 발생한 경고 메시지
명령어 실행 시 다음과 같은 경고가 발생했습니다:

DeprecationWarning: pkg_resources is deprecated as an API.
 이는 ROS 2 내부에서 사용하는 pkg_resources가 향후 사용 중단(deprecated) 예정이라는 경고로, 기능 동작에는 영향이 없지만, 향후 업데이트 시 주의가 필요합니다.


3. 활성화된 ROS 2 토픽 확인
Teleop 노드 실행 직후 다음과 같은 ROS 2 토픽들이 표시되었습니다:

 bash
/parameter_events
/rosout

/parameter_events: ROS 2 파라미터 변경 사항을 전파하는 토픽입니다.

/rosout: 로그 메시지를 출력하는 시스템 기본 토픽입니다.
핵심 요약
목적
YOLO를 이용해 교통 신호등을 인식하고, 인식 결과에 따라 차량(로봇카)이 출발하거나 멈추는 자동 반응 시스템을 구현.

사용 기술 및 구조
Object Detection:
 → [ultralytics YOLOv8 사용], ROS2용 wrapper 직접 구현.
 → 학습 데이터 관련 내용은 8-1, 8-2장에서 다룸.

패키지 구조:
 yolo_ros 패키지를 monicar3 패키지 내부에 포함.

주요 라이브러리 버전:

torch==1.12.0a0

ultralytics==8.3.98

numpy==1.24.4

기타: torchvision, pandas, ultralytics-thop

동작 흐름
Camera가 이미지 publish

YOLO가 교통 신호등 감지
감지 결과에 따라 차량 제어:

감지되면: Throttle = 0.2, Steering = 0.0 (출발)

감지 안 되면: Throttle = 0.0 (정지)

실행 명령어 예시

# YOLO + 차량 제어 전체 시스템 실행
ros2 launch monicar3_control traffic_all.launch.py

예시 출력 로그 의미
BlobX 0.00: 신호등 위치 정보

is _detected: 신호등이 감지됨

Throttle = 0.2: 차량 전진

Throttle = 0.0: 차량 정지

go / stop: 차량 상태 메시지
