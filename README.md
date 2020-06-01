
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
* I will update this repo when i solve new challenges.
