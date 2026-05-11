## Problema 7 — Guess-Sort y Fun-Sort

### 7A. ¿Por qué guess-sort mejora a bozo-sort+_opt?

La diferencia clave está en la definición de un "paso" y el aprovechamiento
del tiempo.

**bozo-sort+_opt:** elige un par de índices al azar y pregunta si están mal
ordenados. Si el par ya está bien, no hace nada y desperdicia el turno. A
medida que el arreglo se va ordenando, encontrar un par malo es como buscar
una aguja en un pajar — el algoritmo pierde cada vez más tiempo eligiendo
pares que ya están en su lugar.

**guess-sort:** sigue buscando pares al azar internamente hasta que encuentra
uno malo, y solo entonces hace el intercambio.

La consecuencia directa es que en guess-sort cada intercambio está garantizado
a reducir el número de inversiones en el arreglo. No existen pasos
desperdiciados en términos de intercambios, lo que lo hace significativamente
más eficiente en comparaciones totales.

### 7B. Fun-Sort — Caso Promedio

Analizando el Teorema 6 y los resultados del paper, el tiempo de ejecución
de Fun-Sort es:
T(n) = O((n + F) log n)

donde `F` es el número de inversiones en el arreglo de entrada. Para que
Fun-Sort sea rápido (`O(n log n)`), `F` tendría que ser muy pequeño —
específicamente `o(n²/log n)`.

Sin embargo, del Problema 5 sabemos que en una permutación aleatoria el
número esperado de inversiones es:
E[F] = n(n−1)/4  ≈  n²/4

Como `n²/4` crece mucho más rápido que el umbral del teorema, el caso
promedio de Fun-Sort termina siendo:
E[T] = O((n + n²/4) log n) = O(n² log n)

Aunque Fun-Sort es sofisticado en su uso de búsqueda binaria, en un arreglo
totalmente desordenado sigue teniendo rendimiento cuadrático — similar a
Insertion Sort pero con un factor `log n` adicional por la búsqueda. Solo
alcanza `O(n log n)` cuando el arreglo de entrada ya está casi ordenado, es
decir, cuando `F` es suficientemente pequeño.