+++
title = 'Anagrama'
date = '2026-07-06T20:46:02-03:00'
draft = false

tags = ["python", "anagrama", "aritmética"]
+++

Un anagrama, es una palabra o frase que resulta de la reordenación de las letras de otra.
Por ejemplo **roma** es anagrama de **mora** y **entrelazar** es anagrama de **relanzarte**

Por otro lado, una implicancia del **Teorema fundamental de la aritmética**, es que cualquier número entero **n** mayor que **1** puede escribirse de **manera única**, salvo el orden, como un producto de números primos.

Ejemplo: 12 se puede escribir como 2 × 2 × 3 o bien 2² × 3


Se pueden vincular ambos conceptos y llegar a una conclusion interesante:
Si asociamos cada letra del abecedario a un numero primo mayor que 1, podremos determinar si una palabra es anagrama de otra si tienen el mismo numero obtenido como resultado de la multiplicación de los valores asociados a las letras que lo componen

Es decir, si hacemos a->2, b->3, c->5, d->7, e->11 ...

y para cada palabra multiplicamos esos valores:

**roma**: `61×47×41×2 = 235094`  
**mora**: `41×47×61×2 = 235094`

Para palabras cortas es trivial, pero para palabras un poco más largas podemos hacer uso de programación


```python
from math import prod
from operator import itemgetter
```


```python
letra_a_primo =  {
    'a': 2, 'b': 3, 'c': 5, 'd': 7, 'e': 11, 'f': 13,
    'g': 17, 'h': 19, 'i': 23, 'j': 29, 'k': 31, 'l': 37,
    'm': 41, 'n': 43, 'o': 47, 'p': 53, 'q': 59, 'r': 61,
    's': 67, 't': 71, 'u': 73, 'v': 79, 'w': 83, 'x': 89,
    'y': 97, 'z': 101
}
```


```python
palabra1 = "entrelazar"
palabra2 = "relanzarte"
producto_palabra1 = prod(itemgetter(*palabra1)(letra_a_primo))
producto_palabra2 = prod(itemgetter(*palabra2)(letra_a_primo))
                        
```


```python
print(producto_palabra1 == producto_palabra2)

```

    True


Donde [itemgetter()](https://docs.python.org/es/3/library/operator.html#operator.itemgetter) es una función del módulo operator que hace más elegante la extracción de los valores de un diccionario en este caso.

También podría haberse usado una list comprehension quedando:


```python
palabra1 = "entrelazar"
palabra2 = "relanzarte"
producto_palabra1 = prod([letra_a_primo.get(k) for k in palabra1])
producto_palabra2 = prod([letra_a_primo.get(k) for k in palabra2])
```


```python
print(producto_palabra1 == producto_palabra2)

```

    True
