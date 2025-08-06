# ROS2_RC_YOLOv11
'ifconfig' 네트워크 인터페이스를 설정하거나 정보를 확인할 때 사용하는 명령어로 주로 리눅스나 유닉스 계열 시스템에서 사용한다.
aarch64는 ARM 아키텍처에서 **64비트 명령어 세트(Instruction Set Architecture, ISA)**를 의미합니다.
ROS 2는 로봇의 센서, 모터, 카메라 등 여러 부품이 서로 통신하며 동작할 수 있도록 돕는 로봇 운영체제입니다.
pca9685는 I²C(아이투씨) 통신을 사용하는
16채널, 12비트 PWM 신호 생성기입니다.

pca9685+MotorHat

Steering에 Servo motor를 사용
Throttle에 MotorHat B channel 사용
Throttle은 one motor → Gear → two wheel 구동
This car(pca9685+MotorHat) Vs Road balance RCCar
Steering(조종종)
I2C → pca9685 0x60→Servo Motor
Throttle(엔진출력조절장치)
I2C→(pca9685 0x40 → TB6612FNG)(=MotorHat B) → DC motor
Note
Steer Center값을 찾아야합니다.

./jetRccarParam.sh pca9685Steer
-I2C-
0x60이 PCA9685 주소입니다. 0x40은 Motorhat의 주소입니다.



jetson@jp4612GCv346Py37:~$ sudo i2cdetect -r -y 1
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- -- 
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
40: 40 -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
60: 60 -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
70: 70 -- -- -- -- -- -- --​
PCA9685 주소를 A5 short시켜서 0x60으로 만들어주세요

Circuit Diagram
steering motor를 pwm으로 servo motor값을 조정합니다. Throttle motor는 MotorHat B channel에 연결합니다. Throttole motor하나가 기어에 연결된 두 바퀴를 동작시킵니다.
전체 블록도는 아래와 같습니다.

