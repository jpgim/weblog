+++
title = 'Sqlite Context Manager'
date = '2026-07-07T17:23:20-03:00'
draft = false

tags = ["python", "sqlite3", "built-ins"]
+++

En python se recomienda gestionar recursos con context-managers. Es más limpio, es más legible y previene olvidos como omitir `close()` 
Pero el context manager de sqlite **no** se comporta igual que otros context manager o mejor dicho: no se comporta como uno asume que se comporta.

Lo explico mejor con unos ejemplos. (*nota: en los ejemplos uso python 3.14.5*)


```python
>>> # Ejemplo para abrir archivos.
... with open("file.txt", "w") as archivo:
...     archivo.write("Se graba esta linea al archivo\n")
... 
31
>>> 
```
Aquí context manager se encarga de cerrar el archivo. Si uno intenta acceder al archivo después del bloque *with* se tiene una excepción "I/O operation on closed file"

```python
>>> archivo.write("grabamos otra linea al archivo\n")
Traceback (most recent call last):
  File "<python-input-1>", line 1, in <module>
    archivo.write("grabamos otra linea al archivo\n")
    ~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
ValueError: I/O operation on closed file.
>>> 
```
Ahora bien, si usamos el context manager de sqlite para administrar una conexión tendremos el siguiente comportamiento

*nota: usamos una base de datos en memoria para los ejemplos*

```python
>>> import sqlite3
... with sqlite3.connect(":memory:") as conn:
...     conn.execute("CREATE TABLE persona(id INTEGER PRIMARY KEY, nombre TEXT NOT NULL, apellido TEXT NOT NULL)")
...     conn.execute("INSERT INTO persona VALUES (1, 'Juan', 'Gonzalez')")
...     rows = conn.execute("SELECT * FROM persona").fetchall()
... 
... print(rows)
... 
[(1, 'Juan', 'Gonzalez')]
>>> 
```

Entonces ahora yo no podría seguir insertando en la tabla ¿cierto?

```python
>>> conn.execute("INSERT INTO persona VALUES (2, 'Pedro', 'Gutierrez')")
... rows = conn.execute("SELECT * FROM persona").fetchall()
... print(rows)
... 
[(1, 'Juan', 'Gonzalez'), (2, 'Pedro', 'Gutierrez')]
>>> 
```
¿Qué paso aquí? Resulta que el context manager es para **gestionar transacciones, no conexiones**.

De los [docs](https://docs.python.org/3/library/sqlite3.html#how-to-use-the-connection-context-manager): "*A Connection object can be used as a context manager that automatically commits or rolls back open transactions when leaving the body of the context manager. If the body of the with statement finishes without exceptions, the transaction is committed. If this commit fails, or if the body of the with statement raises an uncaught exception, the transaction is rolled back*" 

```python
>>> # ok cerremos la conexión
... conn.close()
>>> # ahora si, error
... conn.execute("INSERT INTO persona VALUES (3, 'Maria', 'Gonzalez')")
Traceback (most recent call last):
  File "<python-input-5>", line 2, in <module>
    conn.execute("INSERT INTO persona VALUES (3, 'Maria', 'Gonzalez')")
    ~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
sqlite3.ProgrammingError: Cannot operate on a closed database.
>>> 
```

Entonces, ¿cuales son las alternativas?

1) usar context manager para gestionar transacciones como lo indican los docs (y los ejemplos)
2) usar closing de contextlib

#### Alternativa 1:
```python
>>> conn2 = sqlite3.connect(":memory:") #el context manager se encargará de commit o rollback
... with conn2:
...     conn2.execute("CREATE TABLE persona(id INTEGER PRIMARY KEY, nombre TEXT NOT NULL, apellido TEXT NOT NULL)")
...     conn2.execute("INSERT INTO persona VALUES (1, 'Juan', 'Gonzalez')")
...     conn2.execute("INSERT INTO persona VALUES (2, 'Pedro', 'Gutierrez')")
...     rows = conn2.execute("SELECT * FROM persona").fetchall()
...     # sin errores, el context manager hace conn2.commit()
... 
... print(rows)
... 
... with conn2:
...     conn2.execute("INSERT INTO persona VALUES (3, 'Maria', 'Gonzalez')")
...     conn2.execute("INSERT INTO persona VALUES (1, 'Carlos', 'Ramirez')")
...     # violacion de clave primaria, conn2.rollback()
...     # la fila (3, 'Maria', 'Gonzalez') no se inserta
... 
... 
[(1, 'Juan', 'Gonzalez'), (2, 'Pedro', 'Gutierrez')]
<sqlite3.Cursor object at 0x73d412f70640>
Traceback (most recent call last):
  File "<python-input-1>", line 15, in <module>
    conn2.execute("INSERT INTO persona VALUES (1, 'Carlos', 'Ramirez')")
    ~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
sqlite3.IntegrityError: UNIQUE constraint failed: persona.id
>>> with conn2:
...     rows = conn2.execute("SELECT * FROM persona").fetchall()
... print(rows)
... 
... # cerramos
... conn2.close()
... 
[(1, 'Juan', 'Gonzalez'), (2, 'Pedro', 'Gutierrez')]
>>>  
```

#### Alternativa 2:
```python
>>> # solo nos interesa abrir una conexión, hacer operaciones y que se cierre al finalizar
... from contextlib import closing
... 
... with closing(sqlite3.connect(":memory:")) as conn3:
...     conn3.execute("CREATE TABLE persona(id INTEGER PRIMARY KEY, nombre TEXT NOT NULL, apellido TEXT NOT NULL)")
...     conn3.execute("INSERT INTO persona VALUES (1, 'Juan', 'Gonzalez')")
...     rows = conn3.execute("SELECT * FROM persona").fetchall()
... 
... print(rows)
... 
... # confirmamos que esta cerrada la conexión
... try:
...     conn3.execute("SELECT 1")
... except sqlite3.ProgrammingError as exp:
...     print(f"la conexion esta cerrada: {str(exp)}")
...     
[(1, 'Juan', 'Gonzalez')]
la conexion esta cerrada: Cannot operate on a closed database.
```