# ROS2 패키지 및 Python 빌드 시스템 조사

## 1. 수행 목표

본 실습의 목표는 Ubuntu 22.04와 ROS 2 Humble 환경에서 ROS 2 워크스페이스와 패키지의 구조를 이해하고, Python 기반 ROS 2 패키지를 생성·빌드·실행하는 방법을 학습하는 것이다.

주요 학습 내용은 다음과 같다.

- ROS 2 워크스페이스와 `src` 디렉토리의 역할 이해
- `colcon` 빌드 도구의 개념과 주요 명령어 조사
- `ros2 pkg create` 명령을 사용한 패키지 생성
- Python용 빌드 시스템인 `ament_python` 이해
- ROS 2 Python 클라이언트 라이브러리인 `rclpy` 이해
- `package.xml`, `setup.py`, `setup.cfg` 파일의 구조와 역할 이해
- `tree` 명령을 사용한 워크스페이스 구조 확인

---

## 2. 개발 환경

| 구분 | 사용 환경 |
|---|---|
| 운영체제 | Ubuntu 22.04.x LTS |
| 셸 | Bash |
| ROS 2 배포판 | ROS 2 Humble Hawksbill |
| 프로그래밍 언어 | Python 3 |
| 빌드 도구 | colcon |
| ROS 2 빌드 시스템 | ament_python |
| Python 클라이언트 라이브러리 | rclpy |

---

## 3. ROS 2 환경 설정

ROS 2 명령어를 사용하기 전에 현재 터미널에 ROS 2 Humble 환경을 적용해야 한다.

```bash
source /opt/ros/humble/setup.bash
```

매번 터미널을 열 때마다 자동으로 적용하려면 다음 명령을 실행한다.

```bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

설정 여부는 다음과 같이 확인할 수 있다.

```bash
printenv ROS_DISTRO
```

정상적인 경우 다음과 같이 출력된다.

```text
humble
```

`ros2: command not found` 또는 ROS 2 패키지를 찾을 수 없다는 오류가 발생하면 `/opt/ros/humble/setup.bash`를 source하지 않았을 가능성이 크다.

---

## 4. ROS 2 워크스페이스 생성

### 4.1 워크스페이스란?

ROS 2 워크스페이스는 하나 이상의 ROS 2 패키지를 개발하고 빌드하기 위한 작업 공간이다. 일반적으로 워크스페이스 루트 아래에 `src` 디렉토리를 만들고, 개발할 패키지는 모두 `src` 내부에 배치한다.

본 실습에서는 워크스페이스 이름을 `ros2_ws`로 사용하였다.

### 4.2 디렉토리 생성

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws
```

생성 결과는 다음과 같다.

```text
ros2_ws/
└── src/
```

`mkdir -p`에서 `-p` 옵션은 상위 디렉토리가 없을 경우 함께 생성하고, 이미 디렉토리가 존재하더라도 오류를 발생시키지 않도록 한다.

---

## 5. colcon 조사

### 5.1 colcon의 개념

`colcon`은 ROS 2 워크스페이스 안의 여러 패키지를 검색하고, 패키지 간 의존관계를 고려하여 적절한 순서로 빌드하는 명령행 도구이다.

ROS 2 패키지 자체의 빌드 방식은 `ament_cmake`, `ament_python` 등으로 나뉘지만, 워크스페이스 전체 빌드는 일반적으로 `colcon`이 담당한다.

### 5.2 colcon 설치 확인

```bash
colcon --help
```

명령어가 없을 경우 다음 패키지를 설치한다.

```bash
sudo apt update
sudo apt install python3-colcon-common-extensions
```

설치 후 다시 확인한다.

```bash
colcon --help
```

### 5.3 빈 워크스페이스 빌드

워크스페이스 루트에서 다음 명령을 실행한다.

```bash
cd ~/ros2_ws
colcon build
```

주의할 점은 `colcon build`가 패키지를 생성하는 명령은 아니라는 것이다. 이 명령은 `src`에서 패키지를 검색하여 빌드한다. `src`가 비어 있다면 빌드할 패키지가 없으며, 환경에 따라 `build`, `install`, `log` 디렉토리가 만들어질 수 있다.

ROS 2 패키지는 `ros2 pkg create` 명령으로 생성한다.

### 5.4 주요 colcon 명령 및 옵션

| 명령 또는 옵션 | 설명 |
|---|---|
| `colcon build` | 워크스페이스의 패키지를 빌드한다. |
| `colcon test` | 빌드된 패키지의 테스트를 수행한다. |
| `colcon test-result` | 테스트 결과를 확인한다. |
| `--packages-select 패키지명` | 지정한 패키지만 빌드한다. |
| `--packages-up-to 패키지명` | 지정 패키지와 그 의존 패키지를 함께 빌드한다. |
| `--symlink-install` | Python 파일 등을 복사하지 않고 심볼릭 링크로 설치하여 수정 내용을 빠르게 반영한다. |
| `--event-handlers console_direct+` | 빌드 로그를 터미널에 바로 자세히 출력한다. |

개발 중에는 다음 명령이 편리하다.

```bash
colcon build --symlink-install
```

---

## 6. ROS 2 패키지 생성

### 6.1 ros2 pkg create 명령

기본 형식은 다음과 같다.

```bash
ros2 pkg create [옵션] 패키지명
```

주요 옵션은 다음과 같다.

| 옵션 | 설명 |
|---|---|
| `--build-type` | 사용할 빌드 시스템을 지정한다. |
| `--dependencies` | 패키지가 의존하는 ROS 2 패키지를 지정한다. |
| `--node-name` | 패키지 생성 시 기본 노드 파일도 함께 만든다. |
| `--license` | 패키지 라이선스를 지정한다. |
| `--description` | 패키지 설명을 입력한다. |
| `--maintainer-name` | 관리자 이름을 지정한다. |
| `--maintainer-email` | 관리자 이메일을 지정한다. |

명령어 도움말은 다음과 같이 확인한다.

```bash
ros2 pkg create --help
```

### 6.2 my_robot_controller 패키지 생성

ROS 2 패키지는 워크스페이스 루트가 아니라 `src` 디렉토리에서 생성한다.

```bash
cd ~/ros2_ws/src
ros2 pkg create \
  --build-type ament_python \
  --dependencies rclpy \
  --license Apache-2.0 \
  my_robot_controller
```

한 줄로 입력하면 다음과 같다.

```bash
ros2 pkg create --build-type ament_python --dependencies rclpy --license Apache-2.0 my_robot_controller
```

이 명령은 다음 내용을 의미한다.

- 패키지 이름: `my_robot_controller`
- 빌드 시스템: `ament_python`
- 의존 패키지: `rclpy`
- 라이선스: `Apache-2.0`

`--dependencies rclpy`를 지정하면 `package.xml`에 `rclpy` 의존성이 자동으로 추가된다.

---

## 7. ament_python 빌드 시스템

### 7.1 개념

`ament`는 ROS 2 패키지의 빌드와 설치를 지원하는 빌드 시스템 및 관련 도구의 집합이다.

ROS 2에서 대표적으로 사용하는 빌드 유형은 다음과 같다.

| 빌드 유형 | 주 사용 언어 및 용도 |
|---|---|
| `ament_cmake` | C++ 중심 ROS 2 패키지 |
| `ament_python` | 순수 Python 중심 ROS 2 패키지 |
| `ament_cmake_python` | CMake 패키지 내부에서 Python 코드도 함께 설치할 때 사용 |

`ament_python`은 Python의 `setuptools` 구조를 기반으로 ROS 2 Python 패키지를 설치한다. 따라서 패키지 루트에 `setup.py`, `setup.cfg`, `package.xml` 등이 생성된다.

맞아. 한 ROS 2 패키지 안에서 C++ 코드와 Python 코드를 함께 포함하고 설치할 때 ament_cmake_python을 사용할 수 있어.

다만 “C++과 Python이 자동으로 동시에 실행된다”는 뜻은 아니야.

예를 들면 패키지 하나에 이렇게 들어갈 수 있어.

my_robot_package/
├── CMakeLists.txt
├── package.xml
├── src/
│   └── camera_node.cpp
├── include/
│   └── my_robot_package/
└── my_robot_package/
    ├── __init__.py
    └── analysis_node.py

여기서:

camera_node.cpp는 C++ ROS 2 노드
analysis_node.py는 Python ROS 2 노드
CMakeLists.txt는 C++ 빌드와 Python 파일 설치를 모두 설정
ament_cmake_python은 Python 모듈을 ROS 2 설치 공간에 넣도록 도와줌

빌드 후 각각 따로 실행할 수 있어.

ros2 run my_robot_package camera_node
ros2 run my_robot_package analysis_node

두 노드를 동시에 실행하고 싶다면 각각 터미널에서 실행하거나 launch 파일을 사용해야 해.

C++ 카메라 노드
       │ 이미지 토픽 발행
       ▼
Python 분석 노드
       │ 이미지 토픽 구독
       ▼
객체 탐지 및 결과 처리

즉, 두 언어가 서로 직접 섞여서 한 프로그램처럼 동작한다기보다는, 보통은 C++ 노드와 Python 노드를 각각 만들고 ROS 2 토픽·서비스·액션으로 통신하게 만드는 방식이야.

예를 들어 네 프로젝트라면:

C++ 노드
- FLIR 카메라 고속 데이터 수집
- 영상 처리
- 하드웨어 제어

Python 노드
- YOLO 객체 탐지
- 온도 분석
- DB 저장
- Flask 서버 연동

이렇게 역할을 나눌 수 있어.

정리하면:

빌드 유형	의미
ament_cmake	주로 C++ 코드 빌드
ament_python	순수 Python 패키지
ament_cmake_python	CMake 기반 패키지에서 C++과 Python 코드를 함께 관리·설치

과제 문서의 문장은 아래처럼 바꾸면 더 정확해.

ament_cmake_python은 CMake 기반 ROS 2 패키지에서 C++ 코드와 Python 모듈을 함께 구성하고 설치할 때 사용하는 방식이다. 단, C++과 Python 코드가 자동으로 동시에 실행되는 것은 아니며, 각각의 노드로 실행하거나 launch 파일을 이용해 함께 실행할 수 있다.

네 프로젝트에서는 이런 구성이 현실적이야.

노드	언어	이유
FLIR 영상 수집	C++	빠른 처리와 카메라 SDK 연동
YOLO 객체 탐지	Python	PyTorch와 YOLO 사용이 편리함
온도 위험도 판단	Python	분석 로직 작성이 편리함
DB 저장 및 웹 API	Python	MariaDB, Flask 연동이 편리함
하드웨어 실시간 제어	C++	일정하고 빠른 제어에 유리함

### 7.2 ament_python을 사용하는 이유

- Python으로 ROS 2 노드를 쉽게 개발할 수 있다.
- Python 표준 패키징 방식인 `setuptools`를 활용한다.
- `setup.py`의 `console_scripts`를 통해 `ros2 run`으로 실행 가능한 명령을 등록할 수 있다.
- `colcon build`와 연동되어 워크스페이스 단위로 패키지를 빌드할 수 있다.
- `--symlink-install`을 사용하면 Python 소스 수정 결과를 빠르게 확인할 수 있다.

---

## 8. rclpy 조사

### 8.1 rclpy란?

`rclpy`는 ROS 2에서 Python 프로그램이 ROS 2 기능을 사용할 수 있게 해주는 Python 클라이언트 라이브러리이다.

C++에서 사용하는 `rclcpp`와 대응되는 Python용 라이브러리이다.

### 8.2 rclpy의 주요 용도

`rclpy`를 사용하면 Python 코드에서 다음 기능을 구현할 수 있다.

- ROS 2 노드 생성
- 토픽 Publisher 생성
- 토픽 Subscriber 생성
- Service 서버 및 Client 생성
- Action 서버 및 Client 생성
- Parameter 선언 및 사용
- Timer와 Callback 실행
- Logger를 통한 로그 출력
- ROS 2 Context 초기화 및 종료
- Executor를 통한 Callback 처리

### 8.3 기본 사용 흐름

```python
import rclpy
from rclpy.node import Node


class RobotController(Node):

    def __init__(self):
        super().__init__('robot_controller')
        self.get_logger().info('제어 노드가 시작되었습니다.')


def main(args=None):
    rclpy.init(args=args)

    node = RobotController()
    rclpy.spin(node)

    node.destroy_node()
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

주요 함수와 클래스의 역할은 다음과 같다.

| 항목 | 역할 |
|---|---|
| `rclpy.init()` | ROS 2 Python 통신 환경을 초기화한다. |
| `Node` | ROS 2 노드를 구현하기 위한 기본 클래스이다. |
| `rclpy.spin(node)` | 노드를 계속 실행하며 콜백을 처리한다. |
| `destroy_node()` | 노드가 사용한 자원을 해제한다. |
| `rclpy.shutdown()` | ROS 2 Python 환경을 종료한다. |
| `get_logger()` | ROS 2 로그 메시지를 출력한다. |

---

## 9. 생성된 패키지 구조

`my_robot_controller` 패키지를 생성하면 기본적으로 다음과 유사한 구조가 만들어진다.

```text
my_robot_controller/
├── my_robot_controller/
│   └── __init__.py
├── package.xml
├── resource/
│   └── my_robot_controller
├── setup.cfg
├── setup.py
└── test/
    ├── test_copyright.py
    ├── test_flake8.py
    └── test_pep257.py
```

본 제출용 예시에는 실행 확인을 위한 `controller_node.py`도 추가하였다.

```text
my_robot_controller/
├── my_robot_controller/
│   ├── __init__.py
│   └── controller_node.py
├── package.xml
├── resource/
│   └── my_robot_controller
├── setup.cfg
├── setup.py
└── test/
    ├── test_copyright.py
    ├── test_flake8.py
    └── test_pep257.py
```

---

## 10. package.xml 파일

### 10.1 package.xml의 역할

`package.xml`은 ROS 2 패키지의 메타데이터와 의존성을 기록하는 패키지 매니페스트 파일이다. 모든 ament 패키지는 패키지 루트에 하나의 `package.xml`을 가져야 한다.

주요 기록 내용은 다음과 같다.

- 패키지 이름
- 버전
- 설명
- 관리자 이름과 이메일
- 라이선스
- 빌드 및 실행 의존성
- 테스트 의존성
- 빌드 시스템 유형

### 10.2 package.xml 예시

```xml
<?xml version="1.0"?>
<package format="3">
  <name>my_robot_controller</name>
  <version>0.0.0</version>
  <description>ROS 2 Python 로봇 제어 패키지</description>

  <maintainer email="student@example.com">Kim Hanmin</maintainer>
  <license>Apache-2.0</license>

  <depend>rclpy</depend>

  <test_depend>ament_copyright</test_depend>
  <test_depend>ament_flake8</test_depend>
  <test_depend>ament_pep257</test_depend>
  <test_depend>python3-pytest</test_depend>

  <export>
    <build_type>ament_python</build_type>
  </export>
</package>
```
package.xml은 쉽게 말하면 이 ROS 2 패키지의 설명서이자 명세서야.

ROS 2가 이 파일을 보고:

패키지 이름이 무엇인지
어떤 기능의 패키지인지
누가 만들었는지
어떤 라이브러리가 필요한지
어떤 방식으로 빌드해야 하는지

를 확인한다.

각 항목은 이렇게 보면 돼.

<?xml version="1.0"?>

이 파일이 XML 형식으로 작성되었다는 뜻이다.

<package format="3">

ROS 패키지 형식 버전 3을 사용한다는 뜻이다.

<name>my_robot_controller</name>

패키지 이름이다.

<version>0.0.0</version>

패키지 버전이다. 처음 만든 상태라 보통 0.0.0으로 시작한다.

<description>ROS 2 Python 로봇 제어 패키지</description>

이 패키지가 무슨 역할을 하는지 설명한다.

<maintainer email="student@example.com">Kim Hanmin</maintainer>

패키지를 관리하는 사람의 이름과 이메일이다.

<license>Apache-2.0</license>

이 패키지를 어떤 조건으로 사용하거나 배포할 수 있는지를 나타내는 라이선스다.

<depend>rclpy</depend>

이 패키지가 실행되려면 rclpy가 필요하다는 뜻이다.

즉, ROS 2에게 이렇게 알려주는 것이다.

이 패키지는 Python용 ROS 2 라이브러리인 rclpy를 사용합니다.

<test_depend>ament_copyright</test_depend>

라이선스와 저작권 표기가 올바른지 검사할 때 사용하는 패키지다.

<test_depend>ament_flake8</test_depend>

Python 코드 스타일에 문제가 없는지 검사한다.

예를 들어 들여쓰기, 띄어쓰기, 너무 긴 줄 등을 확인한다.

<test_depend>ament_pep257</test_depend>

Python 함수나 클래스의 설명문인 docstring이 규칙에 맞는지 검사한다.

<test_depend>python3-pytest</test_depend>

Python 테스트 코드를 실행하기 위한 테스트 도구다.

<export>
  <build_type>ament_python</build_type>
</export>

이 패키지는 Python 방식인 ament_python으로 빌드해야 한다는 뜻이다.

마지막의:

</package>

는 패키지 정보가 여기서 끝났다는 뜻이다.

전체적으로 보면 다음처럼 이해하면 된다.

package.xml

패키지 이름: my_robot_controller
패키지 버전: 0.0.0
설명: ROS 2 Python 로봇 제어 패키지
관리자: Kim Hanmin
필요한 라이브러리: rclpy
테스트 도구: flake8, pytest 등
빌드 방식: ament_python

한 문장으로 정리하면:

package.xml은 ROS 2에게 이 패키지가 무엇이고, 무엇이 필요하며, 어떤 방식으로 빌드해야 하는지를 알려주는 파일이다.

### 10.3 주요 태그

| 태그 | 역할 |
|---|---|
| `<name>` | 패키지 이름 |
| `<version>` | 패키지 버전 |
| `<description>` | 패키지 설명 |
| `<maintainer>` | 유지관리자 이름과 이메일 |
| `<license>` | 적용 라이선스 |
| `<depend>` | 빌드와 실행에 모두 필요한 의존성 |
| `<build_depend>` | 빌드 시 필요한 의존성 |
| `<exec_depend>` | 실행 시 필요한 의존성 |
| `<test_depend>` | 테스트 수행 시 필요한 의존성 |
| `<export>` | 빌드 시스템 등 추가 정보를 내보내는 영역 |
| `<build_type>` | 패키지의 빌드 유형 |

`rclpy`는 실행 시 반드시 필요하므로 다음과 같이 기록한다.

```xml
<depend>rclpy</depend>
```

Python 패키지임을 나타내기 위해 다음 항목이 필요하다.

```xml
<export>
  <build_type>ament_python</build_type>
</export>
```

---

## 11. setup.py 파일

### 11.1 setup.py의 역할

`setup.py`는 Python 패키지의 설치 방법과 실행 가능한 노드 등록 방법을 정의한다.

ROS 2에서 `ament_python` 패키지를 빌드할 때 `colcon`은 `setup.py` 정보를 사용하여 다음 작업을 수행한다.

- Python 모듈 설치
- `package.xml` 및 resource 파일 설치
- 패키지 이름과 버전 등록
- 의존 Python 패키지 지정
- `ros2 run`에서 사용할 실행 명령 등록

### 11.2 setup.py 예시

```python
from setuptools import find_packages, setup

package_name = 'my_robot_controller'

setup(
    name=package_name,
    version='0.0.0',
    packages=find_packages(exclude=['test']),
    data_files=[
        (
            'share/ament_index/resource_index/packages',
            ['resource/' + package_name]
        ),
        (
            'share/' + package_name,
            ['package.xml']
        ),
    ],
    install_requires=['setuptools'],
    zip_safe=True,
    maintainer='Kim Hanmin',
    maintainer_email='student@example.com',
    description='ROS 2 Python 로봇 제어 패키지',
    license='Apache-2.0',
    tests_require=['pytest'],
    entry_points={
        'console_scripts': [
            'controller_node = my_robot_controller.controller_node:main',
        ],
    },
)
```

### 11.3 주요 항목

| 항목 | 역할 |
|---|---|
| `name` | 설치되는 Python 패키지 이름 |
| `version` | 패키지 버전 |
| `packages` | 설치할 Python 모듈 검색 |
| `data_files` | `package.xml`, resource 파일 등의 설치 위치 지정 |
| `install_requires` | Python 패키지 설치 의존성 |
| `maintainer` | 관리자 이름 |
| `maintainer_email` | 관리자 이메일 |
| `description` | 패키지 설명 |
| `license` | 라이선스 |
| `tests_require` | 테스트에 필요한 Python 패키지 |
| `entry_points` | 실행 가능한 콘솔 명령 등록 |

다음 설정이 가장 중요하다.

```python
entry_points={
    'console_scripts': [
        'controller_node = my_robot_controller.controller_node:main',
    ],
},
```

이 설정은 다음 관계를 의미한다.

```text
ros2 run에서 사용할 실행 이름
        ↓
controller_node
        ↓
Python 모듈
my_robot_controller.controller_node
        ↓
실행 함수
main
```

빌드 후에는 다음 명령으로 노드를 실행할 수 있다.

```bash
ros2 run my_robot_controller controller_node
```

---

## 12. setup.cfg 파일

`setup.cfg`는 ROS 2가 Python 실행 스크립트를 패키지별 경로에 설치하도록 지정한다.

```ini
[develop]
script_dir=$base/lib/my_robot_controller

[install]
install_scripts=$base/lib/my_robot_controller
```

이 파일이 올바르게 설정되어야 `ros2 run 패키지명 실행파일명` 형식으로 Python 노드를 찾을 수 있다.

---

## 13. resource 디렉토리

```text
resource/
└── my_robot_controller
```

resource 파일은 보통 내용이 없는 빈 파일이지만, ROS 2의 ament index가 설치된 패키지를 검색할 수 있도록 패키지 이름을 등록하는 역할을 한다.

`setup.py`의 다음 부분이 resource 파일의 설치 위치를 지정한다.

```python
(
    'share/ament_index/resource_index/packages',
    ['resource/' + package_name]
),
```

---

## 14. 패키지 빌드

패키지를 만든 후 워크스페이스 루트로 이동한다.

```bash
cd ~/ros2_ws
```

의존성을 확인하고 설치한다.

```bash
rosdep install --from-paths src --ignore-src -r -y
```

패키지를 빌드한다.

```bash
colcon build --symlink-install
```

특정 패키지만 빌드하려면 다음 명령을 사용한다.

```bash
colcon build --symlink-install --packages-select my_robot_controller
```

빌드가 완료되면 워크스페이스 환경을 현재 터미널에 적용한다.

```bash
source install/setup.bash
```

ROS 2 기본 환경과 현재 워크스페이스 환경을 자동 적용하려면 `~/.bashrc`에 다음 내용을 추가할 수 있다.

```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
```

단, 아직 한 번도 빌드하지 않아 `install/setup.bash`가 없는 상태라면 두 번째 줄에서 오류가 발생할 수 있다.

---

## 15. 노드 실행

예시 노드가 `setup.py`의 `console_scripts`에 등록되어 있다면 다음 명령으로 실행한다.

```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
ros2 run my_robot_controller controller_node
```

예상 출력은 다음과 같다.

```text
[INFO] [시간] [robot_controller]: my_robot_controller 노드가 실행되었습니다.
```

다른 터미널에서 실행 중인 노드를 확인한다.

```bash
ros2 node list
```

예상 출력:

```text
/robot_controller
```

노드를 종료할 때는 실행 터미널에서 `Ctrl+C`를 누른다.

---

## 16. tree 프로그램 설치 및 실행

### 16.1 설치

```bash
sudo apt update
sudo apt install tree
```

설치 확인:

```bash
tree --version
```

### 16.2 src 디렉토리 확인

```bash
cd ~/ros2_ws
tree src
```

예상 결과:

```text
src
└── my_robot_controller
    ├── my_robot_controller
    │   ├── __init__.py
    │   └── controller_node.py
    ├── package.xml
    ├── resource
    │   └── my_robot_controller
    ├── setup.cfg
    ├── setup.py
    └── test
        ├── test_copyright.py
        ├── test_flake8.py
        └── test_pep257.py

5 directories, 9 files
```

### 16.3 빌드 후 워크스페이스 확인

```bash
cd ~/ros2_ws
tree -L 2
```
```bash
tree src
→ src 내부를 자세히 확인
→ 패키지 파일 구조 확인용
```
tree -L 2
→ 현재 폴더에서 2단계까지만 확인
→ 워크스페이스 전체 구조 확인용

예상 결과:

```text
.
├── build
│   └── my_robot_controller
├── install
│   ├── COLCON_IGNORE
│   ├── _local_setup_util_ps1.py
│   ├── _local_setup_util_sh.py
│   ├── local_setup.bash
│   ├── local_setup.ps1
│   ├── local_setup.sh
│   ├── local_setup.zsh
│   ├── my_robot_controller
│   ├── setup.bash
│   ├── setup.ps1
│   ├── setup.sh
│   └── setup.zsh
├── log
│   ├── COLCON_IGNORE
│   ├── build_날짜_시간
│   ├── latest -> latest_build
│   └── latest_build -> build_날짜_시간
└── src
    └── my_robot_controller
```

환경과 `colcon` 버전에 따라 세부 파일명은 달라질 수 있다.

### 16.4 빌드 디렉토리의 역할

| 디렉토리 | 역할 |
|---|---|
| `src` | 사용자가 작성한 ROS 2 패키지 원본 코드 |
| `build` | 패키지별 빌드 과정에서 생성되는 중간 파일 |
| `install` | 빌드가 완료되어 실행 가능한 형태로 설치된 결과 |
| `log` | 빌드 및 테스트 과정의 로그 |

---

## 17. 테스트 수행

패키지 테스트는 워크스페이스 루트에서 실행한다.

```bash
cd ~/ros2_ws
colcon test --packages-select my_robot_controller
```

테스트 결과를 자세히 확인한다.

```bash
colcon test-result --verbose
```

테스트 관련 오류가 발생하면 먼저 다음 명령으로 필요한 패키지가 설치되었는지 확인한다.

```bash
rosdep install --from-paths src --ignore-src -r -y
```

---

## 18. src 디렉토리 압축

과제에서는 워크스페이스 전체가 아니라 워크스페이스의 `src` 디렉토리를 압축하여 제출하도록 요구하고 있다.

### 18.1 zip 설치

```bash
sudo apt update
sudo apt install zip
```

### 18.2 src 압축

워크스페이스 루트에서 실행한다.

```bash
cd ~/ros2_ws
zip -r ros2_ws_src.zip src
```

압축 파일 확인:

```bash
ls -lh ros2_ws_src.zip
```

압축 내용 확인:

```bash
unzip -l ros2_ws_src.zip
```

`build`, `install`, `log` 디렉토리는 빌드할 때 다시 생성할 수 있고 용량이 크므로, 과제 지시가 `src` 압축인 경우 포함하지 않는다.

---

## 19. 프로젝트 제출 디렉토리 구성

프로젝트 루트에 `2/4` 디렉토리를 생성한다.

```bash
cd ~/프로젝트_루트
mkdir -p 2/4
```

조사 문서를 복사한다.

```bash
cp ~/ros2_ws/4_ros2_package.md 2/4/
```

압축 파일을 복사한다.

```bash
cp ~/ros2_ws/ros2_ws_src.zip 2/4/
```

최종 제출 구조는 다음과 같이 구성한다.

```text
프로젝트_루트/
└── 2/
    └── 4/
        ├── 4_ros2_package.md
        └── ros2_ws_src.zip
```

실습 화면 캡처가 필요한 경우 다음 항목을 추가하면 평가자가 수행 과정을 확인하기 쉽다.

- `printenv ROS_DISTRO` 결과
- `ros2 pkg create` 실행 결과
- `tree src` 실행 결과
- `colcon build --symlink-install` 성공 결과
- `ros2 run my_robot_controller controller_node` 실행 결과
- `ros2 node list` 결과

---

## 20. 전체 실습 명령 요약

아래 명령을 순서대로 실행하면 워크스페이스 생성부터 패키지 빌드까지 진행할 수 있다.

```bash
# 1. ROS 2 Humble 환경 적용
source /opt/ros/humble/setup.bash

# 2. 필요한 도구 설치
sudo apt update
sudo apt install -y python3-colcon-common-extensions python3-rosdep tree zip

# 3. 워크스페이스 생성
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws

# 4. 빈 워크스페이스 빌드 확인
colcon build

# 5. Python 패키지 생성
cd ~/ros2_ws/src
ros2 pkg create \
  --build-type ament_python \
  --dependencies rclpy \
  --license Apache-2.0 \
  my_robot_controller

# 6. 패키지 구조 확인
cd ~/ros2_ws
tree src

# 7. 의존성 설치
rosdep install --from-paths src --ignore-src -r -y

# 8. 패키지 빌드
colcon build --symlink-install

# 9. 워크스페이스 환경 적용
source install/setup.bash

# 10. 패키지 확인
ros2 pkg list | grep my_robot_controller

# 11. 노드 실행
ros2 run my_robot_controller controller_node

# 12. src 압축
cd ~/ros2_ws
zip -r ros2_ws_src.zip src
```

패키지 생성 직후에는 `controller_node.py`와 `setup.py`의 `console_scripts` 등록이 자동으로 포함되지 않을 수 있다. 노드 실행 실습까지 진행하려면 본 문서의 예시처럼 두 항목을 직접 추가해야 한다.

---

## 21. 자주 발생하는 오류와 해결 방법

### 21.1 ros2 명령어가 없는 경우

오류:

```text
ros2: command not found
```

해결:

```bash
source /opt/ros/humble/setup.bash
```

자동 적용:

```bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### 21.2 colcon 명령어가 없는 경우

오류:

```text
colcon: command not found
```

해결:

```bash
sudo apt update
sudo apt install python3-colcon-common-extensions
```

### 21.3 패키지를 찾지 못하는 경우

오류:

```text
Package 'my_robot_controller' not found
```

해결:

```bash
cd ~/ros2_ws
colcon build --symlink-install
source install/setup.bash
```

새 터미널을 열었다면 다시 source해야 한다.

### 21.4 실행 파일을 찾지 못하는 경우

오류:

```text
No executable found
```

확인할 사항:

1. `setup.py`의 `entry_points`에 실행 항목이 등록되어 있는지 확인한다.
2. Python 파일에 `main()` 함수가 존재하는지 확인한다.
3. 패키지를 다시 빌드한다.
4. `install/setup.bash`를 다시 source한다.

```bash
cd ~/ros2_ws
colcon build --symlink-install --packages-select my_robot_controller
source install/setup.bash
```

### 21.5 build 결과가 이전 상태로 남은 경우

```bash
cd ~/ros2_ws
rm -rf build install log
colcon build --symlink-install
source install/setup.bash
```

이 명령은 빌드 결과를 삭제하므로 반드시 워크스페이스 루트에서 정확히 실행해야 한다. `src` 디렉토리는 삭제하지 않는다.

---

## 22. 결론

본 실습에서는 ROS 2 Humble 환경에서 `ros2_ws` 워크스페이스와 `src` 디렉토리를 생성하고, `ros2 pkg create` 명령을 이용하여 `ament_python` 기반의 `my_robot_controller` 패키지를 생성하였다.

`colcon`은 워크스페이스 내부 패키지의 검색, 의존성 순서 결정, 빌드 및 테스트를 담당한다. `ament_python`은 Python 기반 ROS 2 패키지를 설치하기 위한 빌드 유형이며, `rclpy`는 Python 코드에서 노드, 토픽, 서비스, 액션, 파라미터 등의 ROS 2 기능을 사용할 수 있게 한다.

또한 `package.xml`은 패키지의 메타데이터, 의존성 및 빌드 유형을 기록하고, `setup.py`는 Python 모듈의 설치와 실행 명령을 정의한다. 빌드 후 생성되는 `build`, `install`, `log` 디렉토리의 용도를 확인하였으며, 제출을 위해 `src` 디렉토리를 별도의 ZIP 파일로 압축하는 방법도 실습하였다.

---

## 23. 참고 자료

1. ROS 2 Humble Documentation, Creating a workspace  
   `https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Creating-A-Workspace/Creating-A-Workspace.html`

2. ROS 2 Humble Documentation, Using colcon to build packages  
   `https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Colcon-Tutorial.html`

3. ROS 2 Humble Documentation, Creating a package  
   `https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Creating-Your-First-ROS2-Package.html`

4. ROS 2 Humble Documentation, Writing a simple publisher and subscriber with Python  
   `https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Py-Publisher-And-Subscriber.html`

5. ROS 2 Humble Documentation, About the build system  
   `https://docs.ros.org/en/humble/Concepts/Advanced/About-Build-System.html`

6. ROS 2 Humble Documentation, Configuring environment  
   `https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Configuring-ROS2-Environment.html`

7. rclpy API Documentation  
   `https://docs.ros.org/en/humble/p/rclpy/`
