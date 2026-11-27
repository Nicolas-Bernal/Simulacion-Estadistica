# Random Walk MCMC

# 🎲 Caminata Aleatoria de Monte Carlo (Random Walk MCMC)

> **Fuente principal:** *Computational Statistics* (2da Edición) — Geof H. Givens y Jennifer A. Hoeting (Capítulo 7: *Markov Chain Monte Carlo*)[cite: 1].
> 

---

## 📌 1. Introducción y Contexto

En estadística computacional, el objetivo principal de los métodos de **Cadenas de Markov Monte Carlo (MCMC)** es simular observaciones de una distribución objetivo compleja $f(x)$ (usualmente una distribución posterior bayesiana) de la cual no es posible muestrear de forma directa ni analítica.

El algoritmo de **Metropolis-Hastings** construye una cadena de Markov $X^{(0)}, X^{(1)}, X^{(2)}, \dots$ cuya distribución estacionaria es precisamente $f(x)$. La **Caminata Aleatoria de Monte Carlo** (o *Random Walk Metropolis-Hastings*) es una de las variantes más utilizadas y fundamentales de este algoritmo.

---

## ⚙️ 2. Definición Matemática del Modelo

En un muestreador de Metropolis-Hastings de caminata aleatoria, el estado candidato $X^*$ se propone perturbando el estado actual $x^{(t)}$ mediante un incremento aleatorio $\epsilon$:

$$
X^* = x^{(t)} + \epsilon
$$

donde $\epsilon \sim g(\cdot)$ es una variable aleatoria independiente del estado actual $x^{(t)}$, con densidad $g(\cdot)$ definida sobre el soporte apropiado (por ejemplo, una distribución normal centrada en cero o una uniforme simétrica).

### Densidad de Propuesta

Bajo este esquema, la densidad de transición o propuesta $g(x^* \mid x^{(t)})$ depende únicamente de la diferencia entre el candidato y el estado actual:

$$
g(x^* \mid x^{(t)}) = g(x^* - x^{(t)})
$$

### Caso Simétrico (Algoritmo de Metropolis)

Si la densidad de los incrementos es simétrica alrededor de cero, es decir, $g(\epsilon) = g(-\epsilon)$, entonces:

$$
g(x^* \mid x^{(t)}) = g(x^{(t)} \mid x^*)
$$

En este escenario (el caso más común en la práctica, usando $\epsilon \sim \mathcal{N}(0, \sigma^2)$ o $\epsilon \sim \text{Uniforme}(-\delta, \delta)$), los términos de la densidad de propuesta se cancelan en la razón de Metropolis-Hastings.

---

## 🔄 3. Algoritmo Paso a Paso

Para generar una muestra de la distribución objetivo $f(x)$ (conocida al menos hasta una constante de proporcionalidad):

1. **Inicialización:** Elegir un punto de partida arbitrario $x^{(0)}$ dentro del soporte de $f$ (donde $f(x^{(0)}) > 0$) y fijar $t = 0$.
2. **Propuesta:** Generar un incremento aleatorio $\epsilon \sim g(\cdot)$ y calcular el candidato:
    
    $$
    x^* = x^{(t)} + \epsilon
    $$
    
3. **Cálculo de la Razón de Aceptación ($R$):**
    - *Caso general:*
        
        $$
        R(x^{(t)}, x^*) = \frac{f(x^*) \, g(x^{(t)} - x^*)}{f(x^{(t)}) \, g(x^* - x^{(t)})}
        $$
        
    - *Caso simétrico ($g(\epsilon) = g(-\epsilon)$):*
        
        $$
        R(x^{(t)}, x^*) = \frac{f(x^*)}{f(x^{(t)})}
        $$
        
4. **Decisión (Aceptación / Rechazo):** Generar $U \sim \text{Uniforme}(0, 1)$:
    - Si $U \le \min\{1, R(x^{(t)}, x^*)\}$, aceptar el candidato: $x^{(t+1)} = x^*$.
    - En caso contrario, rechazar el candidato y mantener el estado actual: $x^{(t+1)} = x^{(t)}$.
5. **Iteración:** Incrementar $t = t + 1$ y volver al paso 2 hasta alcanzar el tamaño de muestra deseado $N$.

---

## 🎯 4. El Problema del Ajuste (*Tuning*) y Escala de Propuesta

Givens y Hoeting destacan que el éxito práctico de la caminata aleatoria depende críticamente de la **varianza (escala) de la distribución de los incrementos** $\epsilon$ (por ejemplo, el parámetro $\sigma$ en $\epsilon \sim \mathcal{N}(0, \sigma^2)$):

| Tamaño del paso ($\sigma$) | Comportamiento de la Cadena | Tasa de Aceptación | Consecuencia |
| --- | --- | --- | --- |
| **Muy pequeño** ($\sigma \to 0$) | Movimientos diminutos alrededor de $x^{(t)}$. | Muy alta ($\approx 90\% - 99\%$) | **Mezcla lenta (*slow mixing*):** La cadena tarda muchísimo en explorar todo el soporte de $f$. Alta autocorrelación. |
| **Muy grande** ($\sigma \to \infty$) | Propuestas caen frecuentemente en las colas de $f$ donde $f(x^*) \approx 0$. | Muy baja ($\approx 1\% - 5\%$) | **Cadena estancada (*sticky chain*):** Se rechazan casi todos los movimientos y se repite el mismo valor muchas veces. |
| **Óptimo** (Balanceado) | Exploración eficiente de las zonas de alta probabilidad y colas. | Moderada ($\approx 23\% - 50\%$) | **Convergencia rápida:** Baja autocorrelación y buena estimación de momentos. |

### Reglas Empíricas de Tasa de Aceptación (Roberts, Gelman y Gilks)

Según la literatura teórica citada en el texto para propuestas normales multivariadas en objetivos normales:

- **1 dimensión ($d = 1$):** La tasa de aceptación óptima teórica es aproximadamente **44%** ($0.44$).
- **Dimensiones altas ($d \to \infty$):** La tasa de aceptación óptima converge a **23.4%** ($0.234$).

---