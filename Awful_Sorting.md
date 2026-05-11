## Problema 6 — Análisis de Bogo-sort

### 6A. ¿Por qué P[Iₖ] = 1/k!?

La lógica es puramente combinatoria. En un arreglo de `n` elementos distintos
permutado aleatoriamente, los primeros `k` elementos pueden estar organizados
de `k!` formas diferentes, todas con la misma probabilidad. De todas esas
combinaciones posibles, solo una corresponde al orden correcto (ascendente).
Por lo tanto, la probabilidad de que los primeros `k` elementos pasen la
prueba de ordenamiento es exactamente:
P[Iₖ] = 1/k!

### 6B. ¿Por qué E[C] = Σₖ₋₀ P[Iₖ]?

Aquí se aplica la **Identidad de Tail Sum** para variables aleatorias enteras
no negativas. El número de comparaciones `C` es al menos `k` (es decir,
`C ≥ k`) solo si los primeros `k-1` pares ya estaban bien ordenados — que
es exactamente el evento `Iₖ`. Usando la fórmula general:
E[X] = Σₖ₌₁^∞ P(X ≥ k)

Al sumar las probabilidades `1/1! + 1/2! + 1/3! + ...`, la serie converge a
`e - 1 ≈ 1.718`. Esto significa que en promedio Bogo-sort necesita menos de
2 comparaciones para darse cuenta de que un arreglo aleatorio **no** está
ordenado — un resultado sorprendente dado lo ineficiente que es el algoritmo
en general.

### 6C. ¿Por qué el número de intentos I sigue una distribución geométrica?

Porque el algoritmo se comporta exactamente como una serie de ensayos de
Bernoulli independientes:

1. **Independencia:** cada vez que se baraja el arreglo, el resultado no
   depende de los barajados anteriores.
2. **Probabilidad constante:** en cada intento, la probabilidad de éxito
   (que el arreglo quede ordenado) es siempre `p = 1/n!`.
3. **Hasta el primer éxito:** el algoritmo se detiene justo cuando ocurre
   el primer éxito.

Por definición, el número de intentos hasta el primer éxito bajo estas
condiciones sigue una **distribución geométrica**, con valor esperado:
E[I] = 1/p = n!

Para `n = 10`, esto equivale a un esperado de `3,628,800` barajadas antes de
ordenar el arreglo — lo que confirma que Bogo-sort es, en la práctica,
completamente inviable para cualquier tamaño relevante de entrada.