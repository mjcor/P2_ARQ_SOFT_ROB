# Práctica P2 (Tema 2) — Informe

**Nombre:** <<RELLENAR>>
**Fecha:** <<RELLENAR>>
**Entorno usado:** (Colab / local; versión de Python; ¿ROS2? ¿cuál?) <<RELLENAR>>

> Regla de oro: **hipótesis escrita ANTES de medir**. Cada afirmación lleva su evidencia (comando + salida o gráfica).

---

## Declaración de autoría y uso de IA

Trabajo individual. Fuentes consultadas: <<RELLENAR>>
Uso de asistentes de IA (para qué, o «ninguno»): <<RELLENAR>>


## Ejercicio 1 — Caso 5 (`bags/caso_5_casa`)

| Campo | Contenido |
|---|---|
| **Síntoma (lo que se observa)** | El topic `/distancia` publica a 1 Hz, aunque se esperan 10 Hz. Los otros dos topics también funcionan a 1 Hz. |
| **Hipótesis (escrita antes del análisis completo)** | Al observar que los tres topics tenían 40 mensajes durante unos 39 segundos, sospeché que el problema comenzaba en el sensor `/distancia`, que estaba publicando más lentamente de lo esperado. Supuse que el filtro y el controlador estaban siguiendo ese mismo ritmo porque recibían menos datos desde el origen. |
| **Comando(s)** | `python3 analiza_bag.py bags/caso_5_casa --info` y `python3 analiza_bag.py bags/caso_5_casa --esperado /distancia=10 --png informe_caso5.png` |
| **Evidencia (salida / gráfica)** | El análisis confirma que los tres topics contienen 40 mensajes y tienen una frecuencia de 1,00 Hz. La herramienta detecta `[ERROR] RITMO_BAJO` en `/distancia`, con 1 Hz medido frente a los 10 Hz esperados. El intervalo máximo entre mensajes es de 1 segundo. Las latencias medias son pequeñas: 0,002 s para `/distancia` y 0,007 s para `/distancia_filtrada`. La gráfica `informe_caso5.png` permite observar los ritmos, las latencias y los valores de distancia. |
| **Eslabón donde se rompe** | El problema aparece en el origen de la cadena, en el sensor que publica `/distancia`. No se observa una caída adicional de frecuencia entre el sensor, el filtro y el controlador. |
| **Familia (estructural / dinámico / tiempo / origen)** | Fallo de origen aguas arriba, que se manifiesta como una frecuencia de publicación inferior a la esperada. |
| **Causa raíz probable** | Es posible que el nodo del sensor tenga mal configurada su frecuencia de publicación. Por ejemplo, podría estar utilizando un temporizador de 1 segundo en lugar de 0,1 segundos. Esta causa es probable, pero no se puede confirmar únicamente con el bag. |
| **Arreglo concreto y cómo comprobarlo** | Revisaría el código o la configuración del nodo que publica `/distancia` y ajustaría su temporizador a 0,1 segundos para conseguir 10 Hz. Después comprobaría la frecuencia con `ros2 topic hz /distancia`, grabaría un nuevo bag y volvería a ejecutar `analiza_bag.py`. El resultado esperado sería una frecuencia cercana a 10 Hz y la desaparición del error `RITMO_BAJO`. |

### ¿Qué nos dice el grafo del sistema y qué nos dice el bag?

El grafo de ROS 2 permite observar qué nodos y topics existen y cómo están conectados, pero no demuestra que los mensajes se estén publicando a la frecuencia adecuada.

En cambio, el bag contiene los mensajes grabados y sus tiempos, por lo que permite analizar el comportamiento real del sistema.

En este caso, los tres topics tienen mensajes, pero el análisis del bag demuestra que funcionan a 1 Hz en lugar de los 10 Hz esperados para el sensor.

Por tanto, una conexión aparentemente correcta no garantiza que el sistema funcione correctamente. Para diagnosticarlo hay que complementar la información del grafo con mediciones, como en el método estudiado en la sesión 7.

### Pregunta de control: ¿qué ocurriría sin `--esperado`?

La herramienta supone por defecto que el primer topic debe funcionar a 10 Hz.

Si analizáramos un sensor diseñado realmente para publicar a 1 Hz, podría detectar incorrectamente un error `RITMO_BAJO`, aunque el sensor estuviera funcionando bien.

Por eso, la frecuencia esperada debería estar definida en los requisitos del sistema y reflejada en la configuración del nodo o en un contrato de pruebas. La herramienta de análisis debería consultar ese requisito, en lugar de asumir siempre una frecuencia de 10 Hz.
## Ejercicio 2 — Mis propios bags

Bags entregados (en `bags/`, nombres que empiezan por `mi_`): <<RELLENAR>>
Cómo los generé (comandos): <<RELLENAR>>

Tabla comparativa sano / con fallo (ritmos, latencia, hallazgos): <<RELLENAR>>

**¿Qué no puede medir la herramienta con mensajes sin cabecera (`Float32`)? ¿Por qué?** <<RELLENAR>>

## Ejercicio 3 — Extensión de la herramienta

Comprobación(es) implementada(s): <<RELLENAR>>
Decisiones de diseño (¿ERROR o WARN? ¿margen?, ¿qué falsos positivos hay?): <<RELLENAR>>
Pruebas propias añadidas (nombre y qué comprueban): <<RELLENAR>>
Salida de `python3 -m unittest -v test_mis_comprobaciones`: <<RELLENAR>>
Resultado sobre `bags/mi_rango` (antes: TODO_OK; ahora: ...): <<RELLENAR>>

## Ejercicio 4 — Un quinto fallo

Ficha del fallo (SIN la solución; es lo que recibirá tu compañero):
- Nombre del bag: <<RELLENAR>>
- Síntoma observable: <<RELLENAR>>
- Qué debe averiguar quien lo resuelva: <<RELLENAR>>

**Solución (aparte, en sobre cerrado hasta que el compañero entregue):**
- Cómo se genera (parámetros o función `post`): <<RELLENAR>>
- Hallazgo(s) de `analiza_bag.py` que lo detectan, o por qué la herramienta no lo detecta: <<RELLENAR>>
- Eslabón, familia, causa y arreglo: <<RELLENAR>>

Resultado de la resolución del compañero (¿acertó? ¿qué le faltó? Si no hubo intercambio, «sin intercambio» y por qué): <<RELLENAR>>

## Ejercicio 5 — Lista de comprobación ampliada

Las 7 preguntas de la clase + 3 mías (cada una con su herramienta, qué detecta y un caso que la justifique):

| # | Pregunta | Herramienta | Detecta | Caso que la justifica |
|---|---|---|---|---|
| 8 | <<RELLENAR>> | | | |
| 9 | <<RELLENAR>> | | | |
| 10 | <<RELLENAR>> | | | |

## Ejercicio 6 — Autoformación guiada

Puntos de control de la *Guía de autoformación de la S8* (resumen de lo aprendido y dudas): <<RELLENAR>>

## Ejercicio 7 — Repositorio Git con integración continua

| Campo | Contenido |
|---|---|
| URL del repositorio (público, o privado con acceso para el profesor) | <<RELLENAR>> |
| Nº de commits y rama(s) de trabajo | <<RELLENAR>> |
| Pull Request (o merge) que integra una rama en `main` (URL) | <<RELLENAR>> |
| Ejecución de CI en **verde** (URL de la ejecución en Actions) | <<RELLENAR>> |
| Ejecución de CI en **rojo** provocada a propósito (URL) y commit que la arregla | <<RELLENAR>> |
| Qué rompí a propósito y cómo lo diagnostiqué en el log | <<RELLENAR>> |
| Bag(s) que añadí a `esperado.json` y por qué | <<RELLENAR>> |

Si **no** has podido usar GitHub, escribe aquí por qué y adjunta en `evidencias/` el `git_log.txt`, la salida de `simula_ci.py` y las capturas `ci_rojo` / `ci_verde` de la simulación local: la nota máxima del ejercicio baja (ver enunciado).

## Memoria conceptual (con tus palabras; máximo 1 página en total)

**M1.** Familias de fallo de arquitectura (estructural, dinámico, de tiempo, de origen aguas arriba): un ejemplo de cada una y la herramienta más barata que la detectaría. <<RELLENAR>>

**M2.** ¿Por qué una herramienta de análisis necesita pruebas unitarias, y qué significa que «un fallo no detectado es una especificación pendiente»? <<RELLENAR>>

**M3.** ¿Qué ocurre exactamente en un conflicto de fusión en Git, cómo se ve en el fichero y cómo se resuelve? ¿Por qué no se debe «aceptar el mío» sin leer? <<RELLENAR>>

**M4.** ¿Qué garantiza y qué NO garantiza una CI en verde? Usa como ejemplo el contrato de regresión con bags. <<RELLENAR>>

## Opcional — `ros2 bag play` y RViz2

<<RELLENAR o "No realizado">>

## Reflexión final (5 líneas)

¿Qué fue lo más difícil? ¿Qué cambiarías en la herramienta? ¿Qué le faltó al bag para estar seguro de la causa? <<RELLENAR>>
