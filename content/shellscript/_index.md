+++
title = "자주쓰는 쉘 스크립트 모음"
pre ="<i class='fa fa-linux' ></i> "
weight = 3
+++

# if문
## if-then
가장 기본적인 if-else 구문의 형식은 다음과 같다.
```
if command
then
    commands
fi
```
```
if command; then
    commands
fi
```
bash 쉘은 if문 줄에 정의된 명령을 실행한다. 이 명령의 종료 상태가 0(명령이 성공적으로 완료됨)이라면 then 아래에 있는 명령이 실행된다. 명령의 종료 상태가 0이 아니라면 then 아래에 있는 명령은 실행되지 않고, bash 쉘은 스크립트의 다른 명령으로 넘어간다.

## if-then-else
```
if command
then
    commands
else
    commands
fi
```
if문 줄의 명령이 0이 아닌 종료 상태 코드를 돌려주면 bash 쉘은 else 부분의 명령을 실행한다.

## 중첩된 if문
```
if command1
then
    commands
elif command2
then
    more commands
fi
```
elif는 if-then 구문의 else 부분을 이어 나간다. elif 명령의 종료 상태 코드가 0이라면 bash는 두 번째 then문 부부너의 명령들(more commands)을 실행한다.

## 테스트 명령 써보기
```
if test condition
then
    명령
fi
```
테스트 명령에 나와있는 조건이 참으로 평가되면 테스트 명령은 종료 상태 코드를 0으로 돌려준다.

bash 쉘은 if-then 구문에서 테스트 명령을 쓰지 않고도 조건을 테스트하는 다른 방법을 제공한다.
```
if [ condition ]
then
    명령
fi
```
대괄호는 테스트 조건을 정의한다. 한 가지 주의할 점이 있다. 여는 대괄호 뒤와 닫는 대괄호 앞에 각각 빈 칸이 있어야 한다. 안그러면 오류 메세지가 나타난다. 테스트 명령 및 테스트 조건은 세 가지 종류의 조건을 평가할 수 있다(파일 비교, 숫자 비교, 문자열 비교).

### 파일 비교
쉘 스크립트에서 가장 강력하고 가장 많이 사용되는 비교다. 리눅스 파일 시스템에서 파일과 디렉토리의 상태를 테스트할 수 있다.

| 비교  | 설명  |
|---|---|
| -d file  | 파일이 존재하고 디렉토리인지 검사한다  |
| -e file  | 파일이 존재하는지 검사한다  |
| -f file  | 파일이 존재하고 파일인지 검사한다  |
| -L file  | 파일이 심볼릭 링크이면 참  |
| -r file  | 파일이 존재하고 읽을 수 있는지 검사한다  |
| -s file  | 파일이 존재하고 비어 있지 않은지 검사한다  |
| -w file  | 파일이 존재하고 기록할 수 있는지 검사한다  |
| -x file  | 파일이 존재하고 실행할 수 있는지 검사한다  |
| -O file  | 파일이 존재하고 현재 사용자가 소유한 것인지 검사한다  |
| -G file  | 파일이 존재하고 기본 그룹이 현재 사용자와 같은지 검사한다  |
| file1 -nt file2  | file1이 file2보다 새것인지 검사한다  |
| file1 -ot file2  | file1이 file2보다 오래된 것인지 검사한다  |

**디렉토리 확인하기**

지정된 디렉토리가 시스템에 존재하는지 보려면 -d 검사를 한다. 보통은 디렉토리에 파일을 쓰거나 디렉토리의 위치를 변경하기 전에 사용하면 좋다.
```
jump_directory=/home/arthur

if [ -d $jump_directory ]
then
    echo "hi"
fi
```

**개체가 존재하는지 여부 검사하기**
```
location=$HOME
file_name="bong"

if [ -e $location ]
then # Directory does exist
    if [ -e $location/$file_name ]
    then
        echo "file does exist"
    else
        echo "file doesn't exist"
    fi
else
    echo "Directory doesn't exist"
fi
```
스크립트에서 파일 또는 디렉토리를 사용하기 전에 이 개채가 있는지 확인하려면 -e 비교를 사용한다(-e 비교는 파일과 디렉토리 양쪽 모두에 적용된다).

### 숫자 비교
| 비교  | 설명  |
|---|---|
| n1 -eq n2  | n1과 n2가 같은지 검사한다  |
| n1 -ge n2  | n1이 n2보다 크거나 같은지 검사한다  |
| n1 -gt n2  | n1이 n2보다 큰지 검사한다  |
| n1 -le n2  | n1이 n2보다 작거나 같은지 검사한다  |
| n1 -lt n2  | n1이 n2보다 작은지 검사한다  |
| n1 -ne n2  | n1과 n2가 같지 않은지 검사한다  |

```
if [ $val1 -gt 1 ]
then
   echo "hi"
fi
```
### 문자열 비교
| 비교  | 설명  |
|---|---|
| str1 = str2  | str1이 str2와 같은지 검사한다  |
| str1 != str2  | str1이 str2와 같지 않은지 검사한다  |
| str1 < str2  | str1이 str2보다 작은지 검사한다  |
| str1 > str2  | str1이 str2보다 큰지 검사한다  |
| -n str1  | str1의 길이가 0보다 큰지(0이 아닌지) 검사한다  |
| -z str1  | str1의 길이가 0인지 검사한다  |

문자열이 같은가 같지 않은가 비교할 때는 모든 문장부호와 대문자도 고려된다는 점을 잊지 말자.
```
testuser=bong

if [ $USER = $testuser ]
then
    echo "hi"
fi
```

어떤 문자열이 다른 문자열보다 큰가 작은가를 판단할 때부터 일이 복잡해진다. 문자열이 큰지의 여부를 테스트하는 기능을 사용하려고 할 때 두가지 문제가 있다.

첫째, 부등호 기호를 이스케이프 해야 하는 것. 그렇지 않으면 쉘은 이를 리다이렉트 기호로 해석해서 문자열 값을 파일 이름으로 사용한다.
```
val1=baseball
val2=hockey

if [ $val1 \> $val2 ]
then
    echo "hi"
fi
```
둘째, 어느 것이 더 큰지 순서를 결정하는 논리는 sort 명령에서 쓰이는 것과 같지 않다.

비교 테스트에서는 표준 ASCII 순서를 사용하며, 정렬 순서를 결정하기 위하여 각 문자의 ASCII 숫자 코드값을 이용한다. sort 명령은 시스템 로케일의 언어 설정에 정의된 정렬 순서를 사용한다. 영어라면 로케일 설정은 소문자를 대문자보다 앞서서 정렬하도록 지정한다.

비어있고 초기화되지 않은 변수는 쉘 스크립트 테스트에 치명적인 영향을 미칠 수 있다. 변수의 내용이 확실하지 않으면 숫자 또는 문자열 비교를 사용하기 전에 -n 또는 -z를 사용하여 값을 포함하는지 테스트하는 것이 가장 좋다.
```
val1=testing

if [ -n $val1 ]
then
    echo "hi"
fi

if [ -z $val2 ]
then
    echo "hi"
fi
```

# for문
## 일반 for문
```
FILE="/Users/bong"

for state in $(ls $FILE)
do
  echo $state
done
```

```
for i in ~/bong*.sh ; do
    if [ -r "$i" ]; then
        . $i
    fi
done
```
for문 돌리면서 스크립트 파일 실행

## C 스타일 for문
```
TEST_TRIES=7
TEST_INTERVAL_SECONDS=5

  for (( TRY_COUNT = 1; TRY_COUNT <= $TEST_TRIES; TRY_COUNT ++ ))
  do
    sleep $TEST_INTERVAL_SECONDS
    echo "Checking HTTP port. ("$TRY_COUNT"/$TEST_TRIES)"

    HTTP_STATUS_CODE=`curl -sL -o /dev/null -I -w "%{http_code}" $TEST_URL --max-time 10`
    if [ $HTTP_STATUS_CODE -gt 199 ] && [ $HTTP_STATUS_CODE -lt 300 ]; then
      echo "The Spring Boot process has started successfully."
      break
    fi
    if [ $TRY_COUNT = $TEST_TRIES ]; then
      echo "ERROR : The Spring Boot process failed to start."
      exit 1
    fi
  done
```

# while문
## 기본 while 형식
```
var1=10
while [ $var1 -gt 0 ]
do
  echo $var1
  var1=$[ $var1 - 1 ]
done
```

## getopts 명령어 사용
```
# -a 옵션이 있는지 플래그 변수 a_flag와
# -p 옵션의 구분자를 정의하기
a_flag=0
separator=""

while getopts "ap:" option
do
  case $option in
    a)
      a_flag=1
      ;;
    p)
      separator="$OPTARG"
      ;;
    \?)
      echo "Usage: getopts.sh [-a] [-p separator] target_dir" 1>&2
      exit 1
      ;;
    esac
done
```

# case 명령
```
case variable in
pattern1 | pattern2) commands1;;
pattern3) commands2;;
*) default commands;;
esac
```

예제
```
echo_host(){
  host=`hostname`
  echo "[$host] $1"
}

#### Main #####

OPTION=$1

case $OPTION in
"start") start_nginx;;
"stop") stop_nginx;;
"restart") restart_nginx;;
*) echo_host "option : $0 start|stop|restart";;
esac
```

결과
```
$ ./bong.sh ttt
[MyMacBook] option : ./bong.sh start|stop|restart
```

# 스크립트 종료하기
## 리눅스 종료 상태코드
| 코드  | 설명  |
|---|---|
| 0  | 명령이 성공적으로 완료됨  |
| 1  | 일반 알 수 없는 오류  |
| 2  | 쉘 명령을 잘못 사용함  |
| 126  | 명령을 실행할 수 없음(Permission denied)  |
| 127  | 명령을 찾을 수 없음  |
| 128  | 잘못된 종료 매개변수  |
| 128+x  | 치명적인 오류로 리눅스 신호 x를 포함  |
| 130  | Ctrl+C로 명령이 종료됨  |
| 255  | 범위를 벗어난 종료 상태  |

## 종료 코드 확인하는 명령어
```
$ echo $?
```

cf) 종료 상태 코드는 0 ~ 255까지 쓸 수 있다.

## exit 명령
쉘 스크립트는 마지막 명령의 종료 상태로 끝마친다. 사용자 정의 종료 상태 코드를 돌려주도록 이를 변경할 수 있다. exit 명령은 스크립트가 종료될 때 종료 상태를 지정할 수 있다.
```
#!/bin/bash
var1=10
var2=30
var3=$[$var1 + $var2]
echo The answer is $var3
exit5
#exit $var3 처럼 exit 명령의 매개변수에 변수를 사용할 수도 있다.
```
결과
```
$ chmod u+x test13
$ ./test13
The answer is 40
$ echo $?
5
```

# 자주쓰는 명령어 모음

쉘 스크립트 작성후 실행하기 전에 문법을 확인하는 -n 옵션
```
$ sh -n script.sh
```

java 프로세스 PID 리스트업 해서 출력
```
$ ps -ef | grep java | awk '{print $2}'
```

grep 하면 방금 실행한 것도 잡히기 때문에 그거 제외해주는 명령어
```
$ ps -ef | grep java | grep -v grep
```

pid가 여러개일 때 check하는 if문 추가적으로 `ps -ef | grep nginx | grep -v grep | wc -l` 방법도 있다
```
check_running() {
    PID=`ps -ef | grep nginx | grep -v grep | awk '{print $2}'`
    if [[ -n $PID ]] ; then
        echo "nginx process already started (PID:$PID)"
        exit 1
    fi
}
```

`/dev/null`은 어떤 데이터를 보내든 블랙홀로써 전부 버려질것이다.
```
$ >/dev/null 2>&1
```

`2` 표준 에러를 뜻하는 파일 디스크립터다. <br>
`&` 파일 디스크립터를 뜻하는 심볼이다. 이 기호가 없으면 다음 1은 파일이름으로 간주된다. <br>
`1` 표준 출력을 뜻하는 파일 디스크립터다. <br>
결론 : 프로그램의 출력을 /dev/null로 보내고 출력을 하는데 표준에러를 표준출력으로 보내라 <br>

다른 프로세스가 사용하고 있는 포트가 아닌지 확인하기 위한 명령어
```
$ netstat -an | grep 8080
```

control + z하면 백그라운드로 suspend되는데(다시 포그라운드로 부르는건 fg)

예를들어 [5]  + 17215 suspended  node server.js
```
$ kill %5
```
하면 죽일 수 있다.

실수로 git add 했을 때 되돌리기
```
$ git rm —cached <filename>
```

os 버전 확인
```
OS_VERSION=$(sed 's/.*release \([0-9]\).*/\1/' /etc/redhat-release)
```

네트워크 인터페이스명은 빼고 IP주소만 쓰고 싶을 때 사용한는 명령어
```
$ LANG=C /sbin/ifconfig | awk '/inet / {split($2,arr,":"); print arr[2]}'
```

현재 디렉토리에 있는 서브 디렉토리들의 디스크 사용량 조사
```
$ du -h ./
```

디스크 용량 검사
```
$ df -h
```

네트워크 상태 모니터링 (ESTABLISHED, CLOSE_WAIT, TIME_WAIT 등)
```
$ netstat -ton
```


