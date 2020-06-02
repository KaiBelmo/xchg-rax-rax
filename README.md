
# Write-up "xchg rax,rax"
xchg rax,rax  It’s a collection of assembly riddles. The book contains 0x40 short assembly snippets, with no text.

## 0x00
```
xor      eax,eax
lea      rbx,[0]
loop     $
mov      rdx,0
and      esi,0
sub      edi,edi
push     0
pop      rbp
```
Different ways to set registers to 0 or implementation of `nop`.

## 0x01
```
.loop:
    xadd     rax,rdx
    loop     .loop
```
it execute xadd instruction in a loop.

implementation of xadd in c:
```
int Temporary, Source, Destination;
scanf("%d %d", &Source, &Destination);
printf("Source = %d    |   Destination = %d    |   Temporary = %d\n", Source, Destination, Temporary);
// Exchanges the first operand with the second operand then loads the sum of the two values into the destination operand.
Temporary = Source + Destination;
Source = Destination;
Destination = Temporary;
printf("Source = %d    |   Destination = %d    |   Temporary = %d\n", Source, Destination, Temporary);
```
execute && debugging
```
(gdb) i r rax                  #rax = 0, rdx = 1
    $rax            0x0	0
(gdb) i r rax
    $rax            0x1	1
(gdb) i r rax
    $rax            0x1	1
(gdb) i r rax
    $rax            0x2	2
(gdb) i r rax
    $rax            0x3	3
```
## 0x02
```
neg      rax            ; cf = 1 if rax != 0
sbb      rax,rax
neg      rax
```
neg instruction reverse signs of a number by converting number to two compliments `1 = 00000001 / -1 = 11111111`

implementation of neg in asm:
```
xor  ecx, ecx      ; ecx = ecx ^ ecx
sub  ecx, eax      ; ecx = 0 - eax
```
and sbb equals to `des = des - (src + CF)`
if we initialize `rax = 1`

```
-1 - ( -1 + 1 ) = -1
```
## 0x03
```
sub      rdx,rax         ; rdx = rdx - rax
sbb      rcx,rcx         ; carry flag will be 0 if rdx was greater or equal to rax
and      rcx,rdx         ; rcx & rdx = 0
add      rax,rcx         ; simple arithmetic instruction
```
we initialize	rax = 1 and rdx = 2
```
if ( rdx > rax ) {        // CF=0
  rdx -= rax
  rcx = 0                 // AND 0 (rcx) with anything (in rdx) is nop here.
                          // ADDing 0 (rcx) to rax is a nop
}
else {                    // rax > rdx (CF=1)
  rdx -= rax
  rcx = -1                // all bits on
  rcx = rdx               // code is rcx &= rdx
                          // remember -1 & x == x
  rax += rcx
}
```
## 0x04
```
xor      al,0x20
```
convert code into python
```
>>> chr(65)
'A'
>>> chr(65 ^ 0x20)
'a'
```
## 0x05
```
sub      rax,5
cmp      rax,4
```
nothing special
## 0x06
```
not      rax        ; ~rax = -2
inc      rax        ; rax++
neg      rax        ; By applying the two’s complement twice they cancel each other
````
not is bitwise operator, inc is `add rax, 1`

we initialize	rax = 1
## 0x07

```
inc      rax
neg      rax
inc      rax
neg      rax
```
4 instruction they cancel each other
## 0x08
```
add      rax,rdx
rcr      rax,1       ; right rotate with carry
```
rcr is rotate right for the carry flag

## 0x09
```
shr      rax,3        ; rax >>= 3
adc      rax,0        ; rax = rax + 0 + CF
```
shr is shifts the bits of the first operand right by the specified number of bits in this case is 3 .

`ADC dest, 0` is just adding the CF
we initialize	rax = 8

## 0x0a
```
add      byte [rdi],1
.loop:
inc      rdi
adc      byte [rdi],0
loop     .loop
```
it increments the least-significant byte at the memory location pointed by rdi
## 0x0b

```
not      rdx
neg      rax
sbb      rdx,-1
```
```
rax = 0
rdx = 5
rdx = ~rdx + 1
```
## 0x0c
```
mov      rcx,rax
xor      rcx,rbx       ; rcx = rcx ^ rbx
ror      rcx,0xd       ; rotate rcx right 0xd(13) positions
ror      rax,0xd
ror      rbx,0xd
xor      rax,rbx
cmp      rax,rcx       ; compare rax and rcx
```
In a rotate instruction
```
mov eax,0xA //the value in eax is 0000 0000 0000 0000 0000 0000 0000 1010
ror eax,2 // now eax will be      1000 0000 0000 0000 0000 0000 0000 0010
```
implementating this code in c
```
int rcx , rax = 123456, rbx = 987654;
rcx = rax;
rcx = (rcx ^ rbx) >> 13;                // 001110111
rax = (rax >> 13) ^ (rbx >> 13);
printf("rcx = %d      |       rax = %d", rcx, rax);
```
## 0x0d
```
mov      rdx,rbx
xor      rbx,rcx
and      rbx,rax
and      rdx,rax
and      rax,rcx
xor      rax,rdx
cmp      rax,rbx       ; compare rax and rbx
```
implementating this code in c
```
int rax = 123, rbx = 456, rcx = 789, rdx = rbx;
rbx = (rbx ^ rcx) & rax;
rax = (rax & rcx) ^ (rdx & rax);
printf("rax = %d        |       rbx = %d", rax, rbx);
```
## 0x0e
```
mov      rcx,rax
and      rcx,rbx
not      rcx
not      rax
not      rbx
or       rax,rbx
cmp      rax,rcx
```
implementating this code in c
```
int rax = 1337, rbx = 7331, rcx;
rcx = rax;
rcx = ~(rcx & rbx);
rax = ~rax | ~rbx;
printf("rax = %d        |       rbx = %d        |           rcx = %d", rax, rbx, rcx);
```
## 0x0f
```
.loop:
    xor      byte [rsi],al
    lodsb
    loop     .loop
```
what lodsb does is:
```
mov al,[rsi]
inc rsi           ; (or dec, according to direction flag)
```
simple xor encryption, implementating this code in c
```
int i;
char rax;
char *str = "blablabla";
while(str[i] != '\0'){
		str[i] = str[i] ^ rax;
		rax = str[i];
    i++;
}
```

## 0x10
```
push	 rax		; push the value of rax onto the stack
push	 rcx		; push the value of rcx onto the stack
pop	  rax		; load the value on top of the stack into rax
pop	  rcx		; load the value on top of the stack into rcx
xor      rax,rcx
xor      rcx,rax
xor      rax,rcx
add      rax,rcx
sub      rcx,rax
add      rax,rcx
neg      rcx
xchg     rax,rcx
```
implement swap in different ways

## 0x11
```
.loop:
    mov      dl,byte [rsi]
    xor      dl,byte [rdi]
    inc      rsi              ; counter for the first buffer
    inc      rdi              ; counter for the second buffer
    or       al,dl
    loop     .loop
```
compare two strings

## 0x12
```
mov      rcx,rdx
and      rdx,rax
or       rax,rcx
add      rax,rdx
```
just addition between rax and rdx

## 0x13

* I will update this repo when i solve new challenges.
