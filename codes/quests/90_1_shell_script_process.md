## **✅ 문제 : 간단한 서버 관리 스크립트 작성**

### **🔧 요구사항**

- start: 포트 8000에서 http.server를 백그라운드로 실행 (nohup, 로그는 server.log)
- stop: 실행 중인 프로세스를 찾아 종료
- status: 프로세스가 실행 중인지 확인하여 출력
- restart: 중지 후 다시 실행

### **🎯 실행 예시**

$ ./webserver.sh start

서버가 백그라운드에서 시작되었습니다.

$ ./webserver.sh status

서버 실행 중입니다. PID: 13579

$ ./webserver.sh stop

서버가 종료되었습니다.

$ ./[webserver.sh](http://webserver.sh/) tail_log

… log message 확인

문제 모두 조건에 따라:

- if [ "$1" == "start" ] 식으로 흐름 제어
- 변수 PORT, PID, LOGFILE 등을 정의해 구성 가능

```
# start
[hwang@192.168.0.44 ~/Downloads]$ source ./webserver.sh start
 Server is running in background
nohup: ignoring input and redirecting stderr to stdout

# status
[hwang@192.168.0.44 ~/Downloads]$ source ./webserver.sh status
 Sever is running PID: 5342
[2]+  Exit 1                  nohup python3 -m http.server 8000 --bind 0.0.0.0 > server.log
[hwang@192.168.0.44 ~/Downloads]$ source ./webserver.sh start
 Server is running in background
nohup: ignoring input and redirecting stderr to stdout

# stop
[hwang@192.168.0.44 ~/Downloads]$ source ./webserver.sh stop
 Server Down
[1]-  Killed                  nohup python3 -m http.server 8000 --bind 0.0.0.0 > server.log
[2]+  Exit 1                  nohup python3 -m http.server 8000 --bind 0.0.0.0 > server.log

# tail_log
[hwang@192.168.0.44 ~/Downloads]$ source ./webserver.sh tail_log
Log Message:
Traceback (most recent call last):
  File "/usr/lib64/python3.9/runpy.py", line 197, in _run_module_as_main
    return _run_code(code, main_globals, None,
  File "/usr/lib64/python3.9/runpy.py", line 87, in _run_code
    exec(code, run_globals)
  File "/usr/lib64/python3.9/http/server.py", line 1308, in <module>
    test(
  File "/usr/lib64/python3.9/http/server.py", line 1259, in test
    with ServerClass(addr, HandlerClass) as httpd:
  File "/usr/lib64/python3.9/socketserver.py", line 452, in __init__
    self.server_bind()
  File "/usr/lib64/python3.9/http/server.py", line 1302, in server_bind
    return super().server_bind()
  File "/usr/lib64/python3.9/http/server.py", line 137, in server_bind
    socketserver.TCPServer.server_bind(self)
  File "/usr/lib64/python3.9/socketserver.py", line 466, in server_bind
    self.socket.bind(self.server_address)
OSError: [Errno 98] Address already in use

# webserver.sh
[hwang@192.168.0.44 ~/Downloads]$ cat webserver.sh
#!/bin/bash

PID=" "

if [ "$1" == "start" ]; then
        nohup python3 -m http.server 8000 --bind 0.0.0.0 > server.log &
        echo " Server is running in background"
elif [ "$1" == "status" ]; then
        PID=$(pgrep -f "python3 -m http.server")
        echo " Sever is running PID: $PID"
elif [ "$1" == "stop" ]; then
        PID=$(pgrep -f "python3 -m http.server")
        kill -9 $PID
        echo " Server Down"
elif [ "$1" == "restart" ]; then
        PID=$(pgrep -f "python3 -m http.server")
        kill -9 $PID
        nohup python3 -m http.server 8000 --bind 0.0.0.0 > server.log &
        echo "Server restarted"
elif [ "$1" == "tail_log" ]; then
        echo "Log Message: " && cat server.log
else
        echo "400:bad request"
fi

```

주의 사항

- if 문에서 문자열 비교에 [ “a” == “b” ] 처럼 대괄호와 = 사이에 문자열에는 공백이 있어야한다.

```
[hwang@192.168.0.44 ~/Downloads]$ ps -aux | grep "http.server"
hwang       5279  0.0  0.1 221800  2176 pts/2    S+   14:43   0:00 grep --color=auto http.server
[hwang@192.168.0.44 ~/Downloads]$ kill -9 5279
-bash: kill: (5279) - No such process
[hwang@192.168.0.44 ~/Downloads]$ ps -aux | grep "http.server"
hwang       5287  0.0  0.1 221800  2176 pts/2    S+   14:43   0:00 grep --color=auto http.server
```

- 위에 코드는 실행중인 프로세스의 PID를 찾기 위해 실행한거지만 찾을 때마다 PID가 달라진다. 왜냐하면 **웹서버 프로세스가 아니라, `grep` 명령어 자체의 프로세스 ID(PID)를 보고 있기 때문이다.**
- grep 명령어 역시 하나의 실행중인 프로세스이기 때문이다. 그래서 우리가 찾는 프로세스를 찾으려면 아래 명령어를 사용하는 것이 좋다.

```
pgrep -f "python3 -m http.server"
```

- 이 명령어는 **`grep`** 프로세스는 무시하고, 정확히 "python3 -m http.server"로 실행된 프로세스의 PID만 깔끔하게 출력해 줍니다.