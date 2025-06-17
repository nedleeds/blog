---
title: Circular Buffer
type: docs
date: 2025-06-17
prev: /tech
---


## 정의 

- 컴퓨터 공학에서 circular 버퍼는 circualr queue, cyclic buffer 또는 링버퍼라고 불린다.
- 링버퍼는 마치 끝에서 끝까지 연결된 것처럼 단일의 고정 크기 버퍼를 사용하는 데이터 구조다.


## 동작 요약 

![Circular Buffer Animation](https://upload.wikimedia.org/wikipedia/commons/3/3f/Circular_Buffer_Animation.gif)
> 이미지 출처: [MuhannadAjjan – Wikimedia Commons](https://commons.wikimedia.org/w/index.php?curid=45368479)  
> 라이선스: [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0)

- 24KB 의 키보드용 circular buffer 이미지
- microprocessor 가 응답이 없어서, `write` pointer 가 `read` pointer 를 만날 때, buffer 는 keystroke 기록을 멈춘다.
- keystroke 을 멈추는 것처럼 ring buffer 의 overwrite 을 방지하는 것은 어플리케이션의 기능에 따라 달라진다. 


## 동작 상세 

- 초기에는 read == write 이며, 버퍼는 비어 있음(empty) 상태
> 버퍼 크기: 5
> read 포인터는 다음 읽을 위치를 가리킨다.
> write 포인터는 다음 쓸 위치를 가리킵니다.

```
[ _ ] [ _ ] [ _ ] [ _ ] [ _ ]
  ↑
 R,W
(empty 상태)
```

- A, B를 write 한 경우
> write = (write + 1) % buffer_size
> A는 0번, B는 1번 인덱스에 쓰이게 된다.

```
[ A ] [ B ] [ _ ] [ _ ] [ _ ]
  ↑           ↑
  R           W
(2개 사용됨, 남은 공간 3)
```


- A를 read 한 경우
> read = (read + 1) % buffer_size
> A가 사라지고, read 포인터가 다음 위치인 1번으로 이동합니다.

```
[ _ ] [ B ] [ _ ] [ _ ] [ _ ]
        ↑     ↑
        R     W
(1개 사용됨, 남은 공간 4)
```

- C, D, E를 write 한 경우
> write는 계속 증가하며 끝까지 도달한 뒤 0으로 순환합니다.

```
[ _ ] [ B ] [ C ] [ D ] [ E ]
  ↑     ↑                  
  W     R                  
(4개 사용됨, 남은 공간 1)
```

- F를 write 한 경우 → overwrite 발생
> write = 0 → A가 있던 자리에 F를 쓰게 됨
> 이때 write 포인터가 read 포인터를 따라잡게 되며,
> (write + 1) % size == read → 버퍼가 가득 찼음을 의미
> overwrite 정책이 없으면 에러 발생 가능,
> 혹은 overwrite 허용 시 가장 오래된 데이터를 덮어씌움.

```
[ F ] [ B ] [ C ] [ D ] [ E ]
        ↑                  
       R,W                  
(full 상태 or overwrite 발생)
```

- 상태 판별 조건
> Empty: read == write
> Full: (write + 1) % buffer_size == read
> 포인터 값만으로는 full/empty를 구분할 수 없기 때문에, 별도의 is_full 또는 count 변수를 함께 관리하는 방식도 자주 사용됩니다.

- 링버퍼 공간 관리 시퀀스 
```mermaid
flowchart TD
    Start([Start]) --> CheckFull{(write + 1) % size == read?}
    CheckFull -- Yes --> HandleFull[버퍼가 가득 참 → 쓰기 실패 or 덮어쓰기]
    CheckFull -- No --> WriteData[데이터 write<br/>write = (write + 1) % size]
    WriteData --> UpdateUsed[used += 1<br/>available -= 1]
    UpdateUsed --> WaitRead[다음 요청 대기]

    WaitRead --> ReadReq[Read 요청 발생]
    ReadReq --> CheckEmpty{read == write?}
    CheckEmpty -- Yes --> HandleEmpty[버퍼가 비어 있음 → 읽기 실패]
    CheckEmpty -- No --> ReadData[데이터 read<br/>read = (read + 1) % size]
    ReadData --> UpdateUsedRead[used -= 1<br/>available += 1]
    UpdateUsedRead --> WaitRead

    subgraph 상태 계산
        UsedUsed[used = (write - read + size) % size]
        AvailUsed[available = size - used - 1]
    end

    Start --> 상태계산[상태 계산 필요시 →]
    상태계산 --> UsedUsed
    상태계산 --> AvailUsed
```
