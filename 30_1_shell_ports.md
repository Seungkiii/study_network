# **Shell Script 실무 연습문제 - 네트워크 프로세스 관리**

## **사용 가능한 문법 제한사항**

- **허용**: if문, 변수, for문, 배열, 기본 shell 명령어
- **금지**: awk, sed
- **핵심 명령어**: ss, lsof, curl, telnet, kill, ps
- 서버 상황

```
[hwang@192.168.0.44 ~/Downloads]$ ps -aux | grep "http"
hwang       6907  0.0  1.0 242444 18688 pts/4    S    17:18   0:00 python3 -m http.server 7000 --bind 0.0.0.0
hwang       6948  0.0  1.0 242444 18688 pts/2    S    17:25   0:00 python3 -m http.server 8000 --bind 0.0.0.0
hwang       6952  0.0  1.0 242444 18816 pts/2    S    17:25   0:00 python3 -m http.server 8080 --bind 0.0.0.0
hwang       6974  0.0  1.0 242444 18816 pts/2    S    17:25   0:00 python3 -m http.server 5731 --bind 0.0.0.0
hwang       6978  0.0  1.0 242444 18688 pts/2    S    17:26   0:00 python3 -m http.server 9000 --bind 0.0.0.0
hwang       6982  0.0  1.0 242444 18688 pts/2    S    17:26   0:00 python3 -m http.server 6000 --bind 0.0.0.0
hwang       6986  0.0  1.0 242444 18688 pts/2    S    17:26   0:00 python3 -m http.server 5000 --bind 0.0.0.0
hwang       7027  0.0  1.0 242444 18688 pts/2    S    17:28   0:00 python3 -m http.server 7777 --bind 0.0.0.0
hwang       7036  0.0  1.0 242444 18688 pts/2    S    17:28   0:00 python3 -m http.server 1234 --bind 0.0.0.0
hwang       7040  0.0  1.0 242444 18688 pts/2    S    17:28   0:00 python3 -m http.server 5678 --bind 0.0.0.0
hwang       7413  0.0  0.1 221796  2304 pts/2    R+   17:46   0:00 grep --color=auto http

```

---

## **연습문제 1: 포트 사용 여부 확인 스크립트**

**문제**: 여러 포트 번호를 인수로 받아서 각 포트가 사용 중인지 확인하는 스크립트를 작성하세요.

**요구사항**:

- 스크립트 실행: ./check_ports.sh 80 443 22 3000 8080
- ss 명령어 사용하여 포트 상태 확인
- 사용 중인 포트는 ✓, 사용하지 않는 포트는 ✗ 표시

**실행 예제**:

$ ./check_ports.sh 80 443 22 3000 8080

포트 사용 상태 확인 중...

포트 80: ✓ 사용 중

포트 443: ✗ 사용 안함

포트 22: ✓ 사용 중

포트 3000: ✗ 사용 안함

포트 8080: ✓ 사용 중

총 5개 포트 중 3개가 사용 중입니다.

```bash

[hwang@192.168.0.44 ~/Downloads]$ source ./check_ports2.sh 8000 8080
Port number is checking...
Port '8000': 사용 중 (In use)
Port '8080': 사용 중 (In use)
포트 확인이 완료되었습니다.
[hwang@192.168.0.44 ~/Downloads]$ cat check_ports2.sh
#!/bin/bash

echo "Port number is checking..."

for i in "$@"; do
    port=$(lsof -i :"$i" 2>/dev/null)

    if [ -n "$port" ]; then
        echo "Port '$i': 사용 중 (In use)"
    else
        echo "Port '$i': 사용 안 함 (Not in use)"
    fi
done

echo "포트 확인이 완료되었습니다."

```

---

## **연습문제 2: 특정 포트 프로세스 종료 스크립트**

**문제**: 특정 포트를 사용하는 프로세스를 찾아서 종료하는 스크립트를 작성하세요.

**요구사항**:

- 스크립트 실행: ./kill_port.sh 8080
- lsof -i :포트번호 사용하여 프로세스 찾기
- 프로세스가 있으면 PID 출력 후 종료
- 프로세스가 없으면 해당 메시지 출력

**실행 및 출력 예제**:

$ ./kill_port.sh 8080

포트 8080 사용 프로세스 검색 중...

발견된 프로세스:

PID: 12345, 프로세스명: node

PID: 12346, 프로세스명: nginx

프로세스 종료 중...

PID 12345 종료 완료

PID 12346 종료 완료

포트 8080이 해제되었습니다.

**프로세스가 없는 경우**:

$ ./kill_port.sh 9999

포트 9999 사용 프로세스 검색 중...

포트 9999를 사용하는 프로세스가 없습니다.

```bash
#!/bin/bash

PORT=$1

echo "포트 $PORT 사용 프로세스 검색 중..."

PIDS=$(lsof -t -i :$PORT)

if [ -z "$PIDS" ]; then
  echo "포트 $PORT를 사용하는 프로세스가 없습니다."
  exit 0
fi

echo
echo "발견된 프로세스:"
# NR>1은 첫번째 줄 제외하고 두번째 줄부터 컬럼 2,1의 값 탐색
lsof -i :$PORT | awk 'NR>1 {print "PID: "$2", 프로세스명: "$1}'

echo
echo "프로세스 종료 중..."

for PID in $PIDS; do
  kill $PID 2>/dev/null
  # $?은 바로 뒤에 명령어의 종료 상태코드 저장-> 성공하면 0, 아니면 다른 값 반환
  if [ $? -eq 0 ]; then
    echo "PID $PID 종료 완료"
  else
    echo "PID $PID 종료 실패"
  fi
done

echo
echo "포트 $PORT이 해제되었습니다."

---

 source ./codes/quests/kill_port.sh 8000
포트 8000 사용 프로세스 검색 중...

발견된 프로세스:
PID: 19572, 프로세스명: Python

프로세스 종료 중...
PID 19572 종료 완료

```

---

## **연습문제 3: 웹 서비스 상태 모니터링 스크립트**

**문제**: 여러 웹 서비스의 상태를 확인하는 모니터링 스크립트를 작성하세요.

**요구사항**:

- 설정 파일 형태로 서버 정보 관리: IP:포트 형식
- curl 명령어로 HTTP 응답 확인 (타임아웃 5초)
- 연결 가능하면 ✓, 불가능하면 ✗ 표시

**실행 예제**:

$ ./monitor_services.sh

웹 서비스 상태 모니터링

========================

서비스 상태 확인 중...

192.168.1.100:80   ✓ 정상 (응답시간: 0.234초)

192.168.1.100:443  ✗ 연결 실패

localhost:3000     ✓ 정상 (응답시간: 0.012초)

192.168.1.200:8080 ✗ 타임아웃

localhost:22       ✗ HTTP 서비스 아님

========================

총 5개 서비스 중 2개 정상

모니터링 완료: 2024-07-28 14:30:15

```bash
#!/bin/bash

echo "웹 서비스 상태 모니터링"
echo "========================"

# 동적으로 http.server 프로세스 목록 추출 (grep으로 필터, grep -v grep으로 자기 자신 제외)
processes=$(ps -aux | grep "http.server" | grep -v grep)

# 프로세스 없음 확인
if [ -z "$processes" ]; then
    echo "현재 실행 중인 http.server가 없습니다."
    echo "========================"
    echo "총 0개 서비스 중 0개 정상"
    echo "모니터링 완료: $(date '+%Y-%m-%d %H:%M:%S')"
    exit 0
fi

echo "서비스 상태 확인 중..."

ports=()
normal_count=0

echo "$processes" | while IFS= read -r line; do
    port=$(echo "$line" | tr -s ' ' | cut -d' ' -f13)  

    # 포트가 숫자인지 if 확인 (비어 있지 않고 숫자 형식)
    if [ -n "$port" ] && [[ "$port" =~ ^[0-9]+$ ]]; then
        ports+=("$port")

        # curl로 상태 확인
        response=$(curl -I -s -m 5 -w "%{time_total}" "http://localhost:$port" 2>/dev/null)

        if [ -n "$response" ]; then
            time_total=$(echo "$response" | tail -n1)
            echo "localhost:$port   ✓ 정상 (응답시간: ${time_total}초)"
            normal_count=$((normal_count + 1))
        else
            echo "localhost:$port   ✗ 연결 실패"
        fi
    fi
done

echo "========================"
echo "총 ${#ports[@]}개 서비스 중 $normal_count개 정상"
echo "모니터링 완료: $(date '+%Y-%m-%d %H:%M:%S')"

```

---

## **연습문제 4: 네트워크 연결 테스트 스크립트**

**문제**: telnet을 이용하여 원격 서버의 포트 연결 상태를 확인하는 스크립트를 작성하세요.

**요구사항**:

- 스크립트 실행: ./test_connection.sh google.com:80 localhost:22 192.168.1.1:443
- telnet 명령어 사용 (타임아웃 3초)
- 연결 성공/실패 상태 표시

**실행 예제**:

$ ./test_connection.sh google.com:80 localhost:22 192.168.1.1:443

네트워크 연결 테스트 시작

==========================

google.com:80 연결 테스트 중...

✓ 연결 성공

localhost:22 연결 테스트 중...

✓ 연결 성공

192.168.1.1:443 연결 테스트 중...

✗ 연결 실패 (타임아웃)

==========================

테스트 완료: 3개 중 2개 성공

```bash
#!/bin/bash

if [ $# -eq 0 ]; then
    echo "사용법: $0 <호스트:포트> ..."
    exit 1
fi

echo "네트워크 연결 테스트 시작"
echo "=========================="

#!/bin/bash

echo "네트워크 포트 사용 현황"
echo "======================"
echo "LISTEN 상태 포트 분석 중..."

# ss로 LISTEN 포트 추출 (포트 번호만, sort로 정렬)
ports=$(ss -tlnp | grep "LISTEN" | cut -d':' -f2 | cut -d' ' -f1 | sort -n | uniq)

# ports가 비어 있는지 확인
if [ -z "$ports" ]; then
    echo "LISTEN 상태 포트가 없습니다."
    exit 0
fi

# 각 포트 순회
total=0
for port in $ports; do
    # lsof로 프로세스 정보
    info=$(lsof -i :$port | head -n2 | tail -n1)
    pid=$(echo "$info" | cut -d' ' -f2)
    proc_name=$(echo "$info" | cut -d' ' -f1)
    address=$(echo "$info" | cut -d' ' -f9 | cut -d'->' -f1)

    echo "포트 $port ($proc_name)"
    echo "프로세스: $proc_name (PID: $pid)"
    echo "주소: $address"
    total=$((total + 1))
done

echo "======================"
echo "총 $total개 포트가 LISTEN 상태입니다."
success_count=0

for arg in "$@"; do
    host=${arg%%:*} 
    port=${arg#*:}  

    echo "$arg 연결 테스트 중..."

    result=$(timeout 3 telnet $host $port 2>&1)

    if echo "$result" | grep -q "Connected"; then
        echo "✓ 연결 성공"
        success_count=$((success_count + 1))
    else
        echo "✗ 연결 실패 (타임아웃)"
    fi
done

echo "=========================="
echo "테스트 완료: $#개 중 $success_count개 성공"

```

---

## **연습문제 5: 프로세스별 네트워크 포트 사용 현황 스크립트**

**문제**: 현재 네트워크 포트를 사용하는 프로세스들의 정보를 정리하여 출력하는 스크립트를 작성하세요.

**요구사항**:

- ss 명령어로 LISTEN 상태 포트 확인
- lsof 명령어로 프로세스 정보 확인
- 포트 번호별로 정렬하여 출력

**실행 예제**:

$ ./port_usage.sh

네트워크 포트 사용 현황

======================

LISTEN 상태 포트 분석 중...

포트 22 (SSH)

프로세스: sshd (PID: 1234)

주소: 0.0.0.0:22

포트 80 (HTTP)

프로세스: nginx (PID: 5678)

주소: 0.0.0.0:80

포트 3306 (MySQL)

프로세스: mysqld (PID: 9012)

주소: 127.0.0.1:3306

포트 8080 (사용자 정의)

프로세스: java (PID: 3456)

주소: 0.0.0.0:8080

======================

총 4개 포트가 LISTEN 상태입니다.

```bash
#!/bin/bash

echo "네트워크 포트 사용 현황"
echo "======================"
echo "LISTEN 상태 포트 분석 중..."

# ss로 LISTEN 포트 추출 (포트 번호만, sort로 정렬)
ports=$(ss -tlnp | grep "LISTEN" | cut -d':' -f2 | cut -d' ' -f1 | sort -n | uniq)

if [ -z "$ports" ]; then
    echo "LISTEN 상태 포트가 없습니다."
    exit 0
fi

total=0
for port in $ports; do
    info=$(lsof -i :$port | head -n2 | tail -n1)
    pid=$(echo "$info" | cut -d' ' -f2)
    proc_name=$(echo "$info" | cut -d' ' -f1)
    address=$(echo "$info" | cut -d' ' -f9 | cut -d'->' -f1)

    echo "포트 $port ($proc_name)"
    echo "프로세스: $proc_name (PID: $pid)"
    echo "주소: $address"
    total=$((total + 1))
done

echo "======================"
echo "총 $total개 포트가 LISTEN 상태입니다."

```

---

## **연습문제 6: 대량 포트 킬러 스크립트**

**문제**: 포트 범위를 지정하여 해당 범위의 모든 사용 중인 포트를 찾아 프로세스를 종료하는 스크립트를 작성하세요.

**요구사항**:

- 스크립트 실행: ./kill_port_range.sh 3000 3010
- 지정된 범위의 포트 중 사용 중인 것만 확인
- 사용자에게 확인 후 프로세스 종료

**실행 예제**:

$ ./kill_port_range.sh 3000 3010

포트 범위 3000-3010 검색 중...

사용 중인 포트 발견:

포트 3000: node (PID: 12345)

포트 3001: node (PID: 12346)

포트 3005: python (PID: 12347)

위 3개 프로세스를 종료하시겠습니까? (y/N): y

프로세스 종료 중...

포트 3000 (PID: 12345) 종료 완료

포트 3001 (PID: 12346) 종료 완료

포트 3005 (PID: 12347) 종료 완료

모든 프로세스가 종료되었습니다.

```bash
#!/bin/bash

if [ $# -ne 2 ]; then
    echo "사용법: $0 <시작포트> <종료포트>"
    exit 1
fi

start=$1
end=$2

echo "포트 범위 $start-$end 검색 중..."

ports=()
pids=()

for ((i=$start; i<=$end; i++)); do
    pid=$(lsof -t -i :$i)
    if [ -n "$pid" ]; then
        ports+=("$i")
        pids+=("$pid")
        proc=$(ps -p $pid -o comm=)
        echo "포트 $i: $proc (PID: $pid)"
    fi
done

total=${#ports[@]}

if [ $total -eq 0 ]; then
    echo "사용 중인 포트가 없습니다."
    exit 0
fi

echo "위 $total개 프로세스를 종료하시겠습니까? (y/N): "
read answer

if [ "$answer" = "y" ]; then
    echo "프로세스 종료 중..."
    for ((j=0; j<$total; j++)); do
        kill ${pids[$j]}
        echo "포트 ${ports[$j]} (PID: ${pids[$j]}) 종료 완료"
    done
    echo "모든 프로세스가 종료되었습니다."
else
    echo "종료 취소."
fi

```

---

## **연습문제 7: 서비스 자동 재시작 스크립트**

**문제**: 특정 포트의 서비스가 다운되었을 때 자동으로 재시작하는 스크립트를 작성하세요.

**요구사항**:

- 설정: 포트 번호, 재시작 명령어
- curl 또는 telnet으로 서비스 상태 확인
- 다운 시 기존 프로세스 정리 후 재시작

**실행 예제**:

$ ./auto_restart.sh

서비스 자동 재시작 모니터

========================

설정된 서비스들:

- 포트 3000: npm start

- 포트 8080: java -jar app.jar

- 포트 5000: python app.py

모니터링 시작...

[14:30:15] 포트 3000 상태 확인... ✓ 정상

[14:30:16] 포트 8080 상태 확인... ✗ 다운됨

[14:30:16] 포트 8080 서비스 재시작 중...

[14:30:16] 기존 프로세스 정리 완료

[14:30:17] 새 프로세스 시작: java -jar app.jar

[14:30:20] 포트 8080 재시작 완료 ✓

[14:30:21] 포트 5000 상태 확인... ✓ 정상

다음 확인까지 30초 대기 중...

```bash
#!/bin/bash

services=("3000:npm start" "8080:java -jar app.jar" "5000:python app.py")

echo "서비스 자동 재시작 모니터"
echo "========================"
echo "설정된 서비스들:"
for svc in "${services[@]}"; do
    port=${svc%%:*} 
    cmd=${svc#*:}  
    echo "포트 $port: $cmd"
done
echo "모니터링 시작..."

while true; do
    time_now=$(date '+%H:%M:%S')

    for svc in "${services[@]}"; do
        port=${svc%%:*} 
        cmd=${svc#*:}  

        echo "[$time_now] 포트 $port 상태 확인..."

        if curl -I -s -m 5 "http://localhost:$port" >/dev/null; then
            echo "[$time_now] 포트 $port 상태 확인... ✓ 정상"
        else
            echo "[$time_now] 포트 $port 상태 확인... ✗ 다운됨"
            echo "[$time_now] 포트 $port 서비스 재시작 중..."

            # 기존 PID 찾기 & kill
            pid=$(lsof -t -i :$port)
            if [ -n "$pid" ]; then
                kill $pid
                echo "[$time_now] 기존 프로세스 정리 완료"
            fi

            # 재시작 (백그라운드)
            $cmd &
            echo "[$time_now] 새 프로세스 시작: $cmd"
            echo "[$time_now] 포트 $port 재시작 완료 ✓"
        fi
    done

    echo "다음 확인까지 30초 대기 중..."
    sleep 30
done

```

---

## **연습문제 8: 포트 스캐너 스크립트**

**문제**: 특정 IP 주소의 포트 범위를 스캔하여 열린 포트를 찾는 스크립트를 작성하세요.

**요구사항**:

- 스크립트 실행: ./port_scan.sh 192.168.1.100 20 30
- telnet 명령어 사용하여 포트 연결 테스트
- 타임아웃 2초로 설정
- 열린 포트만 출력

**실행 예제**:

$ ./port_scan.sh 192.168.1.100 20 30

192.168.1.100 포트 스캔 (범위: 20-30)

=====================================

스캔 진행 중... [###########] 100%

열린 포트 발견:

포트 22: SSH (OpenSSH)

포트 80: HTTP

포트 443: HTTPS

=====================================

총 11개 포트 중 3개가 열려있습니다.

스캔 완료 시간: 22초

```bash
#!/bin/bash

if [ $# -ne 3 ]; then
    echo "사용법: $0 <IP> <시작포트> <종료포트>"
    exit 1
fi

ip=$1
start=$2
end=$3

total=$((end - start + 1))
open_count=0
open_ports=()

echo "$ip 포트 스캔 (범위: $start-$end)"
echo "====================================="
echo "스캔 진행 중... "

# for문으로 범위 스캔
for ((i=$start; i<=$end; i++)); do
    # telnet 테스트 (timeout 2초)
    result=$(timeout 2 telnet $ip $i 2>&1)

    if echo "$result" | grep -q "Connected"; then
        service=$(grep "^$i/" /etc/services | cut -f1 | head -n1)  # /etc/services에서 서비스명
        if [ -z "$service" ]; then
            service="알 수 없음"
        fi
        open_ports+=("포트 $i: $service")
        open_count=$((open_count + 1))
    fi

    # 진행률 계산 (총 / 현재)
    progress=$(((i - start + 1) * 100 / total))
    echo -n "[$(printf '#%.0s' {1..$((progress/10))})] $progress% "  # 간단 진행 바
    echo
done

echo "열린 포트 발견:"
for port in "${open_ports[@]}"; do
    echo $port
done
echo "====================================="
echo "총 $total개 포트 중 $open_count개가 열려있습니다."
echo "스캔 완료 시간: $(( (end - start + 1) * 2 ))초 (추정)"

```

---

## **추가 도전 과제**

### **도전과제 1: 네트워크 트래픽 모니터링**

- ss 명령어로 연결 상태별 통계 출력
- TCP/UDP 연결 수 집계

### **도전과제 2: 포트 사용 히스토리**

- 주기적으로 포트 사용 현황을 로그 파일에 기록
- 시간대별 포트 사용 패턴 분석

### **도전과제 3: 멀티 서버 상태 확인**

- 여러 서버의 동일 포트 상태를 동시에 확인
- 병렬 처리로 성능 향상

---

## **참고사항**

**유용한 명령어 조합**:

# 포트 사용 확인

ss -tlnp | grep :포트번호

# 프로세스 정보 확인

lsof -i :포트번호

# 연결 테스트

curl -I --connect-timeout 5 http://IP:포트

timeout 3 telnet IP 포트

# 프로세스 강제 종료

kill -9 PID

이 연습문제들을 통해 실무에서 자주 사용하는 네트워크 관련 스크립트 작성 능력을 기를 수 있습니다!