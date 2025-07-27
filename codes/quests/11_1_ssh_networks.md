# **Network Shell Script 실습 문제**

## **문제 환경 설정**

실습을 위해 다음 파일들을 생성하세요:

# 1. 네트워크 로그 파일 생성

cat > network.log << 'EOF'

2024-01-15 10:30:25 192.168.1.100 CONNECT success

2024-01-15 10:30:30 192.168.1.101 CONNECT failed

2024-01-15 10:31:15 192.168.1.102 CONNECT success

2024-01-15 10:31:20 192.168.1.100 DISCONNECT success

2024-01-15 10:32:10 192.168.1.103 CONNECT success

2024-01-15 10:32:15 192.168.1.101 CONNECT success

2024-01-15 10:33:25 192.168.1.104 CONNECT failed

2024-01-15 10:33:30 192.168.1.102 DISCONNECT success

EOF

# 3. 접속 통계 파일 생성

cat > connections.txt << 'EOF'

192.168.1.100 5

192.168.1.101 12

192.168.1.102 8vi

192.168.1.103 3

192.168.1.104 15

192.168.1.105 7

EOF

---

## **문제 1: 네트워크 연결 상태 분석기**

**요구사항:**

- network.log 파일을 분석하여 연결 성공/실패 통계를 출력하는 스크립트 작성
- 전체 연결 시도 수, 성공 수, 실패 수를 계산
- 성공률을 백분율로 표시 (소수점 제거)

**출력 형태:**

=== 네트워크 연결 분석 결과 ===

전체 연결 시도: X건

성공: Y건

실패: Z건

성공률: W%

**제한사항:**

- if문과 변수만 사용
- grep, wc, cut 명령어 활용
- 파일명은 스크립트 실행 시 첫 번째 인자로 받기

```
[yhc@192.168.0.51 ~/Download]$ source network_log.sh
=== Network Connection Analysis Results ===
Full connection attempt: 8 cases
Success: 6 case
Failed: 2 case
Success rate: 75%
[yhc@192.168.0.51 ~/Download]$ cat network_log.sh
#!/bin/bash

all_cnt=0
success_cnt=0
fail_cnt=0

total_lines=$(cat network.log | wc -l)
all_cnt=$(((total_lines / 2 )))

success_cnt=$(grep "success" network.log | wc -l)
fail_cnt=$(grep "fail" network.log | wc -l)

if [ $all_cnt -eq 0 ]; then
    success_rate=0
    echo "Warning: 총 시도 횟수가 0입니다. 성공률을 계산할 수 없습니다."
else
    success_rate=$(( (success_cnt * 100) / all_cnt ))
fi

echo "=== Network Connection Analysis Results ==="

echo "Full connection attempt: $all_cnt cases"

echo "Success: $success_cnt case"

echo "Failed: $fail_cnt case"

echo "Success rate: $success_rate%"

```

---

## **문제 2: IP 주소별 접속 빈도 상위 리스트**

**요구사항:**

- network.log에서 IP 주소별 접속 횟수를 계산
- 접속 횟수 기준으로 내림차순 정렬하여 상위 3개만 출력
- 각 IP의 첫 접속 시간도 함께 표시

**출력 형태:**

=== 접속 빈도 TOP 3 ===

1위: 192.168.1.XXX (X회) - 첫 접속: 10:XX:XX

2위: 192.168.1.XXX (X회) - 첫 접속: 10:XX:XX

3위: 192.168.1.XXX (X회) - 첫 접속: 10:XX:XX

**제한사항:**

- if문과 변수만 사용
- cut, sort, uniq, grep 명령어 활용
- head나 tail로 결과 제한

```

[yhc@192.168.0.51 ~/Download]$ source get_frequency.sh
=== 접속 빈도 TOP 3 ===
1위: 192.168.1.102 (2회) - 첫 접속: 10:31:15
2위: 192.168.1.101 (2회) - 첫 접속: 10:30:30
3위: 192.168.1.100 (2회) - 첫 접속: 10:30:25
[yhc@192.168.0.51 ~/Download]$ cat get_frequency.sh
#!/bin/bash

#!/bin/bash

ip_counts=$(grep -v "^$" network.log | cut -d' ' -f3 | sort | uniq -c | sort -nr| sed 's/^ *//')

top1=$(echo "$ip_counts" | head -n1)
top2=$(echo "$ip_counts" | head -n2 | tail -n1)
top3=$(echo "$ip_counts" | head -n3 | tail -n1)

top1_count=$(echo "$top1" | cut -d' ' -f1)
top1_ip=$(echo "$top1" | cut -d' ' -f2)

top2_count=$(echo "$top2" | cut -d' ' -f1)
top2_ip=$(echo "$top2" | cut -d' ' -f2)

top3_count=$(echo "$top3" | cut -d' ' -f1)
top3_ip=$(echo "$top3" | cut -d' ' -f2)

top1_time=$(grep "$top1_ip" network.log | head -n1 | cut -d' ' -f2)
top2_time=$(grep "$top2_ip" network.log | head -n1 | cut -d' ' -f2)
top3_time=$(grep "$top3_ip" network.log | head -n1 | cut -d' ' -f2)

echo "=== 접속 빈도 TOP 3 ==="
echo "1위: $top1_ip ($top1_count회) - 첫 접속: $top1_time"
echo "2위: $top2_ip ($top2_count회) - 첫 접속: $top2_time"
echo "3위: $top3_ip ($top3_count회) - 첫 접속: $top3_time"

```

---

## **문제 3: 서버 상태 점검 스크립트**

**요구사항:**

- servers.sh 실행해 각 서버에 대해 ping 테스트 실행
- 응답 있는 서버와 없는 서버를 구분하여 출력
- 응답 시간이 100ms 이상인 서버는 "느림" 표시

**입력 형태:**

**~$ [servers.sh](http://servers.sh/) 123.92.0.12**

**출력 형태:**

=== 서버 상태 점검 결과 ===

[정상] web01 (**123.92.0.12**) - 응답시간: XXms

OR

=== 서버 상태 점검 결과 ===

[오프라인] db01 (**123.92.0.11**) - 응답없음

...

**제한사항:**

- if문과 변수만 사용
- cut, ping 명령어 활용
- ping은 1회만 실행 (ping -c 1)

```

[yhc@192.168.0.51 ~/Download]$ cat server_status_check.sh
#!/bin/bash

ping_ip=$1
server_status=$(ping -c 1 $ping_ip)
response_time=$(echo $server_status | grep "time=" | tr -d" " -f7)

if [echo "server_status" | grep -q "1 received"]; then
        [정상] web01 ($ping_ip) - 응답시간: $response_time
else
         [dhvmfkdls] web01 ($ping_ip) - 응답없음

```

---

## **문제 4: 네트워크 트래픽 임계값 모니터링**

**요구사항:**

- connections.txt에서 접속 수가 10 이상인 IP를 "높음", 5-9는 "보통", 4 이하는 "낮음"으로 분류
- 각 분류별로 개수 계산하여 출력
- "높음" 분류의 IP들만 별도로 나열

**출력 형태:**

=== 트래픽 분석 결과 ===

높음(10회 이상): X개

보통(5-9회): Y개

낮음(4회 이하): Z개

[주의 필요 IP 목록]

192.168.1.XXX (XX회)

192.168.1.XXX (XX회)

**제한사항:**

- if문과 변수만 사용
- cut, sort 명령어 활용
- 숫자 비교를 위한 조건문 사용

```bash
#!/bin/bash

# 데이터 추출: IP와 count를 공백으로 구분, sort로 내림차순 (숫자 -nr, 두 번째 필드 -k2)
data=$(sort -k2 -nr connections.txt)

# 변수 초기화
high_count=0
medium_count=0
low_count=0
high_ips=""

# 각 줄 처리 (제한사항으로 루프 대신 head/tail로 하나씩 추출)
line1=$(echo "$data" | head -n1)
line2=$(echo "$data" | head -n2 | tail -n1)
line3=$(echo "$data" | head -n3 | tail -n1)
line4=$(echo "$data" | head -n4 | tail -n1)
line5=$(echo "$data" | head -n5 | tail -n1)
line6=$(echo "$data" | head -n6 | tail -n1)

# 각 줄 분류 함수 (재사용 위해, but 제한 준수로 if 중첩)
classify() {
    local line=$1
    if [ -n "$line" ]; then
        local ip=$(echo "$line" | cut -d' ' -f1)
        local count=$(echo "$line" | cut -d' ' -f2)
        
        if [ "$count" -ge 10 ]; then
            high_count=$((high_count + 1))
            high_ips="$high_ips\n[$ip](http://$ip) ($count회)"
        elif [ "$count" -ge 5 ] && [ "$count" -le 9 ]; then
            medium_count=$((medium_count + 1))
        else
            low_count=$((low_count + 1))
        fi
    fi
}

# 분류 실행
classify "$line1"
classify "$line2"
classify "$line3"
classify "$line4"
classify "$line5"
classify "$line6"

# 출력
echo "=== 트래픽 분석 결과 ==="
echo "높음(10회 이상): $high_count개"
echo "보통(5-9회): $medium_count개"
echo "낮음(4회 이하): $low_count개"
echo "[주의 필요 IP 목록]"
echo -e "$high_ips"

```

---

## **문제 5: 현재 시스템 네트워크 정보 수집기**

**요구사항:**

- 현재 시스템의 IP 주소, 기본 게이트웨이, 활성 인터페이스 개수를 출력
- 인터넷 연결 상태 확인 (8.8.8.8로 ping 테스트)
- 모든 정보를 보기 좋게 정리하여 출력

**출력 형태:**

=== 시스템 네트워크 정보 ===

내부 IP: 192.168.1.XXX

기본 게이트웨이: 192.168.1.X

활성 인터페이스: X개

인터넷 연결: 정상/차단

**제한사항:**

- if문과 변수만 사용
- ip, hostname, ping, grep, wc 명령어 활용
- 각 정보를 변수에 저장 후 출력

```bash
#!/bin/bash

# 내부 IP (ip addr에서 enp0s3 같은 인터페이스 IP 추출, 첫 번째 활성 IP)
internal_ip=$(ip addr show | grep "inet " | grep -v "127.0.0.1" | cut -d' ' -f6 | cut -d'/' -f1 | head -n1)

# 기본 게이트웨이 (ip route에서 default via 추출)
gateway=$(ip route | grep default | cut -d' ' -f3)

# 활성 인터페이스 개수 (ip link에서 UP 상태 wc)
active_interfaces=$(ip link show | grep "state UP" | wc -l)

# 인터넷 연결 확인 (ping -c1 8.8.8.8)
ping_result=$(ping -c 1 8.8.8.8 2>/dev/null)
if echo "$ping_result" | grep -q "1 received"; then
    internet_status="정상"
else
    internet_status="차단"
fi

# 출력
echo "=== 시스템 네트워크 정보 ==="
echo "내부 IP: [$internal_ip](http://$internal_ip)"
echo "기본 게이트웨이: $gateway"
echo "활성 인터페이스: $active_interfaces개"
echo "인터넷 연결: $internet_status"

```

---

## **실행 방법**

각 문제의 스크립트를 작성한 후 다음과 같이 실행:

# 실행 권한 부여

chmod +x script_name.sh

# 스크립트 실행

./script_name.sh [인자]

## **주의사항**

- 모든 스크립트는 #!/bin/bash로 시작
- 변수 선언 시 공백 없이 작성: var=value
- if문 조건 확인 시 사용
- 명령어 결과를 변수에 저장할 때 $(command) 사용
- 파일 존재 여부 확인: [ -f filename ]