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
<kbd>
<img src="/img/google_letter.JPG">
</kbd>

PDF 파일이다. text와 배경색이 검정색으로 되어있어 글씨가 보이지 않지만 드래그 해서 복사하면 값을 알 수 있다.

CTF{ICanReadDis}

#### OCR IS COOL! (misc)
<kbd>
<img src="/img/google_ocr1.png">
</kbd>

<kbd>
<img src="/img/google_ocr2.JPG">
</kbd>

이미지 파일이 주어지는데 URL의 값이 수상하여 Caesar 복호화를 했더니 플래그가 나왔다.
