# ROS2_RC_YOLOv11
'ifconfig' 네트워크 인터페이스를 설정하거나 정보를 확인할 때 사용하는 명령어로 주로 리눅스나 유닉스 계열 시스템에서 사용한다.
pca9685+MotorHat 

1. Introduction
Steering에 Servo motor를 사용
Throttle에 MotorHat B channel 사용
Throttle은 one motor → Gear → two wheel 구동
2. This car Vs Road balance RCCar
Steering
I2C → pca9685 0x60→Servo Motor
Throttle
I2C→(pca9685 0x40 → TB6612FNG)(=MotorHat B) → DC motor
Note
Steer Center값을 찾아야합니다.
3장에서 RCCar parameter를 아래와 같이 pca9685Steer로 선택해주세요.
./jetRccarParam.sh pca9685Steer
​
3.  I2C
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
70: 70 -- -- -- -- -- -- --
​
PCA9685 주소를 A5 short시켜서 0x60으로 만들어주세요

4. Circuit Diagram
steering motor를 pwm으로 servo motor값을 조정합니다. Throttle motor는 MotorHat B channel에 연결합니다. Throttole motor하나가 기어에 연결된 두 바퀴를 동작시킵니다.
전체 블록도는 아래와 같습니다.

아래 위치에 이 프로젝트에 사용한 MotorHat에 관한 회로도가있습니다. 이걸 참고하여 코드를 아래 trottle 코드를 작성하였습니다.
https://www.waveshare.com/w/upload/a/ab/Motor_Driver_HAT_Schematic.pdf

class PWMThrottleHat:
:
if throttle > 0:   
    # MotorHat B
    self.controller.pwm.set_pwm(self.controller.channel+ 5,0,pulse)
    self.controller.pwm.set_pwm(self.controller.channel+ 4,0,0)
    self.controller.pwm.set_pwm(self.controller.channel+ 3,0,4095)

else:
	  # MotorHat B
	  self.controller.pwm.set_pwm(self.controller.channel+ 5,0,-pulse)
	  self.controller.pwm.set_pwm(self.controller.channel+ 4,0,4095)
	  self.controller.pwm.set_pwm(self.controller.channel+ 3,0,0)
