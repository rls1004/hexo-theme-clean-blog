---
layout: post
title: "[Google CTF 2018] Beginners Quest Write-up"
image: /img/hello_world.jpeg
tags: [CTF, write-up, pwnable, misc]
---

## 요약
`misc` `pwnable`

---
#### LETTER (misc)
<img src="/img/google_letter.JPG" border="1">

PDF 파일이다. text와 배경색이 검정색으로 되어있어 글씨가 보이지 않지만 드래그 해서 복사하면 값을 알 수 있다.

CTF{ICanReadDis}


#### OCR IS COOL! (misc)
<img src="/img/google_ocr1.png" border="1">

<img src="/img/google_ocr2.JPG" border="1">

이미지 파일이 주어지는데 메일의 내용 중 VMY{...} 가 플래그 형식과 비슷하여 Caesar 복호화를 했더니 플래그가 나왔다.

VMY{vtxltkvbiaxkbltlnulmbmnmbhgvbiaxk} &rarr; CTF{caesarcipherisasubstitutioncipher}


#### SECURITY BY OBSCURITY (misc)
첨부파일을 받아보면 zip 으로 압축되어 있다. unzip 으로 압축을 여러번 풀면 또 다른 방식으로 압축되어 있다. zip, unzip, 7z, bzip2, gunzip 순서로 압축을 해제하면 마지막으로 패스워드가 걸린 zip 파일이 된다.

advanced ZIP password Recovery 등의 프로그램을 이용하면 패스워드를 알아낼 수 있다. 알아낸 패스워드는 "asdf" 이다.

CTF{CompressionIsNotEncryption}


#### FLOPPY (misc)
<img src="/img/google_floppy.JPG" border="1">

<img src="/img/google_floppy_.JPG" border="1">

ico 파일을 hex editor 로 열어보면 압축 파일이 숨겨져 있는 것을 확인할 수 있다.

CTF{qeY80sU6Ktko8BJW}


#### MOAR (pwn)

> nc moar.ctfcompetition.com 1337

접속하면 socat 의 man 페이지가 뜬다.

<img src="/img/google_moar.JPG" border="1">

![cmd] 를 이용하여 쉘 명령어를 실행시킬 수 있다.

CTF{SOmething-CATastr0phic}


#### ADMIN UI (pwn-re)

> nc mngmnt-iface.ctfcompetition.com 1337

```
=== Management Interface ===
 1) Service access
 2) Read EULA/patch notes
 3) Quit
```
1번 메뉴를 선택하면 패스워드를 입력해야 한다.

```
The following patchnotes were found:
 - Version0.3
 - Version0.2
Which patchnotes should be shown?
../../../etc/passwd
(생략)
user:x:1337:1337::/home/user:
```
2번 메뉴를 선택하면 patchnotes 를 읽을 수 있는데 예를 들어서 "Version0.3" 을 입력하면 "Version0.3" 파일을 읽어서 출력해준다.
이를 이용해서 "../../../etc/passwd" 를 입력하면 사용자 정보를 확인할 수 있다.


```
=== Management Interface ===
 1) Service access
 2) Read EULA/patch notes
 3) Quit
2
The following patchnotes were found:
 - Version0.3
 - Version0.2
Which patchnotes should be shown?
../../../home/user/flag
CTF{I_luv_buggy_sOFtware}
```
user 라는 사용자명을 확인했으면 user의 홈 디렉토리에서 플래그를 읽는다.

CTF{I_luv_buggy_sOFtware}


#### ADMIN UI 2 (pwn-re)

2번 메뉴 선택 후, "../../../proc/self/cmdline" 을 입력하면 "./main" 을 확인할 수 있다. 이게 실행 파일 이름이다.

다시 2번 메뉴 선택 후, "../main" 을 입력하면 실행 파일을 추출할 수 있다.
분석해보면 1번 메뉴 선택시 입력해야 하는 첫 번째 패스워드는 ADMIN UI 에서 읽었던 flag 값이고, 두 번째 패스워드는 길이가 35인 값 아무거나 통과된다.
하지만 플래그를 알아야되므로 좀 더 살펴보면, FLAG 라는 전역 변수에 35개의 값이 있고, 0xC7 이랑 xor 하는 루틴이 보인다.

> FLAG = [0x84, 0x93, 0x81, 0xbc, 0x93, 0xb0, 0xa8, 0x98, 0x97, 0xa6, 0xb4, 0x94, 0xb0, 0xa8, 0xb5, 0x83, 0xbd, 0x98, 0x85, 0xa2, 0xb3, 0xb3, 0xa2, 0xb5, 0x98, 0xb3, 0xaf, 0xf3, 0xa9, 0x98, 0xf6, 0x98, 0xac, 0xf8, 0xba]

xor 연산을 해보면
CTF{Two_PasSworDz_Better_th4n_1_k?}
