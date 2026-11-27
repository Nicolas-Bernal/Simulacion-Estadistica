# Gibbs Sampling

# 🔄 Muestreador de Gibbs (Gibbs Sampling)

> **Fuente principal:** *Computational Statistics* (2da Edición) — Geof H. Givens y Jennifer A. Hoeting (Capítulo 7, Sección 7.2: *Gibbs Sampling*).
> 

---

## 📌 1. Introducción y Contexto

Cuando se trabaja con distribuciones objetivo multivariadas $\mathbf{X} = (X_1, X_2, \dots, X_p)$ de alta dimensión, diseñar una densidad de propuesta conjunta $g(\mathbf{x}^* \mid \mathbf{x}^{(t)})$ eficiente para Metropolis-Hastings suele ser extremadamente difícil.

El **Muestreador de Gibbs** resuelve este problema dividiendo el vector multivariado en componentes (o bloques) univariados y actualizando **una variable a la vez**, extrayendo muestras directamente de su **distribución condicional completa** (*full conditional distribution*), dado el valor más reciente de todas las demás variables.

### Relación con Metropolis-Hastings

Como destacan Givens y Hoeting, el muestreador de Gibbs es un **caso especial de Metropolis-Hastings por bloques** donde:

1. La densidad de propuesta para la $i$ésima componente es exactamente su distribución condicional completa $f(x_i \mid \mathbf{x}_{-i})$.
2. La razón de aceptación resultante es **siempre igual a $1$** ($R = 1$), lo que significa que **todas las propuestas son aceptadas automáticamente** (tasa de aceptación del 100%).

---

## ⚙️ 2. Definición Matemática del Modelo

Sea $f(x_1, x_2, \dots, x_p)$ la densidad conjunta objetivo de interés. Para cada componente $i \in \{1, \dots, p\}$, denotamos por $\mathbf{x}_{-i}$ al vector de todas las variables excepto la $i$-ésima:

$$
\mathbf{x}_{-i} = (x_1, \dots, x_{i-1}, x_{i+1}, \dots, x_p)
$$

La **distribución condicional completa** de $X_i$ dado el resto de las variables se define como:

$$
f(x_i \mid \mathbf{x}_{-i}) = \frac{f(x_1, \dots, x_p)}{\int f(x_1, \dots, x_p) \, dx_i} \propto f(x_1, \dots, x_p)
$$

En la práctica (especialmente en inferencia bayesiana con distribuciones previas conjugadas), $f(x_i \mid \mathbf{x}_{-i})$ suele tener una forma analítica estándar (Normal, Gamma, Beta, Binomial, etc.) de la cual es muy fácil simular directamente, sin necesidad de conocer la constante de normalización de la densidad conjunta.

### Demostración de que la Razón de Aceptación es $R = 1$

Al actualizar solo la componente $i$ desde $x_i^{(t)}$ hacia $x_i^*$ manteniendo fijo $\mathbf{x}_{-i}$, la propuesta es $g(x_i^* \mid x_i^{(t)}, \mathbf{x}_{-i}) = f(x_i^* \mid \mathbf{x}_{-i})$. Sustituyendo en la razón de Metropolis-Hastings:

$$
R = \frac{f(x_i^*, \mathbf{x}_{-i}) \, f(x_i^{(t)} \mid \mathbf{x}_{-i})}{f(x_i^{(t)}, \mathbf{x}_{-i}) \, f(x_i^* \mid \mathbf{x}_{-i})} = \frac{\big[f(x_i^* \mid \mathbf{x}_{-i}) \, f(\mathbf{x}_{-i})\big] \, f(x_i^{(t)} \mid \mathbf{x}_{-i})}{\big[f(x_i^{(t)} \mid \mathbf{x}_{-i}) \, f(\mathbf{x}_{-i})\big] \, f(x_i^* \mid \mathbf{x}_{-i})} = 1
$$

---

## 🔄 3. Algoritmo Paso a Paso (Esquema Estándar / Barrido Sistemático)

Para generar una muestra de la distribución conjunta $f(x_1, \dots, x_p)$:

1. **Inicialización:** Elegir valores iniciales $\mathbf{x}^{(0)} = (x_1^{(0)}, x_2^{(0)}, \dots, x_p^{(0)})$ dentro del soporte de $f$ y fijar $t = 0$.
2. **Actualización secuencial (Iteración $t \to t+1$):** Generar cada componente condicionado a los **valores más recientes** disponibles de las demás componentes:
    - Extraer $x_1^{(t+1)} \sim f\left(x_1 \;\middle|\; x_2^{(t)}, x_3^{(t)}, \dots, x_p^{(t)}\right)$
    - Extraer $x_2^{(t+1)} \sim f\left(x_2 \;\middle|\; x_1^{(t+1)}, x_3^{(t)}, \dots, x_p^{(t)}\right)$
    - $\vdots$
    - Extraer $x_i^{(t+1)} \sim f\left(x_i \;\middle|\; x_1^{(t+1)}, \dots, x_{i-1}^{(t+1)}, x_{i+1}^{(t)}, \dots, x_p^{(t)}\right)$
    - $\vdots$
    - Extraer $x_p^{(t+1)} \sim f\left(x_p \;\middle|\; x_1^{(t+1)}, x_2^{(t+1)}, \dots, x_{p-1}^{(t+1)}\right)$
3. **Iteración:** Incrementar $t = t + 1$ y repetir el paso 2 hasta obtener $N$ vectores completos $\mathbf{x}^{(1)}, \dots, \mathbf{x}^{(N)}$.

*(Nota del libro: También existen variantes como el **barrido aleatorio** [random scan], donde en cada paso se elige al azar qué coordenada $i$ actualizar, o el **Gibbs por bloques** [blocked Gibbs], donde se actualizan subconjuntos de variables correlacionadas simultáneamente).*

---

## ⚖️ 4. Comparación de Métodos MCMC (Capítulo 7)

| Característica | Caminata Aleatoria | Cadena Independiente | Muestreador de Gibbs |
| --- | --- | --- | --- |
| **Dimensión de actualización** | Todo el vector $\mathbf{x}$ (o por bloques). | Todo el vector $\mathbf{x}$ simultáneamente. | Una variable (o bloque) a la vez, secuencialmente. |
| **Propuesta utilizada** | Local: $\mathbf{x}^* = \mathbf{x}^{(t)} + \boldsymbol{\epsilon}$. | Global fija: $\mathbf{x}^* \sim g(\cdot)$. | Condicional completa: $x_i^* \sim f(x_i \mid \mathbf{x}_{-i})$. |
| **Tasa de aceptación** | Requiere ajuste ($\approx 23\% - 44\%$). | Depende del ajuste global de $g(\cdot)$ a $f(\cdot)$. | **Siempre 100%** (no hay paso de rechazo). |
| **Parámetros de ajuste (*Tuning*)** | Sí (matriz de covarianza de $\boldsymbol{\epsilon}$). | Sí (elección de $g$). | **Ninguno** (totalmente automático si las condicionales son conocidas). |
| **Principal debilidad** | Mezcla lenta en altas dimensiones. | Falla si $g$ tiene colas ligeras o en muchas dimensiones. | Mezcla muy lenta si las variables están **fuertemente correlacionadas**. |

---