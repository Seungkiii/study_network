### **🧪 문제 1: 특정 IP 차단 상태 확인 후 차단 설정**

### **✅ 실행 예시**

$ sudo ./problem1.sh

[INFO] 현재 rich rule 목록에 192.168.0.100 차단 룰이 존재하지 않습니다.

[INFO] 차단 룰을 추가합니다...

success

또는

$ sudo ./problem1.sh

[INFO] 192.168.0.100은 이미 차단되어 있습니다.

[SKIP] 추가 작업을 수행하지 않습니다.

```

[hwang@192.168.0.44 ~/Downloads]$ source ./problem.sh
192.168.0.52 is not rejected
The ip is added rule
success
[hwang@192.168.0.44 ~/Downloads]$ cat problem.sh
#!/bin/bash

reject_ip=$(sudo firewall-cmd --list-rich-rules | cut -d'"' -f4)

if [ "$reject_ip" == "192.168.0.52" ]; then
        echo "Already rejected"
else
        echo "192.168.0.52 is not rejected"
        echo "The ip is added rule"
        sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="12.168.0.52" reject'
fi

```

---

### **🔒 문제 2: 포트 8080이 열려 있다면 닫기**

### **✅ 실행 예시**

$ sudo ./problem2.sh

[INFO] 포트 8080/tcp 이 열려 있습니다. 제거합니다...

success

또는

$ sudo ./problem2.sh

[INFO] 포트 8080/tcp 이 열려 있지 않습니다. 아무 작업도 수행하지 않습니다.

```

[hwang@192.168.0.44 ~/Downloads]$ source ./problem2.sh
 Port 8080/tcp is opened. port will be rejected
Warning: NOT_ENABLED: 8080:tcp
success
[hwang@192.168.0.44 ~/Downloads]$ vi problem2.sh
[hwang@192.168.0.44 ~/Downloads]$ source ./problem2.sh
Port 8080/tcp is not opened. No use
[hwang@192.168.0.44 ~/Downloads]$ cat problem2.sh
#!/bin/bash

open_status=$(ss -tunlp | grep :8080)

if [ -n "$open_status" ]; then
        echo " Port 8080/tcp is opened. port will be rejected "
        sudo firewall-cmd --permanent --remove-port=8080/tcp
else
        echo "Port 8080/tcp is not opened. No use"
fi

```

---

### **🧩 문제 3: SSH 서비스 제거 후 특정 IP만 허용**

### **✅ 실행 예시**

$ sudo ./problem3.sh

[INFO] SSH 서비스가 열려 있습니다. 제거합니다...

success

[INFO] 192.168.0.10 IP에만 포트 22 허용 규칙을 추가합니다...

success

또는

$ sudo ./problem3.sh

[INFO] SSH 서비스가 이미 제거되어 있습니다.

[INFO] 포트 22 허용 규칙만 추가합니다...

success

```
[hwang@192.168.0.44 ~/Downloads]$ cat problem3.sh
#!/bin/bash

ssh_open=$(sudo firewall-cmd --list-ports | grep "22/tcp")

if [ -n "$ssh_open" ]; then
        echo " SSH Service is opened. port 22 will be added "
        sudo firewall-cmd --remove-port=22/tcp
        sudo firewall-cmd --reload
else
        echo " SSH is already remvoed "

fi

echo "[INFO] 192.168.0.10 IP에만 포트 22 허용 규칙을 추가합니다..."
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.0.10" port port="22" protocol="tcp" accept'
sudo firewall-cmd --reload
echo "success"

```

---