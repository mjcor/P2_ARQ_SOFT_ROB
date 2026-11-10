# Práctica P2 (Tema 2) — Informe

**Nombre:** Marina
**Fecha:** 02/11/2026
**Entorno usado:** (Colab / local; versión de Python; ¿ROS2? ¿cuál?) Google Colab, con Python 3.13. No se ha utilizado una instalación completa de ROS 2; se ha trabajado con bags sintéticos mediante la biblioteca rosbags, utilizando los tipos de mensajes de ROS 2 Jazzy.

> Regla de oro: **hipótesis escrita ANTES de medir**. Cada afirmación lleva su evidencia (comando + salida o gráfica).

---

## Declaración de autoría y uso de IA

Trabajo individual. Fuentes consultadas: Enunciado de la práctica P2, cuaderno P2_Practica_Colab.ipynb y materiales de clase de las sesiones 7, 8 y 9.
Uso de asistentes de IA (para qué, o «ninguno»): He utilizado ChatGPT como apoyo para interpretar el enunciado y resolver dudas técnicas.


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

Comprobación(es) implementada(s): He implementado la función `comprobar_rangos` en `mis_comprobaciones.py`. Esta función comprueba que las mediciones de un sensor Range estén dentro de los límites `min_range` y `max_range`. Cuando encuentra valores incorrectos, devuelve un hallazgo `RANGO_FUERA` que indica cuántas muestras fallan y el intervalo temporal en el que aparecen.
Decisiones de diseño (¿ERROR o WARN? ¿margen?, ¿qué falsos positivos hay?): He elegido ERROR para `RANGO_FUERA` porque una medición fuera de los límites declarados puede indicar un dato físicamente inválido. La función admite un margen de tolerancia configurable, que por defecto es 0. Un margen demasiado pequeño puede producir falsos positivos por errores de medida o redondeo. También pueden aparecer falsos positivos si un sensor utiliza `inf` para indicar que no ha detectado ningún objeto dentro de su alcance. `FRAME_MIXTO` se plantea como WARN porque encontrar distintos `frame_id` puede indicar una inconsistencia, aunque no necesariamente demuestra un fallo. Esta segunda comprobación queda como ampliación opcional no implementada.
Pruebas propias añadidas (nombre y qué comprueban): He añadido `test_valores_en_los_limites`, que verifica que los valores exactamente iguales al mínimo y al máximo permitidos se aceptan como válidos, y `test_dos_fallos_separados`, que comprueba que la herramienta detecta dos mediciones incorrectas no consecutivas e informa correctamente de su cantidad y sus tiempos.
Salida de `python3 -m unittest -v test_mis_comprobaciones`: Se ejecutaron 11 pruebas en 0.322 segundos, con resultado `OK (skipped=2)`. Las 9 pruebas no omitidas fueron correctas, incluidas las dos pruebas propias. Las dos pruebas omitidas corresponden a `comprobar_frames`, que es una ampliación opcional.
Resultado sobre `bags/mi_rango` (antes: TODO_OK; ahora: ...): Antes de implementar la comprobación, `analiza_bag.py` devolvía `TODO_OK`, aunque existían mediciones incorrectas. Después, `mis_comprobaciones.py` detecta `RANGO_FUERA` en `/distancia`, con 20 de 400 muestras fuera de rango, desde 10.10 s hasta 12.00 s, y código de salida 1. Al analizar el bag sano `caso_0_linea_base`, no se detectan problemas y el código de salida es 0.

## Ejercicio 4 — Un quinto fallo

Ficha del fallo (SIN la solución; es lo que recibirá tu compañero):
- Nombre del bag: `bags/mi_fallo_b` (empaquetado como `reto/reto_1`).
- Síntoma observable: El topic `/cmd_vel_demo` presenta 460 mensajes y una frecuencia media de 11.50 Hz, mientras que `/distancia` y `/distancia_filtrada` presentan 400 mensajes y 10.00 Hz. La herramienta `analiza_bag.py` devuelve `TODO_OK`.
- Qué debe averiguar quien lo resuelva: Por qué el controlador publica más mensajes de los esperados, en qué eslabón aparece la anomalía, cuál es su causa y cómo podría detectarse y corregirse.

**Solución (aparte, en sobre cerrado hasta que el compañero entregue):**
- Cómo se genera (parámetros o función `post`): Se utiliza la función `mi_fallo(msgs)` como argumento `post` de `simulador_bags.generar`. La función duplica los mensajes de `/cmd_vel_demo` entre los segundos 12 y 18, añadiendo una copia de cada mensaje con un retraso de 0.01 s. Después ordena los mensajes por su instante de publicación. Esto genera 60 mensajes adicionales en el controlador.
- Hallazgo(s) de `analiza_bag.py` que lo detectan, o por qué la herramienta no lo detecta: La herramienta devuelve `TODO_OK`, aunque `/cmd_vel_demo` tiene 460 mensajes y 11.50 Hz frente a los 400 mensajes y 10.00 Hz de los otros topics. No detecta el problema porque sus comprobaciones están orientadas a pérdidas de frecuencia, huecos, latencias elevadas y problemas de reloj, pero no comprueban si se publican comandos duplicados. Sería necesario añadir una comprobación de duplicados o de intervalos anormalmente cortos entre mensajes.
- Eslabón, familia, causa y arreglo: El fallo se localiza en el eslabón del controlador que publica `/cmd_vel_demo`. Pertenece a la familia de fallos dinámicos. La causa simulada es la duplicación de comandos durante un intervalo de seis segundos, equivalente a una publicación repetida por un error de programación o una ejecución duplicada del callback. El arreglo consiste en garantizar una única publicación por ciclo de control y comprobar que desaparecen los mensajes duplicados y que la frecuencia vuelve a 10 Hz.

Resultado de la resolución del compañero (¿acertó? ¿qué le faltó? Si no hubo intercambio, «sin intercambio» y por qué): Sin intercambio, porque no se ha podido coordinar la resolución del reto con otro compañero.

## Ejercicio 5 — Lista de comprobación ampliada

Las 7 preguntas de la clase + 3 mías (cada una con su herramienta, qué detecta y un caso que la justifique):

| # | Pregunta | Herramienta | Detecta | Caso que la justifica |
|---|---|---|---|---|
| 8 | ¿Se publican mensajes o comandos duplicados? | Análisis de los intervalos entre mensajes del bag y comprobación de duplicados | Publicaciones repetidas y frecuencias superiores a las esperadas | `mi_fallo_b` del ejercicio 4, donde `/cmd_vel_demo` publica 460 mensajes en lugar de 400 |
| 9 | ¿Los valores publicados por el sensor respetan sus límites físicos? | `comprobar_rangos` de `mis_comprobaciones.py` | Mediciones fuera de los límites `min_range` y `max_range` | `mi_rango` del ejercicio 3, con 20 de 400 muestras fuera de rango |
| 10 | ¿Son coherentes los marcos de referencia de los sensores con las transformaciones del robot? | `tf2_echo`, `view_frames` y RViz2 | Marcos de referencia incorrectos y transformaciones ausentes | Un sensor que publica datos en `sensor_link` sin una transformación válida hacia `base_link`, impidiendo su visualización correcta en RViz2 |


## Ejercicio 6 — Autoformación guiada

Puntos de control de la *Guía de autoformación de la S8* (resumen de lo aprendido y dudas): 
He aprendido que rosbag2 permite grabar, inspeccionar y reproducir mensajes de ROS2, lo que facilita analizar un fallo después de que haya ocurrido. También he comprendido la importancia del formato MCAP, las marcas temporales, la compresión, la división de grabaciones y la configuración de QoS para registrar correctamente los datos. En la práctica de Colab he trabajado con bags sintéticos, comparando frecuencias, huecos y latencias, y he comprobado que los mensajes sin cabecera no permiten medir ciertas propiedades temporales. Sobre RViz2, he aprendido que permite visualizar los datos de los sensores, pero necesita marcos de referencia coherentes. El sistema tf2 gestiona las transformaciones entre esos marcos, y una transformación ausente puede impedir que los datos aparezcan correctamente. En cuanto a la depuración de arquitectura, he practicado el método de formular una hipótesis, analizar las evidencias, localizar el eslabón afectado, clasificar el fallo y proponer una corrección verificable. No he realizado las pruebas prácticas de grabación avanzada y visualización con RViz2 en un entorno ROS2 local. Mis principales dudas son cómo distinguir con seguridad un problema de QoS de un callback lento, cómo medir la latencia de mensajes sin cabecera y cómo determinar la causa raíz cuando un bag solamente muestra los síntomas.

## Ejercicio 7 — Repositorio Git con integración continua

| Campo | Contenido |
|---|---|
| URL del repositorio (público, o privado con acceso para el profesor) | https://github.com/mjcor/P2_ARQ_SOFT_ROB |
| Nº de commits y rama(s) de trabajo | 11 commits en `main`, incluyendo el commit final con los ejercicios 4, 5 y 6, el informe y el cuaderno terminado. Se ha trabajado con las ramas `main` y `ej3-rango`, integrando esta última mediante un Pull Request. |
| Pull Request (o merge) que integra una rama en `main` (URL) | https://github.com/mjcor/P2_ARQ_SOFT_ROB/pull/1 |
| Ejecución de CI en **verde** (URL de la ejecución en Actions) | https://github.com/mjcor/P2_ARQ_SOFT_ROB/actions/runs/42808542747 |
| Ejecución de CI en **rojo** provocada a propósito (URL) y commit que la arregla | https://github.com/mjcor/P2_ARQ_SOFT_ROB/actions/runs/42802839576. Se corrigió mediante el commit `cffbc85`, que restauró el contrato de regresión. |
| Qué rompí a propósito y cómo lo diagnostiqué en el log | Modifiqué intencionadamente `esperado.json`, cambiando la salida esperada de `caso_0_linea_base` de 0 a 1. La CI falló porque el análisis real del bag sano devolvía 0. En el log aparecía `[FALLO] caso_0_linea_base`, con 4 de 5 bags correctos y código de salida 1. Después restauré el valor esperado y comprobé que la CI volvía a verde. |
| Bag(s) que añadí a `esperado.json` y por qué | Añadí `mi_rango` con salida esperada 0 y sin errores porque `analiza_bag.py` no detecta los valores fuera de rango, aunque `comprobar_rangos` sí identifica el error `RANGO_FUERA`. También incorporé `mi_fallo_b` con salida esperada 0 y sin errores, porque la herramienta original devuelve `TODO_OK` ante los comandos duplicados. Ambos casos permiten documentar limitaciones de la herramienta y justificar nuevas comprobaciones. |

Si **no** has podido usar GitHub, escribe aquí por qué y adjunta en `evidencias/` el `git_log.txt`, la salida de `simula_ci.py` y las capturas `ci_rojo` / `ci_verde` de la simulación local: la nota máxima del ejercicio baja (ver enunciado).

## Memoria conceptual (con tus palabras; máximo 1 página en total)

**M1.** Familias de fallo de arquitectura (estructural, dinámico, de tiempo, de origen aguas arriba): un ejemplo de cada una y la herramienta más barata que la detectaría. Un fallo estructural sería un topic mal nombrado o inexistente, detectable con `ros2 topic list -t`. Un fallo dinámico sería un filtro que procesa menos mensajes de los que recibe, detectable con `ros2 topic hz`. Un fallo de tiempo sería un nodo con el reloj desfasado, que podría identificarse comprobando `use_sim_time`, `/clock` y las marcas temporales. Un fallo de origen aguas arriba sería un sensor que deja de publicar durante varios segundos; se detectaría observando los huecos en `/distancia` y comprobando dónde aparecen primero.

**M2.** ¿Por qué una herramienta de análisis necesita pruebas unitarias, y qué significa que «un fallo no detectado es una especificación pendiente»? Las pruebas unitarias permiten comprobar que las funciones detectan correctamente los errores y que los cambios futuros no estropean lo que ya funcionaba. En el ejercicio 3, `analiza_bag.py` devolvía `TODO_OK` ante valores fuera del rango físico del sensor. Esto no significaba que los datos fueran correctos, sino que faltaba una comprobación. Por eso, un fallo no detectado indica que debemos definir un requisito nuevo, implementarlo y añadir pruebas que verifiquen su funcionamiento.

**M3.** ¿Qué ocurre exactamente en un conflicto de fusión en Git, cómo se ve en el fichero y cómo se resuelve? ¿Por qué no se debe «aceptar el mío» sin leer? Un conflicto aparece cuando dos ramas modifican una misma parte de un archivo de manera incompatible y Git no puede decidir qué versión conservar. En el fichero aparecen marcadores como `<<<<<<< HEAD`, `=======` y `>>>>>>> nombre-rama`, que separan los cambios de ambas versiones. Para resolverlo hay que revisar los dos cambios, decidir qué contenido debe permanecer, eliminar los marcadores, comprobar que el archivo funciona y confirmar la resolución con un commit. No se debe aceptar automáticamente la versión propia porque podríamos eliminar correcciones importantes realizadas en la otra rama.

**M4.** ¿Qué garantiza y qué NO garantiza una CI en verde? Usa como ejemplo el contrato de regresión con bags. Una CI en verde garantiza que las comprobaciones configuradas se han ejecutado correctamente para ese commit. En nuestro caso, significa que las pruebas unitarias y la regresión de los bags coinciden con los resultados declarados en `esperado.json`. Sin embargo, no garantiza que el programa esté libre de errores ni que detecte todos los fallos posibles. Por ejemplo, `mi_rango` contenía valores fuera de los límites físicos y la herramienta original devolvía `TODO_OK`. La CI podía estar en verde porque el contrato todavía no contemplaba esa anomalía. Por eso es necesario ampliar tanto las comprobaciones como sus pruebas cuando descubrimos un fallo nuevo.

## Opcional — `ros2 bag play` y RViz2

No realizado

## Reflexión final (5 líneas)

¿Qué fue lo más difícil? ¿Qué cambiarías en la herramienta? ¿Qué le faltó al bag para estar seguro de la causa? 
Lo más difícil fue diferenciar los síntomas observados de las causas reales de los fallos.
También resultó complicado identificar los errores que la herramienta original no era capaz de detectar.
Añadiría comprobaciones de valores fuera de rango, mensajes duplicados y frecuencias superiores a las esperadas.
Para identificar la causa exacta, los bags deberían incluir más información sobre parámetros, QoS y registros de los nodos.
La práctica me ha permitido comprender la importancia de combinar las evidencias, las pruebas unitarias y la integración continua.
