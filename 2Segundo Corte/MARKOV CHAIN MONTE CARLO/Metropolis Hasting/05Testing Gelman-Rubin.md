# Testing Gelman-Rubin

# 📊 Diagnóstico de Convergencia de Gelman-Rubin ($\hat{R}$)

> **Fuente principal:** *Computational Statistics* (2da Edición) — Geof H. Givens y Jennifer A. Hoeting (Capítulo 7, Sección 7.3: *Convergence Diagnostics*).
> 

---

## 📌 1. Introducción y Contexto

Al ejecutar un algoritmo MCMC (como Metropolis-Hastings o Gibbs), nunca sabemos con certeza absoluta en qué iteración $t$ la cadena ha olvidado su punto de partida y ha alcanzado su distribución estacionaria $f(x)$. Mirar una sola cadena puede ser engañoso: podría parecer estable simplemente porque quedó atrapada en un modo local.

Para resolver esto, Andrew Gelman y Donald Rubin (1992) propusieron un diagnóstico cuantitativo basado en ejecutar **múltiples cadenas paralelas e independientes** a partir de **puntos iniciales sobredispersos** (*overdispersed starting points*), es decir, distribuidos ampliamente a lo largo y ancho del espacio de parámetros.

La intuición central es análoga a un **Análisis de Varianza (ANOVA)**: si todas las cadenas han convergido a la misma distribución estacionaria, la variabilidad **entre** las distintas cadenas debe ser prácticamente igual a la variabilidad **dentro** de cada cadena individual.

---

## ⚙️ 2. Definición Matemática del Estadístico $\hat{R}$

Supongamos que ejecutamos **$m$ cadenas paralelas** ($m \ge 2$), cada una de longitud **$2n$**, y descartamos las primeras $n$ iteraciones como período de calentamiento (*burn-in*), conservando las últimas **$n$ iteraciones** de cada cadena. Sea $x_{ij}$ el valor del parámetro en la iteración $i \in \{1, \dots, n\}$ de la cadena $j \in \{1, \dots, m\}$.

### Paso A: Varianza Dentro de las Cadenas ($W$)

Calculamos la varianza muestral $s_j^2$ dentro de cada cadena $j$, y luego promediamos sobre las $m$ cadenas:

$$
s_j^2 = \frac{1}{n - 1} \sum_{i=1}^{n} (x_{ij} - \bar{x}_{\cdot j})^2 \qquad \Longrightarrow \qquad W = \frac{1}{m} \sum_{j=1}^{m} s_j^2
$$

donde $\bar{x}_{\cdot j} = \frac{1}{n}\sum_{i=1}^n x_{ij}$ es la media de la cadena $j$. $W$ suele subestimar la varianza real de la distribución objetivo antes de la convergencia porque cada cadena aún no ha explorado todo el soporte.

### Paso B: Varianza Entre Cadenas ($B$)

Calculamos la varianza de las medias de cada cadena $\bar{x}_{\cdot j}$ alrededor de la gran media global $\bar{x}_{\cdot\cdot} = \frac{1}{m}\sum_{j=1}^m \bar{x}_{\cdot j}$:

$$
B = \frac{n}{m - 1} \sum_{j=1}^{m} (\bar{x}_{\cdot j} - \bar{x}_{\cdot\cdot})^2
$$

### Paso C: Varianza Marginal Posterior Estimada ($\widehat{\text{Var}}^+$)

Combinamos ambas fuentes de variación mediante una suma ponderada para obtener un estimador insesgado (o ligeramente sobrestimado) de la varianza estacionaria total:

$$
\widehat{\text{Var}}^+(X) = \frac{n - 1}{n} W + \frac{1}{n} B
$$

### Paso D: Factor de Reducción de Escala Potencial ($\hat{R}$)

Finalmente, el estadístico de Gelman-Rubin —conocido como **Factor de Reducción de Escala Potencial** (*Potential Scale Reduction Factor*, PSRF)— se define como:

$$
\hat{R} = \sqrt{\frac{\widehat{\text{Var}}^+(X)}{W}}
$$

*(Nota: En la literatura más antigua o en algunas ediciones aparece sin raíz cuadrada como $R = \widehat{\text{Var}}^+/W$, pero la convención moderna estándar adoptada en R y en textos recientes utiliza la raíz cuadrada $\hat{R}$).*

---

## 📏 3. Reglas de Interpretación

Dado que antes de la convergencia $\widehat{\text{Var}}^+(X)$ sobreestima la varianza verdadera y $W$ la subestima, se cumple generalmente que $\hat{R} > 1$:

| Valor de $\hat{R}$ | Interpretación Diagnóstica | Acción Recomendada |
| --- | --- | --- |
| **$\hat{R} \gg 1$** (ej. $> 1.2$) | **No hay convergencia.** Las cadenas aún están en regiones distintas del soporte o el *burn-in* fue insuficiente. | Aumentar el número de iteraciones $n$, revisar el *tuning* o verificar multimodalidad. |
| **$1.05 < \hat{R} \le 1.2$** | **Convergencia dudosa o incompleta.** Persiste cierta diferencia entre las medias o varianzas de las cadenas. | Continuar simulando por más iteraciones hasta estabilizar. |
| **$\hat{R} \le 1.05$** (o **$< 1.01$**) | **Convergencia aceptable.** Las cadenas se han mezclado bien y exploran la misma distribución estacionaria. | Proceder con la inferencia estadística combinando las $m \times n$ muestras post-*burn-in*. |

---