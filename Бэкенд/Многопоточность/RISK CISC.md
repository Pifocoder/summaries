Стадии
1) fetch
2) decode
3) execution
4) memory access
5) write back

## Hazards:
### Data hazards:
1) read after write
2) write after read
3) write after write
example:
```
add r1, r2, r3
add r3, r2, r4
add r4, r2, r5
```

## Branch prediction:

#### if в ассемблере
`jz r1, r2`

если r4 == r5 будет прыжок в r2.
```
cmp r4, r5
je r2
```

Ветвления очень мешают исполняться, пример:
```
cmp r1, r3
	je r4
add r2, r5, r6
```
тут сначала нужно дождаться add перед jump