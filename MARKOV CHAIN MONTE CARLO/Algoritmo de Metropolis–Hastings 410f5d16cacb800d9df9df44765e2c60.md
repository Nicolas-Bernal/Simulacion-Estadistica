# Algoritmo de Metropolis–Hastings

## Cadenas de Markov Monte Carlo (MCMC)

El algoritmo de **Metropolis–Hastings (MH)** genera una cadena de Markov cuya distribución estacionaria es una distribución objetivo $\pi(x)$. Permite obtener muestras cuando calcular directamente esa distribución o sus esperanzas es difícil. La distribución objetivo puede conocerse solo hasta una constante de normalización.

## 1. Notación y algoritmo

- $x$: estado actual de la cadena.
- $y$: estado candidato.
- $\pi(x)$: densidad o función de masa objetivo.
- $q(y\mid x)$: distribución de propuesta; describe la probabilidad o densidad de proponer $y$ desde $x$.

Partiendo de un estado inicial $X^{(0)}=x^{(0)}$ tal que $\pi(x^{(0)})>0$, en cada iteración $t$:

1. **Proponer:** generar $Y\sim q(\cdot\mid X^{(t)})$.
2. **Calcular la razón de aceptación:**

$$
r(X^{(t)},Y)=\frac{\pi(Y)\,q(X^{(t)}\mid Y)}{\pi(X^{(t)})\,q(Y\mid X^{(t)})}.
$$

1. **Calcular la probabilidad de aceptación:**

$$
\alpha(X^{(t)},Y)=\min\{1,r(X^{(t)},Y)\}.
$$

1. **Aceptar o rechazar:** generar $U\sim\text{Uniforme}(0,1)$. Si $U\leq\alpha(X^{(t)},Y)$, establecer $X^{(t+1)}=Y$; de lo contrario, conservar el estado actual: $X^{(t+1)}=X^{(t)}$.

Si $\pi$ solo se conoce salvo una constante, puede usarse cualquier función $\tilde{\pi}(x)\propto\pi(x)$ en la razón: la constante común se cancela. Si la propuesta inversa tiene probabilidad o densidad cero, la propuesta se rechaza ($\alpha=0$).

### Caso de propuesta simétrica

Si $q(y\mid x)=q(x\mid y)$, los términos de propuesta se cancelan y queda:

$$
r(x,y)=\frac{\pi(y)}{\pi(x)},\qquad \alpha(x,y)=\min\left[1,\frac{\pi(y)}{\pi(x)}\right].
$$

Este caso particular se conoce como **algoritmo de Metropolis**.

## 2. Distribución estacionaria y convergencia

La aceptación de Metropolis–Hastings satisface el **balance detallado**. Para dos estados distintos $x$ e $y$:

$$
\pi(x)q(y\mid x)\alpha(x,y)
=\min\{\pi(x)q(y\mid x),\,\pi(y)q(x\mid y)\}
=\pi(y)q(x\mid y)\alpha(y,x).
$$

El balance detallado implica que $\pi$ es una distribución estacionaria de la cadena, siempre que la cadena esté bien definida en el soporte objetivo. La masa de probabilidad de permanecer en el mismo estado por rechazo completa la transición; no debe omitirse.

**Tener una distribución estacionaria no garantiza por sí solo que la cadena converja a ella desde cualquier estado inicial.** También se requieren condiciones de conectividad y recurrencia adecuadas:

- En un espacio **finito**, irreducibilidad garantiza una única distribución estacionaria y, junto con aperiodicidad, la convergencia hacia ella.
- En un espacio **numerable**, se requieren condiciones como irreducibilidad y recurrencia positiva para la existencia y unicidad de la distribución estacionaria; la aperiodicidad permite la convergencia desde los estados iniciales pertinentes.
- En espacios **continuos o generales**, deben verificarse condiciones apropiadas de irreducibilidad y recurrencia (por ejemplo, recurrencia de Harris). No es correcto afirmar que irreducibilidad y aperiodicidad, sin más condiciones, garantizan siempre la convergencia.

En la práctica, la propuesta debe permitir explorar las regiones relevantes del soporte objetivo. Una propuesta que no conecta esas regiones puede producir una cadena reducible y muestras que no representen toda la distribución objetivo.

## 3. Estimación con las muestras

Sean $X^{(0)},X^{(1)},\ldots,X^{(N)}$ los estados generados. Tras descartar, si corresponde, los primeros $B$ estados, se utilizan $M=N-B$ muestras: $X^{(B+1)},\ldots,X^{(N)}$.

Para una función $h$, la esperanza se estima mediante:

$$
\widehat{\mathbb{E}}[h(X)]=\frac{1}{M}\sum_{t=B+1}^{N}h\left(X^{(t)}\right).
$$

Otras cantidades comunes son:

| Cantidad | Estimador |
| --- | --- |
| Varianza de $h(X)$ | $\displaystyle \frac{1}{M-1}\sum_{t=B+1}^{N}\left(h(X^{(t)})-\widehat{\mathbb{E}}[h(X)]\right)^2$ |
| Probabilidad de cola $\Pr(h(X)\leq c)$ | $\displaystyle \frac{1}{M}\sum_{t=B+1}^{N}\mathbf{1}\{h(X^{(t)})\leq c\}$ |

Bajo condiciones ergódicas apropiadas y si $\mathbb{E}_{\pi}[|h(X)|]<\infty$, el promedio muestral converge a la esperanza objetivo. Como las muestras de una cadena suelen estar autocorrelacionadas, su error Monte Carlo no debe calcularse como si fueran observaciones independientes.

> **Conserva los estados repetidos.** Cuando una propuesta se rechaza, la cadena permanece en el mismo estado. Esas repeticiones representan correctamente la probabilidad de rechazo y forman parte de la muestra. Eliminarlas altera la distribución representada por la cadena. El *thinning* tampoco es necesario por defecto; solo se justifica si existe una razón práctica concreta.
> 

## 4. Diagnóstico y prácticas recomendadas

### Calentamiento (*burn-in*)

Descartar un tramo inicial puede reducir la influencia del punto de partida, pero no existe un número universal de iteraciones que garantice haber alcanzado el equilibrio. El *burn-in* no sustituye los diagnósticos ni corrige una cadena que no explora el soporte.

### Cadenas múltiples y diagnóstico

Cuando sea posible, ejecutar varias cadenas desde puntos iniciales dispersos. Revisar gráficas de trazas, autocorrelación y diagnósticos entre cadenas (por ejemplo, $\hat{R}$); también conviene estimar el tamaño de muestra efectivo (ESS). Ningún diagnóstico aislado demuestra por sí solo que la cadena haya convergido.

### Selección de la propuesta

- Una propuesta **demasiado estrecha** suele aceptar con frecuencia, pero avanza poco y puede tener alta autocorrelación.
- Una propuesta **demasiado dispersa** puede generar muchos rechazos y largos periodos sin movimiento.
- La tasa de aceptación es un indicador útil, pero no existe una tasa óptima universal: depende de la dimensión, la distribución objetivo y el tipo de propuesta. Una tasa alta, por sí sola, no implica buena exploración.

## 5. Lista de verificación

- [ ]  La distribución objetivo es válida y el estado inicial pertenece a su soporte.
- [ ]  La razón incluye correctamente la propuesta inversa $q(x\mid y)$; solo se cancela si la propuesta es simétrica.
- [ ]  La propuesta permite recorrer las regiones relevantes del soporte objetivo.
- [ ]  Los estados repetidos por rechazo se conservan en los cálculos.
- [ ]  Se evalúan autocorrelación, tamaño de muestra efectivo y convergencia entre cadenas, según corresponda.
- [ ]  La incertidumbre de las estimaciones tiene en cuenta la dependencia entre muestras.

## Referencias

- Metropolis, N., Rosenbluth, A. W., Rosenbluth, M. N., Teller, A. H., & Teller, E. (1953). [Equation of State Calculations by Fast Computing Machines](https://doi.org/10.1063/1.1699114). *The Journal of Chemical Physics, 21*(6), 1087–1092.
- Hastings, W. K. (1970). [Monte Carlo sampling methods using Markov chains and their applications](https://doi.org/10.1093/biomet/57.1.97). *Biometrika, 57*(1), 97–109.
- GitHub Docs. [Writing mathematical expressions](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/writing-mathematical-expressions).

[Independence Chains](Algoritmo%20de%20Metropolis%E2%80%93Hastings/Independence%20Chains%20410f5d16cacb805ea853d2100578da9c.md)