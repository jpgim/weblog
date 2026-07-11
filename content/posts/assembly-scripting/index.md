+++
title = 'Assembly Scripting'
description = 'post con espíritu mundialista ⚽'
date = '2026-07-10T18:54:49-03:00'
draft = false
tags = ["tcc", "c", "compiler", "assembly", "GAS"]
+++

Bueno, quizás este exagerando un poco. Solo un poco porque es más fácil y divertido de lo que parece

[Tiny C Compiler](https://bellard.org/tcc/) es un compilador de C pequeño y bastante completo que soporta C99. También es muy rápido.

Una característica interesante es la capacidad de ejecutar código C como si fuera un script con `tcc -run` ya que compila directamente en memoria. 

Ejemplo


```c
/* archivo hola.c */
#include <stdio.h>

int main(void){
    
    puts("Hola mundo");
    return 0;

}
```

```console
$ tcc -run hola.c
Hola mundo
```


¿Qué tan lejos se puede llevar esta idea?

Leyendo la [documentación](https://bellard.org/tcc/tcc-doc.html#asm) se observa que `tcc` integra su propio assembler en "gas-like syntax". Por lo que podemos hacer algo como esto:

```gas
.section .rodata
mensaje:
    .ascii "Hola mundo\n" 
mensaje_len = . - mensaje


.text
.globl _start


_start:
    movq    $1, %rax
    movq    $1, %rdi
    movq    $mensaje_len, %rdx
    leaq    mensaje(%rip), %rsi
    syscall

    movq $60, %rax
    movq $1, %rdi
    syscall
```

compilar y ejecutar:

```console
$ tcc -nostdlib -o hola hola.s
$ ./hola
Hola mundo
```

Funciona perfecto.

Ahora el paso extra es ejecutar un archivo assembly como si fuera un script. Antes hay que hacer un par de ajustes:

* cambiar `_start` por `main` debido a como funciona `tcc -run` internamente
* agregar shebang `#!` 
* darle permisos de ejecución al archivo 


```gas {hl_lines=[1,16,19]}
#!/usr/bin/env -S tcc -run
.section .rodata
mensaje:
    .byte   0xe2, 0x9a, 0xbd, 0xe2, 0x9a, 0xbd, 0xf0, 0x9f
    .byte   0x87, 0xa6, 0xf0, 0x9f, 0x87, 0xb7, 0xf0, 0x9f
    .byte   0x87, 0xa6, 0xf0, 0x9f, 0x87, 0xb7, 0x20, 0xc2
    .byte   0xa1, 0x20, 0x56, 0x61, 0x6d, 0x6f, 0x73, 0x20
    .byte   0x41, 0x72, 0x67, 0x65, 0x6e, 0x74, 0x69, 0x6e
    .byte   0x61, 0x21, 0x20, 0xf0, 0x9f, 0x87, 0xa6, 0xf0
    .byte   0x9f, 0x87, 0xb7, 0xf0, 0x9f, 0x87, 0xa6, 0xf0
    .byte   0x9f, 0x87, 0xb7, 0xe2, 0x9a, 0xbd, 0xe2, 0x9a, 0xbd, 0xa
mensaje_len = . - mensaje


.text
.globl main


main:
    movq    $1, %rax
    movq    $1, %rdi
    movq    $mensaje_len, %rdx
    leaq    mensaje(%rip), %rsi
    syscall

    movq $60, %rax
    movq $1, %rdi
    syscall
```

```console
$ file vamo.s
vamo.s: assembler source, ASCII text
$ chmod +x vamo.s
$ ./vamo.s
⚽⚽🇦🇷🇦🇷 ¡ Vamos Argentina! 🇦🇷🇦🇷⚽⚽
```

