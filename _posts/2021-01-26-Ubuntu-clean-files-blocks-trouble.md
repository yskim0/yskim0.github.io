---
title: Ubuntu 18.04 clean, ***/*** files, ***/*** blocks 문제해결
categories:
 - Troubleshooting
tags: 
 - Ubuntu
 - Troubleshooting

---

Ubuntu 18.04에서 clean, \*\*\*/*** files, \*\*\*/\*\*\* blocks 에러가 떴을 때 고치는 방법을 정리한 글입니다.

---

## 잡소리

오늘 연구실에서 정말 당황한 에러를 마주쳤습니다.

사건의 역시나 CUDA에서 시작... 저번주까지만 해도 잘 됐던 gpu사용이 오늘 하려니 갑자기 찾을 수 없다(?)는 에러가 떴습니다.

침착하게 그 에러 메시지를 구글링해서 스택오버플로우에서 하라는 대로 하고 재부팅했는데 아래와 같은 에러가 뜨고 무한 블랙스크린이었습니다 ㅜㅜ.

![](https://i.stack.imgur.com/fBYUJ.jpg) [이미지 출처](https://askubuntu.com/questions/1222496/system-wont-boot-stuck-at-dev-sda5-clean-xxxx-xxxx-files-yyyy-yyyy-blocks)


진심 저거 봤을 때 눈물이 찔끔 나올 뻔 했는데요 ..ㅋㅋㅋ 스택오버플로우보니까 아예 우분투를 재설치 해야 할 수도 있다는 글을 보고... 진짜 온몸이 오싹해졌습니다.

## Solution

1. 우선 다시 부팅하고 recovery mode (safe mode) 로 들어가야 합니다. 부팅될 때 무한 `Shift` 누르십시오. (우분투 18.04 기준)

2. `Advanced options for Ubuntu` 를 클릭하고, `recovery mode`로 들어갑니다.
제가 캡쳐할 상황은 아니었던지라 다 인터넷에서 들고 오는 점 이해 부탁드립니다^^; [출처](https://askubuntu.com/questions/92556/how-do-i-boot-into-a-root-shell)

![](https://i.stack.imgur.com/01e8n.png)
![](https://i.stack.imgur.com/UP5j7.png)

3. 이제 리커버리 모드에 들어왔는데요. 여기서 방향키를 사용해 `root`로 이동하고 Enter합니다. 이제 터미널을 쓸 수 있습니다! 
![](https://i.stack.imgur.com/tHkmh.png)


이제 터미널에서


`sudo apt-get purge nvidia*`


를 치면 됩니다만, 여기서 저는 

```
dpkg was interrupted, you must manually run 'sudo dpkg --configure -a' to correct the problem
```

라는 에러가 또 뜹니다.


여기서 제 부팅 실패의 원인을 유추해볼 수 있었습니다. 재부팅 전에 했던 동작들에서 뭔가를 건드렸고 그 결과 dpkg 패키지가 제대로 구성되지 않았던 겁니다.(maybe....)


아무튼 이 에러는 상대적으로 쉽게 해결할 수 있습니다.
에러 메시지대로 `sudo dpkg --configure -a` 를 해주면 됩니다.


이제 다시 


`sudo apt-get purge nvidia*`
를 하게 되면 잘 설치 됩니다.


4. `reboot` 를 써서 재부팅합니다 -> 성공!

---

이 일은 저에게 있어 정말 끔찍했던 순간이었습니다. 퇴사해야하나 생각도 했어요 ㅋㅋㅋ 아무튼 빠르게 고치고 퇴근했습니다^^.. 혹시 구글링하다 저와 같은 에러를 발견하시게 된다면, 위 과정을 통해 고치길 바라겠습니다.