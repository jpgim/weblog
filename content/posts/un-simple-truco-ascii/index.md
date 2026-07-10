+++
title = 'Un Simple Truco ASCII'
date = '2026-07-06T15:07:41-03:00'
draft = false

tags = ["trucos", "c", "ascii"]
+++

Si uno presta atención a la tabla [ASCII](https://es.wikipedia.org/wiki/ASCII) se da cuenta rápidamente de algo:

|decimal| binario |caracter|
|-------|---------|--------|
|  65   | 1000001 |   A    |
|  66   | 1000010 |   B    |
|  67   | 1000011 |   C    |
|  68   | 1000100 |   D    |
|  69   | 1000101 |   E    |
|  70   | 1000110 |   F    |
...
...
|  97   | 1100001 |   a    |
|  98   | 1100010 |   b    |
|  99   | 1100011 |   c    |
|  100  | 1100100 |   d    |
|  101  | 1100101 |   e    |
|  102  | 1100110 |   f    |
...
...


```pycon
>>> 97-65
32
>>> from math import log2
>>> log2(32)
5.0
 ```

Concretamente:

|  65   | 1**0**00001 |   A    |

|  97   | 1**1**00001 |   a    |

Es decir, las mayúsculas y minúsculas difieren en el 5 bit y es así por diseño. De esta forma es posible cambiar a mayúsculas/minúsculas de forma sencilla manipulando ese bit.

Entonces una posible implementación en C de toupper() podría ser:

```c
char toupper(char c)
{
    return c & ~(1 << 5);
}
```
y para una función tolower():

```c
char tolower(char c)
{
    return c | (1 << 5);
}
```
y para intercambiar mayúsculas/minúsculas indistintamente:

```c
char swapcase(char c)
{
    return c ^ 0x20;
}
``` 

Después de compilar se traducen a operaciones baratas, por ejemplo un compilador como `gcc` con un flag de optimización `-O2` termina haciendo:

```console
$ objdump -d --section=.text toupper.o

toupper.o:     formato del fichero elf64-x86-64


Desensamblado de la sección .text:

0000000000000000 <toupper>:
   0:	f3 0f 1e fa          	endbr64 
   4:	89 f8                	mov    %edi,%eax
   6:	83 e0 df             	and    $0xffffffdf,%eax
   9:	c3                   	ret    
```

Que es rapidísimo y en una época donde los recursos eran escasos se buscaban este tipo de optimizaciones.



Estos son solo ejemplos didácticos para lograr comprender los porqués. Cuando hay algo que llama mi atención intento comprender cómo funciona y cual es la idea detrás. Por supuesto que para el uso cotidiano siempre conviene usar [funciones estándar](https://es.cppreference.com/c/string/byte/toupper).
