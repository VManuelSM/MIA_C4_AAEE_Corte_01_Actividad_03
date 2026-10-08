# Actividad 03 · Algoritmo genético en siete etapas para seleccionar una cartera de proyectos

| | |
|---|---|
| **Alumno** | Víctor Manuel Santos Martínez — matrícula 253220020 |
| **Materia** | Algoritmos Evolutivos — Maestría en Inteligencia Artificial |
| **Docente** | Dr. Jaime Aguilar Ortiz |
| **Actividad** | Actividad 03 — Aplicación de un algoritmo genético con la metodología de siete etapas |
| **Modalidad** | Individual |
| **Periodo** | Septiembre–diciembre de 2026 · Corte 1 |
| **Institución** | Universidad Politécnica Metropolitana de Hidalgo |

## Descripción de la actividad

> Desarrollar una aplicación de algoritmo genético aplicando la metodología de 7 etapas analizadas en clase. Para cada etapa elegir uno de los métodos propuestos y justificar la elección.

El problema elegido es la **selección de una cartera de proyectos indivisibles**, el caso 7.3 del material de la semana 1. Una empresa elige qué proyectos financiar entre veinte o cincuenta candidatos para maximizar el valor esperado. Hay cuatro tipos de restricción: presupuesto de capital, horas-persona del equipo interno, dependencias (un proyecto requiere otro) y exclusiones (dos alternativas no pueden elegirse juntas).

Cada gen es un bit que indica si un proyecto entra en la cartera. Esa representación decide los métodos: de la guía de los treinta métodos se elige uno por etapa entre los compatibles con genes binarios.

| Etapa | Método | Por qué |
|---|---|---|
| 1. Inicialización | **M02** Bernoulli, `p = mín(B/Σc, H/Σh)` | Único método binario; `p` centra el gasto esperado en la capacidad más escasa |
| 2. Evaluación | **M08** prioridad de factibilidad | Hay restricciones violables; no exige ajustar un coeficiente de penalización |
| 3. Selección | **M12** torneo con reemplazo, `k = 2` | Compara claves lexicográficas sin convertirlas en probabilidades |
| 4. Recombinación | **M14** cruce uniforme, `q = 0.5`, `pc = 0.9` | El orden de los proyectos es una etiqueta: no hay bloques contiguos que preservar |
| 5. Mutación | **M17** inversión de bits, `pm = 0.5/d` | Único método nativo de la representación binaria |
| 6. Reemplazo | **M22** generacional con elitismo, `e = 2` | La mejor cartera factible nunca se pierde |
| 7. Paro | **M28** estancamiento, `W = 200` | Termina siempre en un dominio finito y hace explícito el gasto de espera |

`pm`, `k` y `W` se calibraron en una instancia que no se usa para evaluar, con reglas fijadas antes de ver los resultados.

## Resultados

Cuatro instancias de evaluación (20 y 50 proyectos, con correlación fuerte y débil entre valor y recursos) y treinta semillas por instancia. Cada corrida genética se compara con el óptimo exacto (enumeración de las 2²⁰ carteras y programación entera), con una heurística voraz y con una búsqueda aleatoria emparejada por semilla y con el mismo número de evaluaciones.

| Instancia | Brecha mediana del AG | Aciertos del óptimo | Brecha mediana de la búsqueda aleatoria | Brecha del voraz | Wilcoxon, *p* unilateral |
|---|---:|---:|---:|---:|---:|
| E20-F | 0.443 % | 2/30 | 0.804 % | 12.29 % | 0.0034 |
| E20-D | 0.000 % | 23/30 | 3.174 % | 1.81 % | 2.8 × 10⁻⁶ |
| E50-F | 1.163 % | 0/30 | 2.349 % | 14.22 % | 9.3 × 10⁻¹⁰ |
| E50-D | 0.589 % | 0/30 | 10.770 % | 8.26 % | 9.3 × 10⁻¹⁰ |

- **El AG supera siempre al voraz** y supera a la búsqueda aleatoria en las cuatro instancias. La ventaja es pequeña con veinte proyectos, donde la búsqueda aleatoria gana en 6 de 30 semillas de E20-F, y grande con cincuenta.
- **M08 frente a M07.** Con penalización y `ρ ≤ 0.1`, las 30 corridas entregan una cartera inviable. Con `ρ = 1` la entregan 12 de 30. Sólo con `ρ = 10` la penalización iguala a M08, que lo logra sin coeficiente que ajustar.
- **Costo del paro.** M28 gasta `W(N−e) = 19 600` evaluaciones después de la última mejora: la mitad del presupuesto con cincuenta proyectos y el 84–87 % con veinte.
- **Caso principal (E20-F).** La cartera óptima vale 257.38 millones con nueve proyectos y agota ambos recursos. Sólo 53 de las 54 656 carteras factibles quedan a 1 % o menos del óptimo.

## Contenido del repositorio

| Ruta | Contenido |
|---|---|
| `Actividad_03_Cartera_Proyectos.ipynb` | Cuadernillo autocontenido y ejecutado: modelo, siete etapas, trece pruebas, experimentos, figuras y lectura de resultados |
| `resultados/instancias.csv` | Datos agregados, óptimos y voraz de las cinco instancias |
| `resultados/calibracion.csv` | 130 corridas de la calibración de `pm`, `k` y `W` |
| `resultados/corridas.csv` | Una fila por corrida de evaluación (120), con su búsqueda aleatoria emparejada |
| `resultados/historiales.csv` | Trayectoria generación a generación de las 120 corridas |
| `resultados/busqueda_aleatoria.csv` | Trayectoria de las búsquedas aleatorias con la misma granularidad |
| `resultados/resumen.csv` | Agregados por instancia y prueba de Wilcoxon |
| `resultados/ablacion_*.csv` | M08 frente a M07 con cuatro valores de `ρ` (150 corridas) |
| `resultados/caso_principal_*.csv` | Carteras y proyectos de la instancia con nombres |
| `resultados/manifiesto.json` | Versiones, configuración, semillas y huellas SHA-256 de cada archivo |
| `figuras/` | Ocho figuras PNG generadas desde los datos de la ejecución |

## Reproducir

Desde esta carpeta, con el entorno conda `science`:

```bash
conda run -n science jupyter nbconvert --to notebook --execute --inplace \
  --ExecutePreprocessor.timeout=1800 Actividad_03_Cartera_Proyectos.ipynb
```

El cuadernillo no importa módulos `.py` propios y no descarga nada. La primera celda falla si el kernel no pertenece al entorno `science`. Las trece pruebas se ejecutan antes de cualquier experimento, y el cuadernillo se detiene si una falla. Cinco de ellas reproducen reactivos del examen parcial 1: la mochila de cinco objetos, el torneo, el cruce por máscara, el elitismo y el estancamiento.

Entorno comprobado: Python 3.11.16, NumPy 2.4.6, pandas 3.0.6, SciPy 1.17.1 y Matplotlib 3.11.2, en macOS arm64. La ejecución completa tarda unos 25 segundos. Dos ejecuciones dan archivos idénticos salvo en las columnas de tiempo.

## Alcance y límites

- **Los datos son sintéticos.** Las cifras en millones de pesos son una escala de trabajo, no estimaciones financieras de una empresa real.
- **El algoritmo genético no es la herramienta necesaria para este modelo.** El modelo es lineal con variables binarias, y la programación entera lo resuelve de forma exacta en menos de un segundo. El AG es el objeto de estudio. Tendría sentido práctico si la evaluación dejara de ser lineal (sinergias entre proyectos, riesgo de la cartera, simulación), y entonces sólo cambiaría la etapa 2.
- **No se generalizan los resultados.** Describen cuatro instancias y una configuración calibrada en una quinta; no se extrapolan a otros tamaños o estructuras de restricciones.
- **La calibración usa diez semillas por celda.** Las mejores celdas de la rejilla quedan a 0.002 puntos porcentuales entre sí, una diferencia que no las distingue. La ventana elegida está en el extremo de la rejilla probada.

## Procedencia y autoría

Las siete etapas, los treinta métodos y su nomenclatura M01–M30 proceden del material de la materia (Aguilar Ortiz). La regla de prioridad de factibilidad tiene antecedente en Deb (2000), y la clasificación de instancias por correlación en Pisinger (2005). El modelo de cartera, los datos sintéticos, el diseño experimental, la calibración y la ablación son decisiones de este desarrollo.

Se empleó asistencia de inteligencia artificial para la programación y la redacción. Las cifras no proceden de esa asistencia: se obtienen al ejecutar el cuadernillo y sus trece pruebas.
