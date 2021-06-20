---
layout: article
title: os.system 에러 핸들링하기
tags:
  - os.system
  - Python
  - 한글
key: python-os-system-error
---

## Intro

Python에 os.system을 사용해서 작업을 하고 있었다. 와이파이를 코드 내부에서 연결시키도록 하는 장치가 필요해서 nmcli를 사용했고, 콘솔에서 실행시켜야 했기 때문에 os.system을 사용하였다. 와이파이가 연결이 되지 않는 경우를 대비하여 try-except를 사용해서 에러 처리를 했는데, 분명히 와이파이가 연결이 안 되었음에도 불구하고 except블록의 코드가 실행되지 않았다. 이 글에서는 위 상황에 대해 설명하고 내가 찾은 해결 방법을 소개한다.

## os.system

Python의 os.system 메소드는 인자로 전달한 명령어를 해당 운영체제에서 실행시켜 주고 그 결과를 출력해주는 함수이다. 다음 코드를 보면 어떻게 작동하는지 알 수 있다. 다음과 같이 Mac에서 `ls`를 실행하면 Python 코드가 실행되고 있는 디렉터리의 파일과 폴더를 보여준다.

```python
>>> os.system('ls')
Audio Music Apps
GarageBand
Music
iTunes
0
```

여기에서 마지막 출력을 보면 0이라는 출력이 있다는 것을 알 수 있다. 아래와 같이 `ls` 가 아니라 `sl`을 입력하여 명령에서 오류가 발생하는 경우 마지막에 출력되는 숫자가 0이 아니게 된다.

```python
>>> os.system('sl')
sh: sl: command not found
32512
```

### 리턴값의 의미?

이 숫자가 뭘 의미하는지 고민을 해봤다. Python 공식 문서에 따르면 os.system이 C의 standard를 따른다고 설명되어 있다 ([링크](https://docs.python.org/3.7/library/os.html#os.system)). C언어를 공부해본 경험에 의하면 보통 main 함수의 return값이 오류가 아니면 0을 리턴하게 되어 있다. C++ 알고리즘 문제를 풀 때도 중간에 멈추기 위해서 `exit(0);` 함수를 호출하는데, 이것 역시 리턴값이라고 알고 있다.

위 지식을 바탕으로 추측하였을 때, os.system은 콘솔에서 실행한 명령어의 결과를 출력하고 그 리턴값을 리턴하는 함수인 것이다.

### Raises Exception?

또한, os.system에서 에러가 발생했을 때는 일반적인 Exception을 raise하지 않는다는 것도 알 수 있다. 아래와 같이 0으로 나누기를 하는 경우 `ZeroDivisionError`가 raise되고, 그 에러를 Traceback할 수 있다.

```python
>>> 0/0
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ZeroDivisionError: division by zero
```

## 해결책

위의 결과를 종합해본 결과, os.system은 실행한 명령어에서 에러가 발생하더라도 Exception을 raise하지 않고 그 결과를 그대로 출력해주는 함수라는 것을 알 수 있었다. 또한, C의 표준을 따라 정상 종료의 경우 0을, 아닌 경우 해당하는 에러코드를 리턴한다.

따라서, 우리는 다음과 같은 방법으로 error code을 체크하여 에러가 발생한 경우 Exception을 raise하는 것으로 에러 처리를 할 수 있다.

```python
res = os.system(cmd)
if res == 0:
  pass
else:
  raise Exception(f'Error occured while executing {cmd}')
```

Python 의 공식 문서에 따르면 [subprocess](https://docs.python.org/3.9/library/subprocess.html#module-subprocess) 모듈을 사용하는 것이 더 좋다고 되어 있으므로, subprocess를 사용하는 것도 한 방법일 수 있겠다.
