# Shell Script/ if

Bourne-like shell들의 `if` 구문은 아래와 같은 형태를 띈다.

``` bash
if
  command-list1
then
  command-list2
else
  command-list3
fi
```

만약 `command-list1` 목록의 커맨드들의 `exit code`가 0이라면 `then` 구절이 실행된다. 반대로 `exit-code`가 0이 아니라면 `else` 구절이 실행된다. 이때 조건에 해당하는 `command-list1`은 다양한 형태로 작성될 수 있다. 이 문서는 `command-list1`의 형태별 의미와 그 차이를 정확히 알고자 작성하였다.

## if \[ ... \]

\[는 `test` 커맨드의 별칭이다. `test` 커맨드는 주어진 인자를 바탕으로 `exit code`를 설정한다. 아래의 예시 코드를 실행해보자.
   
``` bash
test -d /bin  # /bin 경로가 디렉토리인지 검사한다.
echo $?  # $?는 가장 최근에 실행된 프로세스 exit code를 의미한다.

: 'Output
0
'

test -f /bin  # /bin 경로가 파일인지 검사한다.
echo $?

: 'Output
1
'

test 1 -eq 1
echo $?
: 'Output
0
'
```

이제 `test` 커맨드의 대략적인 쓰임새를 파악했으므로, `if`문에 사용해보자.
   
``` bash
if test -d /bin; then
  echo /bin is directory !
else
  echo /bin is not directory T_T
fi

: 'Output
/bin is directory !
'
```

앞서 설명하였듯이, \[는 `test` 커맨드의 별칭이므로 위와 동일하게 동작하는 코드를 아래와 같이 다시 작성할 수 있다.

``` bash
if [ -d /bin ]; then
  echo /bin is directory !
else
  echo /bin is not directory T_T
fi

: 'Output
/bin is directory !
'
```
   
## if \[\[ ... ]]

\[\[는 기존의 `test` 커맨드와 비교하여 더 다양한 기능을 제공하는 구조이다. *ksh*에서 도입되었으며, *bash*, *zsh*, *yash*, *busybox sh* 또한 이 기능을 지원한다. `[[ ... ]]` 구조 또한 *exit code*를 설정하여 `if` 구문의 조건 분기가 동작할 수 있도록 한다.

특히 `[[ ... ]]` 구조 안에서는 `<`, `>`, `&&`, `||`, `()`를 산술 및 논리연산자로 취급한다. 아래의 코드를 실행하여 그 차이를 명확히 알아보자.

``` bash
if [ 0 == 0 && 1 == 1 ]; then
  echo "error on previous line"
fi

if [ 1 > 0 ]; then
  echo "error on previous line"
  echo "because in [ ... ], < is the redirection operator,"
fi

if [[ 0 == 0 && 1 == 1 ]]; then
  echo "it works well"
fi 

if [[ 1 > 0 ]]; then
  echo "it also works well !"
fi
```

## if \(\( ... ))

`((` 또한 *ksh*의 확장으로, `zsh`와 `bash` 에서도 이를 지원한다. `(( ... ))` 구조는 산술 연산을 위해 사용된다.

``` bash
(( 1 + 3 == 4 ))
echo $?
: 'Output
0
'

read -p "1 + 1 = " ANS
if (( 1 + 1 == $ANS )); then
  echo "Good"
else
  echo "Bad"
fi
```

## if \( ... )

`...`에 해당하는 커맨드를 subshell에서 실행한다. 커맨드의 실행이 완료되면, 해당 커맨드의 *exit code*에 따라 조건 분기를 수행한다. 실행되는 커맨드가 subshell에서 실행되기 때문에 해당 커맨드로 인한 shell 변수의 값이 변경되는 등의 *side-effects*를 방지할 수 있다.

## if command

`command`의 *exit code*에 따라 조건 분기가 수행된다.

## References

["What is the difference between the Bash operators \[\[ vs \[ vs \( vs vs \(\(?", StackExchange](https://www.naver.com)
["리눅스 쉘 스크립트 if 조건 \[, \[\[, \(\(, \( 에 대한 차이 분석", 꾸준함의 미학, Tistory](https://paradise7.tistory.com/42)
