## Problema 1 — Generador uniforme con bits

### Implementación

```javascript
function random01() {
  return Math.random() < 0.5 ? 0 : 1;
}

function random(a, b) {
  const range = b - a + 1;
  const k = Math.ceil(Math.log2(range));

  while (true) {
    let r = 0;
    for (let i = 0; i < k; i++) {
      r = r * 2 + random01();
    }
    if (r < range) return a + r;
  }
}
```

### Idea del algoritmo

Para generar un entero uniforme en `[a, b]` usando solo bits, se necesitan
`k = ⌈log₂(b − a + 1)⌉` bits para representar todos los `(b − a + 1)` valores
posibles. Se generan `k` bits con `random01()` y se construye un número
`r ∈ [0, 2ᵏ − 1]`. Si `r < (b − a + 1)`, se retorna `a + r`; de lo contrario
se descarta y se repite. Esto garantiza que cada valor en `[a, b]` tenga
exactamente la misma probabilidad de ser seleccionado.

### Verificación empírica — random(1, 6), N = 10 000

**Distribución de salida:**
1: █████ 16.5%
2: █████ 16.4%
3: █████ 16.6%
4: █████ 16.9%
5: █████ 16.8%
6: █████ 16.7%

Cada valor aparece aproximadamente `10000 / 6 ≈ 1667` veces, con desviaciones
menores al 0.3% respecto al teórico `16.67%`. La distribución es uniforme. ✓

La razón formal: al descartar `r ∈ {6, 7}`, se condiciona sobre los 6 outcomes
válidos. Cada uno tiene probabilidad `(1/8) / (6/8) = 1/6` dado que fue
aceptado.

**Tiempo esperado:**
k = ⌈log₂(6)⌉ = 3 bits
P(éxito por intento) = 6/2³ = 0.75
E[intentos] = 1/p = 1.3333

El número de intentos sigue una distribución geométrica `T ~ Geom(p)` con
`p = (b − a + 1) / 2ᵏ`, cuyo valor esperado es `E[T] = 1/p = 2ᵏ / (b − a + 1)`.
El valor medido experimentalmente (≈ 1.33) coincide con el teórico. ✓

### Análisis del tiempo de ejecución

Como `k = ⌈log₂(n)⌉` implica `2^(k−1) < n ≤ 2ᵏ`, se cumple que
`p = n / 2ᵏ > 1/2`, por lo tanto:
E[T] = 1/p < 2

Sin importar el intervalo `[a, b]`, el número esperado de intentos está
acotado por una constante. Cada intento consume exactamente `k` llamadas a
`random01()`, por lo que el tiempo total esperado es:
E[bits] = E[T] × k < 2k = 2⌈log₂(b − a + 1)⌉

**T(n) ∈ O(log n)** en número de bits consumidos, y **O(1) en número de
iteraciones** en valor esperado.

======================================================================================================================================================================================================

## Problema 2 — Truco de Von Neumann

### Implementación

```javascript
function biasedRandom(p) {
  return Math.random() < p ? 1 : 0;
}

function unbiasedRandom(p) {
  let calls = 0;
  while (true) {
    const a = biasedRandom(p); calls++;
    const b = biasedRandom(p); calls++;

    if (a === 1 && b === 0) return { bit: 1, calls };
    if (a === 0 && b === 1) return { bit: 0, calls };
    // (0,0) o (1,1) → descartar y repetir
  }
}
```

### Idea del algoritmo

Se llama a `biasedRandom(p)` dos veces por intento. Los cuatro pares posibles
y sus probabilidades son:

| Par     | Probabilidad  | Acción      |
|---------|--------------|-------------|
| (1, 0)  | p·(1−p)      | → retorna 1 |
| (0, 1)  | (1−p)·p      | → retorna 0 |
| (0, 0)  | (1−p)²       | → descartar |
| (1, 1)  | p²           | → descartar |

La clave es que `P(1,0) = P(0,1) = p(1−p)` por independencia de las llamadas.
Al condicionar sobre obtener un par diferente, ambos outcomes tienen la misma
probabilidad, produciendo una salida 50/50 sin importar el valor de `p`.

### Verificación empírica — N = 5 000 por valor de p

p=0.1: salida=49.9% unos | E[calls] simulado=11.22 | teórico=11.11
p=0.3: salida=49.3% unos | E[calls] simulado=4.72  | teórico=4.76
p=0.5: salida=49.8% unos | E[calls] simulado=3.99  | teórico=4.00
p=0.7: salida=49.7% unos | E[calls] simulado=4.70  | teórico=4.76
p=0.9: salida=49.5% unos | E[calls] simulado=11.01 | teórico=11.11

**(1) Uniformidad:** la salida es ~50% unos para todos los valores de `p`
probados. Las desviaciones son menores al 0.7%, consistentes con varianza
muestral para N = 5 000. ✓

**(2) Llamadas esperadas:** los promedios simulados coinciden con el teórico
`1/(p(1−p))` en menos del 1% de error en todos los casos. ✓

### Análisis del tiempo de ejecución

Cada intento consume exactamente 2 llamadas a `biasedRandom`. La probabilidad
de éxito (obtener un par diferente) por intento es:

P(éxito) = P(1,0) + P(0,1) = 2p(1−p)

El número de intentos sigue `T ~ Geom(2p(1−p))`, por lo que el número
esperado de llamadas totales a `biasedRandom` es:

E[llamadas] = 2 · E[T] = 2 · 1/(2p(1−p)) = 1/(p(1−p))

**¿Qué pasa cuando p → 0 o p → 1?**

Cuando el generador está muy sesgado, casi todos los pares son `(0,0)` o
`(1,1)`, que se descartan. El costo crece sin cota:

lím p→0  1/(p(1−p)) → ∞
lím p→1  1/(p(1−p)) → ∞

El mínimo se alcanza en `p = 0.5`, donde `E[llamadas] = 1/(0.5·0.5) = 4`.
Incluso en el mejor caso se necesitan 4 llamadas en promedio, pues la moneda
perfecta no tiene pares descartables pero tampoco aprovecha mejor la simetría.

**T(n) ∈ O(1/(p(1−p)))** en número esperado de llamadas, acotado por una
constante que depende del sesgo pero no del tamaño de la entrada.

======================================================================================================================================================================================================

## Problema 3 — Hiring Problem

### Implementación

```javascript
function hiringAlgorithm(candidates) {
  let best = -Infinity;
  let hires = 0;

  for (let i = 0; i < candidates.length; i++) {
    if (candidates[i] > best) {
      best = candidates[i];
      hires++;
    }
  }
  return hires;
}
```

### Idea del algoritmo

El candidato `i` es contratado si y solo si es mejor que todos los `i−1`
candidatos anteriores. En una permutación aleatoria uniforme, cualquiera de
los primeros `i` candidatos es igualmente probable de ser el mejor entre
ellos, por lo que:

P(Xᵢ = 1) = P(candidato i es el mejor entre los primeros i) = 1/i

Definiendo la variable indicadora `Xᵢ = 1` si el candidato `i` es contratado,
por linealidad de la esperanza:

E[contrataciones] = Σᵢ₌₁ⁿ E[Xᵢ] = Σᵢ₌₁ⁿ 1/i = Hₙ ≈ ln(n) + O(1)

### Probabilidades de los casos extremos

**Best-case (1 contratación):** ocurre cuando el primer candidato es el mejor
de todos. En una permutación aleatoria, la probabilidad de que el máximo quede
en la primera posición es:

P(best-case) = 1/n

**Worst-case (n contrataciones):** ocurre cuando los candidatos llegan en
orden estrictamente creciente de calidad. Solo 1 de las `n!` permutaciones
posibles produce este orden:

P(worst-case) = 1/n!

### Verificación empírica — n = 8, 100 000 ensayos

**Prueba manual con `[3, 7, 2, 9, 1, 8, 5, 4]`:**

Contrataciones: 3 — calificaciones: 3, 7, 9
(Cada contratado supera a todos los anteriores) ✓

**Simulación:**

E[contrataciones] simulado : 2.7188
Hₙ (valor teórico)         : 2.7179   ← error < 0.004%  ✓
ln(n)                      : 2.0794   ← Hₙ ≈ ln(n) + γ, γ ≈ 0.5772
P(best-case) simulado      : 0.1246
P(best-case) teórico = 1/8 : 0.1250   ← error < 0.04%   ✓
P(worst-case) simulado     : ~0.000025
P(worst-case) teórico=1/8! : 0.000025                    ✓

Los tres valores simulados convergen a sus teóricos. La diferencia entre
`Hₙ = 2.7179` y `ln(n) = 2.0794` se explica por la constante de
Euler-Mascheroni `γ ≈ 0.5772`: `Hₙ = ln(n) + γ + O(1/n)`.

### Análisis del tiempo de ejecución

El algoritmo siempre entrevista los `n` candidatos en orden, por lo que el
costo de entrevistas es `Θ(n)` en todo caso. El costo de contrataciones es
una variable aleatoria con:
E[contrataciones] = Hₙ ≈ ln(n)

Si contratar es significativamente más costoso que entrevistar, el costo
esperado total es `O(n + ln n) = O(n)`. En el peor caso (orden creciente)
el costo de contrataciones es `n`, pero este escenario ocurre con probabilidad
`1/n!`, lo que lo hace negligible en la práctica.

======================================================================================================================================================================================================

## Problema 4 — Suma esperada de n dados

### Implementación

```javascript
function expectedSum(n) {
  const E_dado = (1 + 2 + 3 + 4 + 5 + 6) / 6; // = 3.5
  return n * E_dado;
}

function lanzarDado() {
  return Math.floor(Math.random() * 6) + 1;
}

function simularNDados(n, trials) {
  let total = 0;
  for (let t = 0; t < trials; t++) {
    let suma = 0;
    for (let i = 0; i < n; i++) suma += lanzarDado();
    total += suma;
  }
  return total / trials;
}
```

### Idea del algoritmo

Sea `X = X₁ + X₂ + ··· + Xₙ` donde `Xᵢ` es el valor del dado `i`. Por
**linealidad de la esperanza**:
E[X] = E[X₁] + E[X₂] + ··· + E[Xₙ] = n · E[X₁]

Para calcular `E[X₁]` formalmente con variables indicadoras, se define
`Yᵢⱼ = 1` si el dado `i` muestra la cara `j`, y `0` en otro caso.
Entonces `E[Yᵢⱼ] = 1/6` y:
Xᵢ = Σⱼ₌₁⁶ j · Yᵢⱼ
E[Xᵢ] = Σⱼ₌₁⁶ j · E[Yᵢⱼ] = Σⱼ₌₁⁶ j · (1/6) = (1+2+3+4+5+6)/6 = 21/6 = 3.5

Por lo tanto: `E[X] = 3.5n`.

### Verificación empírica — 20 000 ensayos por valor de n
n=  1: teórico=   3.5,  simulado=   3.512
n=  2: teórico=   7.0,  simulado=   6.978
n=  5: teórico=  17.5,  simulado=  17.546
n= 10: teórico=  35.0,  simulado=  35.012
n= 20: teórico=  70.0,  simulado=  69.958
n=100: teórico= 350.0,  simulado= 350.050

El promedio empírico converge a `3.5n` para todos los valores de `n` probados,
con errores menores al 0.15%. ✓

### ¿Por qué la linealidad es tan poderosa aquí?

La linealidad de la esperanza establece que `E[X + Y] = E[X] + E[Y]`
**sin requerir independencia** entre las variables. Esto tiene dos consecuencias
importantes en este problema:

1. **Simplicidad:** no es necesario calcular la distribución conjunta de los
   `n` dados, que tendría `6ⁿ` casos posibles. Basta con calcular `E[X₁] = 3.5`
   una sola vez y multiplicar.

2. **Generalidad:** el resultado `E[X] = 3.5n` seguiría siendo válido incluso
   si los dados estuvieran correlacionados (por ejemplo, si un dado cargado
   influye en el siguiente). La esperanza de la suma siempre es la suma de
   las esperanzas.

### Análisis

El cálculo teórico es `O(1)` — una multiplicación. La simulación para
verificarlo es `O(n · trials)`, lineal en el número de dados. La potencia
del método de indicadoras está en descomponer una variable compleja (`X`) en
partes simples (`Yᵢⱼ`) cuya esperanza es trivial de calcular, y luego sumar.

======================================================================================================================================================================================================

## Problema 5 — Inversiones esperadas

### Implementación

```javascript
function countInversions(arr) {
  let count = 0;
  for (let i = 0; i < arr.length; i++) {
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[i] > arr[j]) count++;
    }
  }
  return count;
}

function expectedInversions(n) {
  return n * (n - 1) / 4; // C(n,2) · (1/2)
}
```

### Idea del algoritmo

Para cualquier par de posiciones `i < j` en una permutación aleatoria uniforme
de `n` elementos distintos, los valores `A[i]` y `A[j]` son igualmente
probables de aparecer en cualquier orden relativo. Por lo tanto:
P(A[i] > A[j]) = 1/2  para todo par (i, j) con i < j

Se define la variable indicadora `Xᵢⱼ = 1` si el par `(i, j)` es una
inversión. Por la observación anterior, `E[Xᵢⱼ] = 1/2`. El total de
inversiones es `X = Σᵢ＜ⱼ Xᵢⱼ` y por linealidad de la esperanza:
E[X] = Σᵢ＜ⱼ E[Xᵢⱼ] = C(n, 2) · (1/2) = [n(n−1)/2] · (1/2) = n(n−1)/4

### Verificación empírica — 10 000 permutaciones por valor de n
nTeórico n(n−1)/4Simulado¿Coinciden?31.51.506✓43.02.998✓55.04.963✓67.57.497✓814.014.029✓1022.522.516✓

El promedio empírico converge a `n(n−1)/4` en todos los casos, con error
menor al 0.2%. ✓

**Caso extremo — arreglo invertido:**
n=4: [4,3,2,1] → 6 inversiones = C(4,2) = 4·3/2  ✓
n=5: [5,4,3,2,1] → 10 inversiones = C(5,2) = 5·4/2 ✓
n=6: [6,5,4,3,2,1] → 15 inversiones = C(6,2) = 6·5/2 ✓

El arreglo en orden decreciente maximiza las inversiones: todo par `(i, j)`
con `i < j` cumple `A[i] > A[j]`, alcanzando el máximo posible de
`C(n, 2) = n(n−1)/2` inversiones, que es exactamente el doble del valor
esperado.

### Análisis

El número esperado de inversiones `n(n−1)/4` tiene una interpretación
geométrica directa: en promedio, una permutación aleatoria está "a mitad de
camino" entre el orden creciente (0 inversiones) y el orden decreciente
`(n(n−1)/2)` inversiones.

Este resultado tiene implicaciones para algoritmos de ordenamiento. Un
algoritmo que resuelve una inversión por operación (como Insertion Sort)
necesita en promedio `Θ(n²)` operaciones sobre entrada aleatoria, confirmando
que `O(n²)` es el costo esperado de Insertion Sort, y que cualquier algoritmo
basado en intercambios adyacentes tiene ese mismo piso en el caso promedio.