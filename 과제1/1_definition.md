## 문제 정의
김대리는 부품과 제품을 공정 간에 운송하는 과정을 자동화하는 프로토타입 운반 로봇을 개발하고자 하며, 이 로봇은 바닥에 그려진 검은색 선(경로)을 인식하여 자율적으로 이동하고, 이동 기록 데이터를 수집하는 것이 목표로 한다

## 요구 사항 정리:
1. 바닥의 검은색 선(경로)을 인식할 수 있어야 한다.
2. 인식된 경로를 따라 자율적으로 이동할 수 있어야 한다.
3. 이동 경로와 시간, 위치 데이터를 기록하고 저장할 수 있어야 한다.
4. 실시간으로 데이터를 수집하고 저장하는 기능이 필요하다.
5. 프로토타입 하드웨어와 소프트웨어를 제작할 수 있어야 한다.

##1.목표정리{
    김대리가 해결해야 할 문제는 공정 간 부품과 준비 제품을 운송하는 작업을 자동화하기 위해, 실제 로봇 운반차 개발 전에 핵심 기능을 검증할 수 있는 프로토타입 로봇을 기획하는 것이다.
}

##2.What is problem?{
    부품&제품 들간의 공정간 이동이 필요, 사람이 하는건 비효율적(가장효율적인 이동데이터 절대적인 관점& 객관적 관점에서 수집 불가, 항상 같은 경로로 다니는 것 등등) 따라서 실제 운반 로봇을 도입하기 전에, 정해진 경로를 따라 이동하고 주행 데이터를 수집할 수 있는 프로토타입 로봇을 제작하여 가능성을 검토
}

##3.What is Solution?
    
    요구사항 (void 요구사항){
        case 1:
            바닥의 검은선을 인식해야 한다.
        case 2:
            인식된 경로를 따라 자율적으로 이동할수 있어야 한다.
        case 3:
            이동경로&시간&위치데이터 기록 및 저장
        case 4:
            실시간 데이터 수집 및 저장 기능
        case 5:
            Prototype 하드웨어 및 S/W 를 제작

    }

    해결방법 (void 해결){
        case 1:
            case1번의 요구사항의 문제 같은 경우 바닥의 검은색 선을 따라 인식해야하는 부분은
            검은색과 대비되는 하얀색 바닥을 통해서 검은색이 좀더 탐지되게 쉽게 만들어 오류를
            적게 만드는걸 기준으로 구성한다.

        case 2:
            case2번의 요구사항의 문제 같은 경우 색깔로 길을 표시하는데 검은색이면 이동할수 있는 경로로 만들어주고 나머지 흰색이면 그 경로로 돌아다닐수 없게 제어하면 가능하다.

        case 3: //기록대상
            if(내가 생각한것 ){
                case3번의 요구사항의 문제 같은경우 이동경로 및 시간 위치데이터 기록은
                이동경로는 Slam 을 통해서 실질적으로 주변환경 탐색 및 내위치 추정을 동시에 하면서 알 수 있는 문제이고 시간은 파이썬 관련 스크립트로 사용하면된다.
                위치 정보같은 경우에는 실질적으로 바닥에 흰색과 검은색 선을 통해서 진행하기 떄문에 각각의 진행되는 분기점(시퀀스)들을 통해서 현재 위치가 어디인지 파악할 수 있게 만든다.
                }
            
            elif(gpt가 정리해준것){
                이동 기록 데이터는 로봇이 실제로 어떤 경로를 따라 이동했는지, 어느 시점에 특정 위치를 통과했는지 확인하기 위해 필요하다. 프로토타입 단계에서는 로봇의 이동 시간, 경로 통과 상태, 현재 위치 상태를 기록할 수 있어야 한다.

                이동 경로는 기본적으로 하얀색 바닥 위의 검은색 운송 경로를 따라 주행한 결과를 기준으로 판단한다. 로봇이 경로를 따라 이동하는 동안 각 구간 또는 분기점을 통과할 때마다 해당 지점을 하나의 시퀀스로 기록하여 현재 로봇이 어느 위치에 있는지 파악할 수 있도록 한다.

                시간 데이터는 주행 제어 프로그램과 별도의 데이터 기록 스크립트를 함께 실행하여 기록할 수 있다고 가정한다. 기록 항목에는 주행 시작 시간, 각 구간 통과 시간, 정지 시간, 주행 종료 시간이 포함된다.

                또한 추후 확장 기능으로 LiDAR 기반 SLAM 기술을 적용할 경우, 주변 환경 지도를 생성하면서 로봇의 현재 위치를 추정할 수 있다. 이를 통해 로봇이 실제로 이동한 경로를 지도 위에 시각적으로 표현하고, 이동 경로 데이터를 더욱 정확하게 분석할 수 있다
            }

        case 4: //기록방식
            if(내가 생각해서 gpt에게 물어본것){
                로봇은 주행 중 발생하는 데이터를 실시간으로 수집하고 저장할 수 있어야 한다. 수집 대상 데이터에는 주행 시간, 현재 주행 상태, 경로 인식 여부, 분기점 또는 구간 통과 여부, 정지 여부, 오류 발생 여부가 포함된다.

                실시간 데이터 수집은 주행 제어 프로그램과 별도의 파이썬 기록 스크립트를 동시에 실행하여 수행할 수 있다고 가정한다. 파이썬 스크립트는 일정 시간 간격으로 로봇의 상태 데이터를 읽고, 이를 CSV 또는 로그 파일 형태로 저장한다.

                위치 데이터는 프로토타입 단계에서는 검은색 운송 경로의 분기점이나 구간 정보를 기준으로 기록한다. 예를 들어 로봇이 출발 지점, 직선 구간, 곡선 구간, 분기점, 도착 지점을 통과할 때마다 해당 위치 상태와 시간을 함께 저장한다.

                추후 확장 기능으로 LiDAR 기반 SLAM을 적용할 경우, 로봇이 주변 환경 지도를 생성하면서 현재 위치를 추정할 수 있다. 이 경우 실시간으로 추정된 위치 좌표와 이동 궤적을 저장하여, 로봇의 실제 이동 경로를 지도 위에서 확인할 수 있다.

                저장된 데이터는 추후 평가 및 검토 과정에서 로봇이 정상적으로 경로를 따라 이동했는지, 어느 구간에서 정지 또는 오류가 발생했는지 분석하는 데 활용된다.

            }
        case 5: 
            if(Hardware prototype 제작){
                로봇중에서는 실제로 색깔을 인식하게 해서 가는 로봇은 RGB센서및 이미지 센서를 통해서 구현해도된다. 하드웨어 같은 경우 다양한 로봇 및 모빌리티 형태들이 존재한다. 예를들어

                case 1:
                (미국)보스턴 다이나믹스 : SPOT
                https://www.irobotnews.com/news/articleView.html?idxno=22939
                case 2:
                (중국)유니트리 :  Unitree Go2 등등
                https://www.irobotnews.com/news/articleView.html?idxno=32211

                case 3:
                (한국)현*기차 : 모베드
                https://www.gpkorea.com/news/articleView.html?idxno=137143#google_vignette

                case 4:
                라인트레이서 모듈 + Donkey car를 합친 형태로 동작시켜도 가능하다.

                case 5:
                저가형 라인트레이싱 로봇
                https://m.funscience.co.kr/product/ai-%EB%9D%BC%EC%9D%B8%ED%8A%B8%EB%A0%88%EC%9D%B4%EC%84%9C-%EB%A7%8C%EB%93%A4%EA%B8%B0-1%EC%9D%B8%EC%9A%A9-15000/942/

                등등 저가형 로봇들도 더 존재하며 각각의 로봇들에게 RGB센서 및 RGBD캠(Real Sence) 를 달아서 동작시켜도 충분히 구분이 가능하다.

                
            }
            elif(S/W 제작 : 아두이노 라인트레이싱 + 상태출력){
                case 1:모두 chatgpt 의 도움을 빌림
                    #define LEFT_SENSOR 2
                    #define CENTER_SENSOR 3
                    #define RIGHT_SENSOR 4

                    #define LEFT_MOTOR_IN1 8
                    #define LEFT_MOTOR_IN2 9
                    #define RIGHT_MOTOR_IN3 10
                    #define RIGHT_MOTOR_IN4 11

                    int sequence_id = 0;
                    String position_state = "START";
                    String robot_status = "STOP";

                    unsigned long lastLogTime = 0;
                    unsigned long logInterval = 500; // 0.5초마다 상태 출력

                    void setup() {
                    Serial.begin(9600);

                    pinMode(LEFT_SENSOR, INPUT);
                    pinMode(CENTER_SENSOR, INPUT);
                    pinMode(RIGHT_SENSOR, INPUT);

                    pinMode(LEFT_MOTOR_IN1, OUTPUT);
                    pinMode(LEFT_MOTOR_IN2, OUTPUT);
                    pinMode(RIGHT_MOTOR_IN3, OUTPUT);
                    pinMode(RIGHT_MOTOR_IN4, OUTPUT);

                    stopMotor();

                    Serial.println("sequence_id,position_state,line_state,robot_status");
                    }

                    void loop() {
                    int left = digitalRead(LEFT_SENSOR);
                    int center = digitalRead(CENTER_SENSOR);
                    int right = digitalRead(RIGHT_SENSOR);

                    String line_state = "";

                    /*
                        센서 모듈에 따라 검은색 감지값이 LOW일 수도 있고 HIGH일 수도 있음.
                        여기서는 검은색 감지 = LOW 라고 가정.
                        만약 반대로 동작하면 LOW/HIGH 조건만 바꾸면 됨.
                    */

                    if (center == LOW && left == HIGH && right == HIGH) {
                        forward();
                        line_state = "CENTER";
                        robot_status = "MOVING";
                        position_state = "STRAIGHT";
                    }
                    else if (left == LOW && center == HIGH) {
                        turnLeft();
                        line_state = "LEFT";
                        robot_status = "MOVING";
                        position_state = "CURVE_LEFT";
                    }
                    else if (right == LOW && center == HIGH) {
                        turnRight();
                        line_state = "RIGHT";
                        robot_status = "MOVING";
                        position_state = "CURVE_RIGHT";
                    }
                    else if (left == LOW && center == LOW && right == LOW) {
                        stopMotor();
                        line_state = "CROSS";
                        robot_status = "STOP_POINT";

                        sequence_id++;
                        position_state = "POINT_" + String(sequence_id);

                        delay(500);
                    }
                    else {
                        stopMotor();
                        line_state = "NONE";
                        robot_status = "LINE_LOST";
                        position_state = "UNKNOWN";
                    }

                    if (millis() - lastLogTime >= logInterval) {
                        lastLogTime = millis();

                        Serial.print(sequence_id);
                        Serial.print(",");
                        Serial.print(position_state);
                        Serial.print(",");
                        Serial.print(line_state);
                        Serial.print(",");
                        Serial.println(robot_status);
                    }
                    }

                    void forward() {
                    digitalWrite(LEFT_MOTOR_IN1, HIGH);
                    digitalWrite(LEFT_MOTOR_IN2, LOW);
                    digitalWrite(RIGHT_MOTOR_IN3, HIGH);
                    digitalWrite(RIGHT_MOTOR_IN4, LOW);
                    }

                    void turnLeft() {
                    digitalWrite(LEFT_MOTOR_IN1, LOW);
                    digitalWrite(LEFT_MOTOR_IN2, LOW);
                    digitalWrite(RIGHT_MOTOR_IN3, HIGH);
                    digitalWrite(RIGHT_MOTOR_IN4, LOW);
                    }

                    void turnRight() {
                    digitalWrite(LEFT_MOTOR_IN1, HIGH);
                    digitalWrite(LEFT_MOTOR_IN2, LOW);
                    digitalWrite(RIGHT_MOTOR_IN3, LOW);
                    digitalWrite(RIGHT_MOTOR_IN4, LOW);
                    }

                    void stopMotor() {
                    digitalWrite(LEFT_MOTOR_IN1, LOW);
                    digitalWrite(LEFT_MOTOR_IN2, LOW);
                    digitalWrite(RIGHT_MOTOR_IN3, LOW);
                    digitalWrite(RIGHT_MOTOR_IN4, LOW);
                }

                elif(파이썬 시간·위치 데이터 기록 코드){
                    import serial
                    import csv
                    from datetime import datetime

                    SERIAL_PORT = "COM3"   # 윈도우 예시: COM3, COM4
                    BAUD_RATE = 9600
                    CSV_FILE = "robot_movement_log.csv"

                    def main():
                        try:
                            arduino = serial.Serial(SERIAL_PORT, BAUD_RATE, timeout=1)
                            print("Serial connected:", SERIAL_PORT)
                        except Exception as e:
                            print("Serial connection failed:", e)
                            return

                        with open(CSV_FILE, mode="w", newline="", encoding="utf-8-sig") as file:
                            writer = csv.writer(file)

                            writer.writerow([
                                "timestamp",
                                "sequence_id",
                                "position_state",
                                "line_state",
                                "robot_status"
                            ])

                            while True:
                                try:
                                    line = arduino.readline().decode("utf-8").strip()

                                    if not line:
                                        continue

                                    if line.startswith("sequence_id"):
                                        continue

                                    data = line.split(",")

                                    if len(data) != 4:
                                        print("Invalid data:", line)
                                        continue

                                    sequence_id = data[0]
                                    position_state = data[1]
                                    line_state = data[2]
                                    robot_status = data[3]

                                    timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S.%f")[:-3]

                                    writer.writerow([
                                        timestamp,
                                        sequence_id,
                                        position_state,
                                        line_state,
                                        robot_status
                                    ])

                                    file.flush()

                                    print(timestamp, sequence_id, position_state, line_state, robot_status)

                                except KeyboardInterrupt:
                                    print("Logging stopped.")
                                    break

                                except Exception as e:
                                    print("Error:", e)

                        arduino.close()

                    if __name__ == "__main__":
                        main()
                }

                이걸 진행하려면 linux를 통해 진행 ros를 사용합니다.
            }

    }
