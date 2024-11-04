---
title: 리눅스 혹은 서버에서 구글 드라이브 대용량 파일 다운로드 하기
categories:
 - Troubleshooting
tags: 
 - linux
 - google drive
---

리눅스 혹은 서버에서 구글 드라이브의 대용량 파일 다운로드 하기

---

share link : https://drive.google.com/file/d/{파일ID}/view?usp=sharing 일 때,
terminal에 가서 아래와 같이 작성.

> ```wget --load-cookies /tmp/cookies.txt "https://docs.google.com/uc?export=download&confirm=$(wget --quiet --save-cookies /tmp/cookies.txt --keep-session-cookies --no-check-certificate 'https://docs.google.com/uc?export=download&id={파일ID}' -O- | sed -rn 's/.*confirm=([0-9A-Za-z_]+).*/\1\n/p')&id={파일ID}" -O {파일이름} && rm -rf /tmp/cookies.txt```