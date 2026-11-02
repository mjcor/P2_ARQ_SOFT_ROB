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
| Síntoma (lo que se observa) | El topic `/distancia` publica a 1 Hz, aunque se esperan 10 Hz. Los otros dos topics también funcionan aproximadamente a 1 Hz. |
| Hipótesis (escrita antes de ejecutar nada) | Sospeché que el problema estaba en el nodo que publica `/distancia`, posiblemente porque su temporizador estaba configurado con un periodo demasiado largo. Esto explicaría que los demás topics recibieran datos a una frecuencia inferior a la esperada. |
| Comando(s) | `python3 analiza_bag.py bags/caso_5_casa --esperado /distancia=10 --png informe_caso5.png` |
| Evidencia (salida / gráfica) | El análisis muestra aproximadamente 40 mensajes por topic durante 39 segundos, con frecuencias cercanas a 1 Hz. La herramienta detecta `RITMO_BAJO` en `/distancia` al compararlo con los 10 Hz esperados. |
| Eslabón donde se rompe | El problema aparece en el origen de la cadena, en el nodo sensor que publica `/distancia`. No se observa una caída adicional de frecuencia entre los siguientes eslabones. |
| Familia (estructural / dinámico / tiempo / origen) | Fallo de origen, con consecuencias temporales sobre el funcionamiento de la cadena. |
| Causa raíz probable | Una configuración incorrecta del temporizador del nodo sensor. Es posible que publique cada 1 segundo en lugar de cada 0,1 segundos. Esta causa es una hipótesis que habría que confirmar revisando el código o la configuración del nodo. |
| Arreglo concreto y cómo comprobarlo | Revisaría la configuración del temporizador del nodo que publica `/distancia` y establecería un periodo de 0,1 segundos para conseguir 10 Hz. Después grabaría un nuevo bag y repetiría el análisis con `--esperado /distancia=10` para comprobar que la frecuencia es correcta y desaparece el error. |

¿Qué os dice el grafo del sistema y qué os dice el bag? (pista: comparad con el caso D de la S7) El grafo de ROS 2 permite observar los nodos, los topics y sus conexiones, pero no demuestra que los mensajes se publiquen a la frecuencia esperada. En cambio, el bag contiene los mensajes registrados y sus tiempos, por lo que permite analizar el comportamiento real del sistema. En este caso, aunque los topics están conectados, el bag demuestra que la frecuencia de publicación es de aproximadamente 1 Hz en lugar de los 10 Hz esperados. Por tanto, es necesario complementar la información del grafo con el análisis de los mensajes para detectar este tipo de fallos.

## Ejercicio 2 — Mis propios bags

Bags entregados (en `bags/`, nombres que empiezan por `mi_`): mi_sano, mi_fallo_a, mi_float y mi_float_lento, guardados en la carpeta bags/.
Cómo los generé (comandos): 

sim.generar('mi_sano', '/content/autonoma/bags', frecuencia=10, duracion=40)
sim.generar('mi_fallo_a', '/content/autonoma/bags', frecuencia=10, duracion=40, retardo_filtro=0.3)
sim.generar('mi_float', '/content/autonoma/bags', frecuencia=10, duracion=40, tipo_mensaje='Float32')
sim.generar('mi_float_lento', '/content/autonoma/bags', frecuencia=10, duracion=40, tipo_mensaje='Float32', retardo_filtro=0.3)

Tabla comparativa sano / con fallo (ritmos, latencia, hallazgos): 

| Bag | Frecuencia sensor | Frecuencia filtro | Latencia filtro | Hallazgos |
|---|---|---|---|---|
| mi_sano | 10 Hz | 10 Hz | 0,007 s | TODO_OK |
| mi_fallo_a | 10 Hz | 3,33 Hz | 1,205 s | CAIDA_EN_CADENA y LATENCIA_ALTA |
| mi_float | 10 Hz | 10 Hz | No medible | TODO_OK |
| mi_float_lento | 10 Hz | 3,33 Hz | No medible | CAIDA_EN_CADENA |

**¿Qué no puede medir la herramienta con mensajes sin cabecera (`Float32`)? ¿Por qué?** 
Con mensajes Float32, la herramienta no puede calcular la latencia desde la generación del dato ni comprobar posibles problemas de reloj, porque estos mensajes no tienen header.stamp. Por eso, en mi_float_lento detecta CAIDA_EN_CADENA, pero no LATENCIA_ALTA, aunque el fallo sea el mismo que en mi_fallo_a. Para medir la latencia en un sistema real utilizaría mensajes con marcas temporales o registraría los tiempos de generación y recepción de cada dato.


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
