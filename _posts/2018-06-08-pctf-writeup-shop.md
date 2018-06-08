---
layout: post
title: "[Plaid CTF] shop"
image: /img/hello_world.jpeg
---

## 요약
`GOT Overwrite` `pwnable`

---
## 프로그램 분석
4개의 명령어를 사용할 수 있다.

```
[a] allocate item (0x400A5B)
[l] list item (0x400D0E)
[c] check out and buy (0x400B78)
[n] shop name (0x400D6A)
```
<br>
#### [a] allocate item (0x400A5B)
#### item 구조:

| offset | desc |
| :------ |:--- |
| 0~7 | &head (\*0x6021F0) |
| 8~11 | random bytes(0-9,a-f) |
| 12~43 | name |
| 44~299 | description |
| 300~304 | price |

이 중에서 `name`, `description`, `price` 를 입력할 수 있다.
head 는 이전에 할당된 item 의 주소를 가리키고, 그 뒤 4바이트는 `0-9` `a-f` 로 구성된 난수를 저장한다.

#### [l] list item (0x400D0E)
<center>
<div class="mermaid">
graph LR
A(item2) --> |head| B(item1)
B --> |head| C(item0)
</div>
</center>
head (0x6021F0) 에 저장된 주소를 시작으로, 각 item 의 head 를 따라가며 목록을 출력한다.
- item+12 : %s (name)
- item+300 : %.2f (price)
- item+44 : %s (description)

출력 방식은 위와 같다.

#### [c] check out and buy (0x400B78)
`heystack` 에 0x10004 만큼의 값을 입력할 수 있다. `heystack` 과 각 item 들의 `random bytes` 를 비교하여 4바이트 모두 일치하는 부분이 있으면 해당 item 을 `buying list` 에 넣는다. `buying list` 는 32개의 포인터를 담을 수 있는 전역 변수 배열이다.

#### [n] shop name (0x400D6A)
프로그램 시작 시 입력했던 `shop name` 을 변경할 수 있다.

---
## 취약점
**[a] allocate item** 명령에서 item 을 추가할 때 `item_count` 가 32 이하인지 확인한다. 0 부터 32 까지, 총 <b>33개</b>의 item 을 추가할 수 있다.

**[c] check out and buy** 명령에서는 `random bytes` 비교를 통과한 item 들을 `buying_list` 에 추가하는데, 이 배열은 <b>32*8</b> 의 크기를 갖고, 바로 뒤에는 `shop_name` 포인터가 있다.

| offset | desc |
| :------ |:--- |
| 0~7 | &head (\*0x6021F0) |
| 8~11 | random bytes(0-9,a-f) |
| 12~43 | name |
| 44~299 | description |
| 300~304 | price |

**[a]** 와 **[c]** 명령어를 이용하여 `shop_name` 포인터를 덮어 쓰고, [l] 명령어를 통해 <span style="color:#cf3030">information leak</span> 을 시도하여 라이브러리의 주소를 알아낼 수 있다.
