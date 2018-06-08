---
layout: post
title: "[Plaid CTF] shop"
image: /img/hello_world.jpeg
---

## 요약
`GOT Overwrite`

## 프로그램 분석
4개의 명령어를 사용할 수 있다.

```
[a] allocate item (0x400A5B)
[l] list item (0x400D0E)
[c] check out and buy (0x400B78)
[n] shop name (0x400D6A)
```
명령어 목록:

| cmd | desc | addr |
| :------ |:--- | :--- |
| a | allocate item | 0x400A5B |
| l | list item | 0x400D0E |
| c | check out and buy | 0x400B78 |
| n | shop name | 0x400D6A |

<br>

#### [a] allocate item (0x400A5B)
item 구조:

| offset | desc |
| :------ |:--- |
| 0~7 | &head (\*0x6021F0) |
| 8~11 | random bytes(0-9,a-f) |
| 12~43 | name |
| 44~299 | description |
| 300~304 | price |

이 중에서 name, description, price 를 입력할 수 있다.
head 는 이전에 할당된 item 의 주소를 가리키고, 그 뒤에 `urandom` 으로부터 가져온 난수 4바이트를 저장한다.
