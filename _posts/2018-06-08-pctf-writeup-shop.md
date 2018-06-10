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
information leak, GOT Overwrite

#### 라이브러리 내의 system 함수 주소 알아내기
**[a] allocate item** 명령에서 item 을 추가할 때 `item_count` 가 32 이하인지 확인한다. 0 부터 32 까지, 총 <b>33개</b>의 item 을 추가할 수 있다.

**[c] check out and buy** 명령에서는 `random bytes` 비교를 통과한 item 들을 `buying_list` 에 추가하는데, 이 배열은 <b>32*8</b> 의 크기를 갖고, 바로 뒤에는 `shop_name` 포인터가 있다.

즉, **[a]** 와 **[c]** 명령어를 이용하여 `shop_name` 포인터를 item0로 덮어 쓸 수 있고, `shop_name` 은 **[n]** 명령어로 수정 가능하므로 item0 구조체의 값을 수정할 수 있다.

**[l]** 명령어는 item 의 head 가 0이 아닐 때까지 추적하면서 내용을 출력하므로 head 값을 조작함으로써 <span style="color:#cf3030">information leak</span> 이 가능해진다.

| offset | desc | value |
| :------ |:--- | :---|
| 0~7 | &head (\*0x6021F0) |<span style="color:#1070c0"><b>0</b></span>|
| 8~11 | random bytes(0-9,a-f) ||
| 12~43 | name (%s) | <span style="color:#1070c0"><b>leak</b></span> |
| 44~299 | description (%s) |  |
| 300~304 | price (%.2f) |  |

item+12 위치의 값이 출력되므로, 출력하고자하는 주소에서 **12**를 뺀 주소를 head 에 넣어야 한다. 또한, head 가 0이 아니라면 추적을 계속 진행하려고 하여 Segmentation Fault 가 발생할 수 있으므로 반드시 **0** 으로 설정되어야 한다.

<br>
#### 실행 흐름 바꾸기
information leak 을 통해 라이브러리 내에 있는 system 함수의 주소를 알아냈으면, 다음으로는 프로그램 내에 존재하는 GOT 테이블을 덮어써야 한다.
<center>
<div class="mermaid">
graph LR
A(item2) --> |head| B(item1)
B --> |head| C(item0)
C --> |head| D(GOT table)
</div>
</center>

**[n]** 명령어를 이용하여 item0 를 수정할 수 있는 상태가 되었으니 item0 의 구조를 아래와 같이 바꿔주고 **[c]** 명령을 호출하면 shop_name 을 GOT 의 주소로 덮어 쓸 수 있다.

| offset | desc | value |
| :------ |:--- | :---|
| 0~7 | &head (\*0x6021F0) |<span style="color:#1070c0"><b>GOT</b></span>|
| 8~11 | random bytes(0-9,a-f) |<span style="color:#1070c0"><b>\x11\x11</b></span>|
| 12~43 | name (%s) |  |
| 44~299 | description (%s) |  |
| 300~304 | price (%.2f) |  |

이때, item0 의 random bytes 는 `heystack` 에서 포함하지 않고 있는 값이어야 한다. item32~item1 까지 32개의 item 을 buying_list 에 추가하고 item0 는 추가하지 않고 head 를 따라가서 GOT 를 buying_list 에 추가하도록 유도하여 shop_name 을 덮어써야 한다.
shop_name 이 GOT 의 주소를 가리키게 되었으면 **[n]** 명령어를 이용하여 해당 GOT 에 system 함수의 주소를 덮어쓴다.
