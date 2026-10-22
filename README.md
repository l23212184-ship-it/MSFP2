[![Open in MATLAB Online](https://www.mathworks.com/images/responsive/global/open-in-matlab-online.svg)](https://matlab.mathworks.com/open/github/v1?repo=l23212184-ship-it/MSFP2)
# Práctica 2 — Modelado y control del sistema cardiovascular (Windkessel)

**Tecnológico Nacional de México · Instituto Tecnológico de Tijuana**  
**Departamento:** Ingeniería Eléctrica y Electrónica  
**Carrera:** Ingeniería Biomédica  
**Asignatura:** Modelado de Sistemas Fisiológicos (IBF-1020)  
**Docente:** Dr. Paul Antonio Valle Trujillo  
**Estudiante:** **Castro Alcántar Daniel Hiram**  
**Número de control:** **23212184**  
**Correo institucional:** **l23212184@tectijuana.edu.mx**  
**Modalidad:** Trabajo individual  
**Fecha:** Octubre de 2026

> **Alcance del repositorio.** La práctica fue realizada por el estudiante. Este README reúne su fundamento, el desarrollo matemático, los resultados analíticos verificables y la metodología seguida según el instructivo. El archivo de cálculos manuscritos se incluye como evidencia. Los modelos de Simulink, capturas, gráficas y ganancias PID efectivamente obtenidas deben incorporarse como archivos originales del estudiante: no se les asignan nombres de resultados ni valores numéricos no suministrados.

**Repositorio oficial de la materia:** [DrPaulValle/MSF-IBF1020](https://github.com/DrPaulValle/MSF-IBF1020)

---

## Contenido

1. [Resumen](#1-resumen)
2. [Competencia y objetivos](#2-competencia-y-objetivos)
3. [Fundamento fisiológico y modelo eléctrico](#3-fundamento-fisiológico-y-modelo-eléctrico)
4. [Variables y parámetros](#4-variables-y-parámetros)
5. [Comparación de las condiciones cardiovasculares](#5-comparación-de-las-condiciones-cardiovasculares)
6. [Modelo matemático y ecuaciones de Kirchhoff](#6-modelo-matemático-y-ecuaciones-de-kirchhoff)
7. [Función de transferencia](#7-función-de-transferencia)
8. [Representación en espacio de estados](#8-representación-en-espacio-de-estados)
9. [Análisis de estabilidad y polos](#9-análisis-de-estabilidad-y-polos)
10. [Error en estado estacionario](#10-error-en-estado-estacionario)
11. [Señal de excitación y simulaciones](#11-señal-de-excitación-y-simulaciones)
12. [Control PID y seguimiento de referencia](#12-control-pid-y-seguimiento-de-referencia)
13. [Implementación y verificación cruzada](#13-implementación-y-verificación-cruzada)
14. [Diagrama fisiológico en BioRender](#14-diagrama-fisiológico-en-biorender)
15. [Resultados y discusión](#15-resultados-y-discusión)
16. [Estructura del repositorio y evidencias](#16-estructura-del-repositorio-y-evidencias)
17. [Reproducibilidad y referencias](#17-reproducibilidad-y-referencias)

---

## 1. Resumen

Se analiza el comportamiento dinámico del sistema cardiovascular a través del **modelo Windkessel de cuatro elementos**, formado por la impedancia característica arterial Z, la inertancia sanguínea L, la distensibilidad arterial C y la resistencia vascular periférica R. Mediante una analogía con circuitos eléctricos, la presión arterial se representa como voltaje y el flujo sanguíneo como corriente.

Se comparan tres conjuntos de parámetros: **normotenso (control), hipotenso e hipertenso (casos)**. La variación entre ellos se introduce únicamente mediante la distensibilidad arterial C, manteniendo constantes Z, L y R. A partir de la topología eléctrica se desarrollan las ecuaciones dinámicas, la función de transferencia G(s) = Pₚ(s)/Pₐ(s), la representación en variables de estado y el cálculo de los polos. La práctica también contempla las implementaciones en MATLAB/Simulink, Simscape Electrical y Python, así como la comparación de sistemas en lazo abierto y lazo cerrado con controladores PID independientes.

**Palabras clave:** Windkessel; sistema cardiovascular; analogía eléctrica; impedancia; inertancia; distensibilidad; resistencia periférica; función de transferencia; estabilidad; control PID; MATLAB; Simulink; Python.

## 2. Competencia y objetivos

**Competencia:** modelar sistemas fisiológicos mediante circuitos eléctricos equivalentes, ecuaciones diferenciales y funciones de transferencia, y aplicar técnicas de control clásico para estudiar sus respuestas dinámicas.

**Objetivo general:** obtener, analizar y comparar el comportamiento de los modelos normotenso, hipotenso e hipertenso, usando como referencia la respuesta del modelo normotenso.

**Objetivos específicos:**

- Relacionar los parámetros del circuito con propiedades del sistema arterial.
- Formular y resolver las ecuaciones del modelo equivalente.
- Obtener las funciones de transferencia de los tres escenarios.
- Verificar polos, estabilidad y error estacionario ante escalón.
- Representar el modelo mediante ecuaciones de estado, función de transferencia y circuito en Simscape.
- Comparar las respuestas mediante las herramientas de simulación de la práctica.
- Analizar dos controladores PID: normotenso–hipotenso y normotenso–hipertenso.
- Documentar resultados y evidencias mediante GitHub.

## 3. Fundamento fisiológico y modelo eléctrico

El modelo Windkessel aproxima la dinámica arterial mediante propiedades concentradas. La **distensibilidad** describe el cambio de volumen de las arterias ante una variación de presión; la **resistencia periférica**, la oposición al flujo en el lecho vascular; la **inertancia**, el efecto de la masa sanguínea en aceleración; y la **impedancia característica**, los efectos dinámicos asociados con la región proximal del sistema arterial.

| Magnitud fisiológica | Analogía eléctrica |
|---|---|
| Presión arterial | Voltaje |
| Flujo sanguíneo | Corriente |
| Impedancia característica Z | Elemento resistivo equivalente |
| Inertancia arterial L | Inductor |
| Distensibilidad arterial C | Capacitor |
| Resistencia periférica R | Resistor |

**Topología de la Figura 5.1 del syllabus:** Z y L están **en paralelo entre Pₐ y Pₚ**; C y R están **en paralelo entre Pₚ y la referencia**. Esta configuración no debe cambiarse al representarla en Simscape o BioRender.

Aunque el modelo tiene cuatro elementos, es **de segundo orden** porque solo L y C almacenan energía de forma independiente. Se trata de una representación simplificada del sistema arterial y no de una reproducción exacta de cada vaso sanguíneo.

## 4. Variables y parámetros

| Símbolo | Descripción | Unidad fisiológica |
|---|---|---|
| Pₐ(t) | Presión de entrada al circuito | mmHg |
| Pₚ(t) | Presión asociada con la distensibilidad arterial; salida del modelo | mmHg |
| Fₐ(t) | Flujo de entrada | mL/s |
| F_Z(t) | Flujo de la rama Z | mL/s |
| F_L(t) | Flujo de la rama L | mL/s |
| F_C(t) | Flujo de la rama C | mL/s |
| F_R(t) | Flujo de la rama R | mL/s |
| Z, R | Impedancia característica y resistencia periférica | mmHg·s/mL |
| L | Inertancia arterial | mmHg·s²/mL |
| C | Distensibilidad arterial | mL/mmHg |

## 5. Comparación de las condiciones cardiovasculares

Los valores son los **establecidos por el profesor en el syllabus, sección 5.4.2, página 78**:

| Parámetro | Normotenso (control) | Hipotenso | Hipertenso | Unidad |
|---|---:|---:|---:|---|
| Z | 0.010 | 0.010 | 0.010 | mmHg·s/mL |
| L | 0.001 | 0.001 | 0.001 | mmHg·s²/mL |
| R | 1.000 | 1.000 | 1.000 | mmHg·s/mL |
| **C** | **1.500** | **3.500** | **1.000** | mL/mmHg |

**Interpretación del ejercicio:** en el escenario hipotenso, una mayor C representa una pared arterial más distensible; en el hipertenso, una menor C representa menor distensibilidad. Estas variaciones son decisiones del **modelo didáctico** y no explican, por sí solas, todas las causas clínicas de hipotensión o hipertensión.

## 6. Modelo matemático y ecuaciones de Kirchhoff

### 6.1. Conservación del flujo

En el nodo de entrada, el flujo se divide entre las ramas Z y L; a la salida, el flujo se reparte entre C y R:

$$
F_a(t)=F_Z(t)+F_L(t)=F_C(t)+F_R(t).
$$

### 6.2. Ecuaciones de los componentes

**Impedancia característica:**

$$
F_Z(t)=\frac{P_a(t)-P_p(t)}{Z}.
$$

**Inertancia arterial:**

$$
P_a(t)-P_p(t)=L\frac{dF_L(t)}{dt}.
$$

Bajo la hipótesis de flujo inicial nulo en L:

$$
F_L(t)=\frac{1}{L}\int_0^t\big[P_a(\tau)-P_p(\tau)\big]d\tau.
$$

**Distensibilidad arterial:**

$$
F_C(t)=C\frac{dP_p(t)}{dt}.
$$

**Resistencia periférica:**

$$
F_R(t)=\frac{P_p(t)}{R}.
$$

### 6.3. Ecuación integro-diferencial del modelo

Al sustituir las relaciones anteriores en la conservación de flujo:

$$
\boxed{
\frac{P_a(t)-P_p(t)}{Z}
+\frac{1}{L}\int_0^t[P_a(\tau)-P_p(\tau)]\,d\tau
=C\frac{dP_p(t)}{dt}+\frac{P_p(t)}{R}
}
$$

Esta expresión describe el balance del flujo arterial entre las cuatro ramas del modelo bajo la condición inicial indicada. El desarrollo manuscrito está disponible en **[análisis.pdf](calculos/an%C3%A1lisis.pdf)**.

## 7. Función de transferencia

Considerando condiciones iniciales nulas, la transformada de Laplace de las ecuaciones anteriores conduce a:

$$
\frac{P_a(s)-P_p(s)}{Z}+
\frac{P_a(s)-P_p(s)}{Ls}
=CsP_p(s)+\frac{P_p(s)}{R}.
$$

Agrupando términos y despejando la salida:

$$
\left(\frac{1}{Z}+\frac{1}{Ls}\right)P_a(s)=
\left(Cs+\frac{1}{R}+\frac{1}{Z}+\frac{1}{Ls}\right)P_p(s).
$$

La función de transferencia general es:

$$
\boxed{G(s)=\frac{P_p(s)}{P_a(s)}
=\frac{RLs+RZ}{CLRZs^2+L(Z+R)s+RZ}}.
$$

Con Z = 0.010, L = 0.001 y R = 1.000:

$$
G_C(s)=\frac{0.001s+0.010}{0.000010Cs^2+0.001010s+0.010}.
$$

### Funciones para los tres escenarios

**Normotenso:**

$$
\boxed{G_{\mathrm{normo}}(s)=\frac{0.001s+0.010}{0.000015s^2+0.001010s+0.010}}
$$

**Hipotenso:**

$$
\boxed{G_{\mathrm{hipo}}(s)=\frac{0.001s+0.010}{0.000035s^2+0.001010s+0.010}}
$$

**Hipertenso:**

$$
\boxed{G_{\mathrm{hiper}}(s)=\frac{0.001s+0.010}{0.000010s^2+0.001010s+0.010}}
$$

En las tres funciones, el cero de la planta es s = −10 s⁻¹. Las diferencias se observan en los denominadores, porque cambia C.

## 8. Representación en espacio de estados

De acuerdo con la sección 5.4.3 del syllabus se seleccionan las variables de estado:

$$
x_1(t)=F_L(t),\quad x_2(t)=P_p(t),\quad u(t)=P_a(t).
$$

Entonces:

$$
\dot x_1(t)=\frac{1}{L}\,[u(t)-x_2(t)],
$$

$$
\dot x_2(t)=\frac{1}{C}x_1(t)
-\left(\frac{1}{ZC}+\frac{1}{RC}\right)x_2(t)
+\frac{1}{ZC}u(t).
$$

En forma matricial:

$$
\boxed{\dot{\mathbf{x}}(t)=
\underbrace{\begin{bmatrix}
0&-1/L\\
1/C&-\left(1/(ZC)+1/(RC)\right)
\end{bmatrix}}_{A}\mathbf{x}(t)
+\underbrace{\begin{bmatrix}1/L\\1/(ZC)\end{bmatrix}}_{B}u(t)}
$$

$$
\boxed{y(t)=\begin{bmatrix}0&1\end{bmatrix}\mathbf{x}(t)=P_p(t).}
$$

Este modelo permite construir directamente la representación mediante integradores, sumadores y ganancias en Simulink.

## 9. Análisis de estabilidad y polos

El denominador característico del modelo es:

$$
D(s)=a_2s^2+a_1s+a_0,
$$

con:

$$
a_2=CLRZ,\qquad a_1=L(Z+R),\qquad a_0=RZ.
$$

Los polos se calculan por:

$$
\boxed{s_{1,2}=
\frac{-L(Z+R)\pm\sqrt{[L(Z+R)]^2-4(CLRZ)(RZ)}}{2CLRZ}.}
$$

| Condición | Polos calculados (s⁻¹) | Tipo de polos | Estabilidad |
|---|---|---|---|
| Normotenso | −55.2717, −12.0616 | Dos reales negativos distintos | Estable, sobreamortiguado |
| Hipotenso | −14.4286 ± 8.8051j | Complejos conjugados con parte real negativa | Estable, subamortiguado |
| Hipertenso | −89.8732, −11.1268 | Dos reales negativos distintos | Estable, sobreamortiguado |

Los tres conjuntos de parámetros tienen polos con parte real negativa, por lo que sus funciones de transferencia son estables. El caso hipotenso presenta un par complejo conjugado, mientras que los otros dos tienen polos reales. La clasificación describe la **dinámica lineal del modelo**: la forma visible de una respuesta temporal también depende de la señal aplicada y del cero de la transferencia.

**Verificación independiente:** [`scripts/verificar_polos.py`](scripts/verificar_polos.py). Este script calcula los polos sin utilizar valores experimentales ni ganancias PID supuestas.

## 10. Error en estado estacionario

Para una entrada escalón unitario aplicada a la planta en lazo abierto, Pₐ(s) = 1/s. Si el sistema es estable:

$$
\begin{aligned}
e_{ss}&=\lim_{s\to0}s\left(P_a(s)-P_p(s)\right)\\
&=1-\lim_{s\to0}G(s).
\end{aligned}
$$

Como G(0) = RZ/(RZ) = 1:

$$
\boxed{e_{ss}=0.}
$$

Este resultado es **para un escalón unitario y condiciones iniciales nulas**; no significa que el modelo siga sin error cualquier entrada variable en el tiempo. La señal de excitación aleatoria utilizada en la práctica tiene un comportamiento distinto al escalón.

## 11. Señal de excitación y simulaciones

La sección 5.4.4 del syllabus indica construir una señal de entrada con el bloque **`Uniform Random Number`** y aplicar **la misma realización** de la señal a las tres condiciones.

| Configuración | Valor del instructivo |
|---|---|
| Valor mínimo | −0.2 |
| Valor máximo | 1 |
| Semilla (`seed`) | 106 |
| Tiempo de muestreo Tₛ | 0.1 s |
| Intervalo de simulación | 0 ≤ t ≤ 15 s |

La reproducibilidad exige conservar la misma semilla, secuencia de entrada, condiciones iniciales y parámetros numéricos en los tres escenarios. La configuración concreta del bloque y la generación equivalente en Python deben comprobarse con los archivos usados en la simulación: no basta con compartir solo la semilla cuando las plataformas utilizan algoritmos aleatorios distintos.

Se comparan las presiones:

$$
P_{p,\mathrm{normo}}(t),\qquad
P_{p,\mathrm{hipo}}(t),\qquad
P_{p,\mathrm{hiper}}(t).
$$

## 12. Control PID y seguimiento de referencia

El modelo normotenso se utiliza como **referencia de control**. Se construyen dos esquemas independientes: uno para el modelo hipotenso y otro para el hipertenso.

### 12.1. Señales de error

$$
\boxed{e_{\mathrm{hipo}}(t)=P_{p,\mathrm{normo}}(t)-P_{p,\mathrm{hipo}}(t)}
$$

$$
\boxed{e_{\mathrm{hiper}}(t)=P_{p,\mathrm{normo}}(t)-P_{p,\mathrm{hiper}}(t)}
$$

### 12.2. Ley general del controlador

$$
\boxed{u_c(t)=k_Pe(t)+k_I\int_0^t e(\tau)\,d\tau
+k_D\frac{de(t)}{dt}}.
$$

En Laplace, para el PID ideal:

$$
K(s)=k_P+\frac{k_I}{s}+k_Ds.
$$

Los dos controladores se ajustan **por separado mediante `PID Controller` y `Tune`**. El objetivo indicado por el profesor es reducir la diferencia respecto de la respuesta normotensa, manteniendo una respuesta estable. La práctica no fija para este ejercicio los mismos límites numéricos de tiempo de establecimiento y sobreimpulso de la práctica introductoria pulmonar.

### 12.3. Evidencia de sintonización

Las ganancias k_P, k_I y k_D obtenidas en Tune, junto con las capturas originales, forman parte del trabajo realizado. **Sus valores numéricos no están transcritos aquí porque no figuran en el PDF manuscrito recibido**. Para incorporarlos al repositorio sin alterar los resultados reales, se debe emplear la exportación o captura de la sintonización efectivamente utilizada.

El contraste requerido por el instructivo es:

- **Hipotenso:** P_p,normo(t) vs. P_p,hipo(t) vs. P_p,hipo,PID(t).
- **Hipertenso:** P_p,normo(t) vs. P_p,hiper(t) vs. P_p,hiper,PID(t).

## 13. Implementación y verificación cruzada

La práctica contempla cinco representaciones o procedimientos que describen el mismo fenómeno matemático:

| Medio | Representación | Qué se compara |
|---|---|---|
| Simulink | Ecuaciones de estado | Señal de salida y dinámica temporal |
| Simulink | Función de transferencia | Concordancia con las EDO |
| Simscape Electrical | Circuito equivalente Z, L, C y R | Concordancia con la analogía eléctrica |
| MATLAB | Cuaderno y procesamiento de resultados | Figuras de referencia, caso y caso controlado |
| Python | Transferencias y simulación independiente | Respuestas equivalentes para los tres escenarios |

En cada implementación se deben respetar los mismos parámetros, condiciones iniciales y señal de entrada. Las figuras de lazo abierto y cerrado se guardan para comparar amplitudes, comportamiento transitorio y seguimiento de la referencia.

**Aclaración:** la concordancia cuantitativa entre plataformas solo puede afirmarse después de contrastar las gráficas o datos exportados; no se incluyen porcentajes de error supuestos.

## 14. Diagrama fisiológico en BioRender

El diagrama biológico de la práctica relaciona el circuito con el trayecto arterial sistémico. Debe mostrar:

- El corazón como origen de la entrada de presión y flujo.
- La aorta proximal asociada con los efectos de **Z** y **L**.
- Las grandes arterias elásticas relacionadas con **C**.
- Los vasos periféricos y arteriolas relacionados con **R**.
- La presión de entrada Pₐ(t), la presión modelada Pₚ(t) y el sentido del flujo sanguíneo.
- La comparación de los tres valores de C, que diferencian las condiciones fisiológicas de esta práctica.

Las correspondencias son conceptuales: los elementos del circuito representan parámetros hemodinámicos concentrados, no segmentos anatómicos idénticos a una resistencia, un capacitor o un inductor físicos.

## 15. Resultados y discusión

### 15.1. Resultados analíticos comprobados

El cálculo del modelo permite establecer que:

1. La función de transferencia es de **segundo orden**, consistente con los elementos de almacenamiento L y C.
2. Los tres casos son **estables**, ya que todos sus polos tienen parte real negativa.
3. El modelo **hipotenso** tiene polos complejos conjugados y es subamortiguado.
4. Los modelos **normotenso** e **hipertenso** presentan polos reales negativos distintos y son sobreamortiguados.
5. Ante un **escalón unitario**, la ganancia estática es uno y el error de seguimiento en estado estacionario de la planta es cero.

### 15.2. Interpretación fisiológica

Al mantener Z, L y R constantes, la comparación permite aislar el efecto de C sobre la dinámica arterial. La condición hipotensa del modelo presenta mayor capacidad de distensión y almacenamiento de volumen frente a un cambio de presión; el caso hipertenso presenta una distensibilidad menor, asociada con una respuesta de presión más pronunciada para una misma variación de volumen. Estas diferencias modifican la posición de los polos y el comportamiento transitorio.

### 15.3. Comparación de resultados computacionales

La comparación de las respuestas de lazo abierto y lazo cerrado, las ganancias de los controladores y la concordancia MATLAB–Python–Simscape **debe respaldarse con las figuras y archivos generados durante la práctica**. Este apartado registra las conclusiones analíticas verificadas; no atribuye amplitudes, errores máximos ni mejoras porcentuales no presentes en las fuentes aportadas.

## 16. Estructura del repositorio y evidencias

La siguiente estructura organiza los entregables solicitados por el syllabus. Los nombres de los modelos y las figuras son **convenciones sugeridas**, no afirmaciones sobre archivos que ya se hayan subido.

```text
Practica2_Windkessel/
├── README.md
├── CITATION.cff
├── calculos/
│   └── análisis.pdf
├── scripts/
│   └── verificar_polos.py
├── matlab/                 # Agregar archivos .m, .mlx y .slx propios
├── python/                 # Agregar código de simulación propio
├── figuras/                # Gráficas exportadas desde MATLAB y Python
├── biorender/              # Figura biológica exportada
└── evidencias/              # Capturas de Tune y Simscape
```

**Archivo ya incluido:** [`análisis.pdf`](calculos/an%C3%A1lisis.pdf), con seis páginas de cálculos manuscritos. Se reorganizó la numeración física del escaneo para poner la hoja del circuito al principio: orden original **6 → 1 → 2 → 3 → 4 → 5**, sin modificar el contenido de las hojas.

**Verificación analítica incluida:** [`scripts/verificar_polos.py`](scripts/verificar_polos.py). Este código permite recalcular los polos y el error ante escalón con los parámetros del syllabus.

**Evidencias que deben añadirse desde los archivos del trabajo terminado:** proyectos de Simulink y Simscape; cuaderno MATLAB y gráficos PDF; programa Python y sus gráficos; configuración y figura de la señal aleatoria; capturas de los dos PID con sus ganancias; resultados en lazo abierto y lazo cerrado; y la ilustración de BioRender. No se generan evidencias simuladas ni se usan valores ficticios para completar estas carpetas.

## 17. Reproducibilidad y referencias

### Reproducibilidad

1. Consultar la sección 5.4 del syllabus e identificar la topología de la **Figura 5.1**.
2. Verificar los parámetros y las funciones de transferencia de los tres escenarios.
3. Ejecutar el script `scripts/verificar_polos.py` como comprobación independiente de los polos.
4. Abrir los modelos del estudiante en Simulink y Simscape y configurar la señal de entrada según el instructivo.
5. Ejecutar las simulaciones de lazo abierto y después las dos configuraciones de control PID.
6. Comparar la referencia normotensa con los casos hipotenso e hipertenso, con y sin controlador.
7. Reproducir el procedimiento en Python y comprobar la coincidencia de las respuestas.
8. Incluir los archivos originales y las figuras generadas; verificar que los enlaces relativos del README correspondan a rutas existentes.

### Referencias

[1] Valle Trujillo, P. A. (2026). *Syllabus para Modelado de Sistemas Fisiológicos*. Tecnológico Nacional de México / Instituto Tecnológico de Tijuana, sección 5.4, pp. 76–85.

[2] Khoo, M. C. K. (2018). *Physiological Control Systems: Analysis, Simulation, and Estimation* (2nd ed.). IEEE Press.

[3] Hall, J. E. (2021). *Guyton and Hall Textbook of Medical Physiology* (14th ed.). Elsevier.

[4] Tecnológico Nacional de México. (2016). *Programa de estudio de la asignatura Modelado de Sistemas Fisiológicos (IBF-1020)*.

---

### Nota de alcance

Esta práctica corresponde a una actividad académica de **modelado y experimentación *in silico***. Los escenarios normotenso, hipotenso e hipertenso son simplificaciones matemáticas y los controladores desarrollados **no constituyen intervenciones clínicas ni recomendaciones terapéuticas**.

**Autor:** Castro Alcántar Daniel Hiram · **23212184** · **Ingeniería Biomédica, Instituto Tecnológico de Tijuana**.
