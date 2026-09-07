# Context Engineering Experiment

En este experimento, se evaluarán tres comportamientos y resultados a un mismo proyecto (A, B y C) generados por el mismo agente de codificación (Antigravity CLI) con la finalidad de determinar qué ocurre cuando se especifica el contexto en tres diferentes niveles (Minimal Context, Repository Context, Engineered Context). 

## Hypothesis

La especificación de los prompts en los experimentos A, B y C tendrán un impacto diferente en la forma en que el agente ejecuta la tarea de corregir el código y agregar la funcionalidad de actualización de correo del cliente. Se espera que entre más específico se convierta el prompt, el agente implementa la tarea con mayor éxito, satisfaciendo los requisitos esperados usando la menor cantidad de código posible. El resultado más satisfactorio será el experimento C con la especificación del SPEC.md y AGENTS.md en el prompt (Engineered Context); por otro lado, el experimento con los resultados menos satisfactorios será el A debido a que el prompt solo provee el contexto menor posible (Minimal Context). 

## Experimental Setup

Para este experimento, se creó un proyecto de software en donde se manejan la operaciones de correo para un conjunto de clientes, junto con las pruebas de Python (pytest) para asegurar que el proyecto cumple con lo requerido. Para que se implemente cada uno de los tres experimentos se subirá el proyecto a un repositorio en el Github con un commit de base línea para regresar a la normalidad y volver a realizar tantos experimentos como sea necesario.

Dentro de los archivos, se presentan algunos errores en las pruebas: el primer error ocurre cuando se trata de transformar a minúscula el correo dado por el cliente; el segundo error ocurre cuando se acepta un correo inválido debido a que no se revisa su sintáctica; el tercer error ocurre cuando se trata de actualizar el correo. 

El objetivo del agente de codificación será reconocer estos errores e implementar la nueva funcionalidad de actualización para cumplir con los requisitos del proyecto. Para ello, se le dará en cada experimento un prompt específico y archivos (SPEC.md, AGENTS.md) conforme se avance en los experimentos.

## A — Minimal Context

Prompt: Implement the customer email update functionality.

Inspect the repository first. Implement the necessary changes and run the tests.

Results:

Corrigió los errores mencionados: para convertir el correo del cliente a minúscula,  el agente empleó la función lower(); para que no se introduzcan correos inválidos, el agente introdujo una expresión regular que presentaba la sintáctica correcta y en customer.py revisó que el correo coincida con la expresión usando la función match(); por último, para actualizar el correo correctamente el agente hizo una llamada en repository.py a la función update\_customer\_email() y regresó el valor actualizado del cliente a través de la variable “updated”.

Finalmente, el agente agregó dos pruebas de repositorio nuevas, una para probar el éxito de la actualización de correo (ej. el cliente tiene el nuevo correo actualizado) y otra para probar que ocurre si el correo es inválido.

Human intervention:

|Terminal o archivos donde se intervino|Acción|
| :- | :- |
|Terminal|<p>Rechazar un comando que publicaba los cambios al repositorio master de inmediato </p><p></p>|
|repository.py|Corregir repository.py para verificar que haya un string (actor) en la variable “updated\_by”|

Score: 95

Observations:

- Agregó un test similar a uno de los tests base.
- No se revisa si hay un string (actor) en la variable “updated\_by” en repository.py

## B — Repository Context

Prompt: Implement the customer email update functionality.

Before making changes:

1. Inspect the repository.
2. Read README.md.
3. Inspect all relevant source files.
4. Inspect the tests.
5. Infer expected behavior from the code and tests.
6. Run tests before changing code.
7. Make the smallest necessary implementation.
8. Run tests again.
9. Explain which repository information influenced the implementation.

Results:

Corrigió los errores mencionados: para convertir el correo del cliente a minúscula,  el agente empleó la función lower(); para que no se introduzcan correos inválidos, el agente introdujo en customer.py una condición en el que si “@” no está en el nuevo correo, se rechaza; por último, para actualizar el correo correctamente el agente hizo una llamada en repository.py a la función update\_customer\_email() y regresó el valor actualizado usando la misma llamada a update\_customer\_email().

A diferencia del experimento A, el agente no agregó pruebas nuevas.

Human intervention:

|Terminal o archivos donde se intervino|Acción|
| :- | :- |
|customer.py|Agregar una validación de correo más fuerte en customer.py|
|repository.py|<p>Corregir repository.py para verificar que haya un string (actor) en la variable “updated\_by”</p><p></p>|

Score: 92.1

Observations:

- La validación del correo electrónico es muy débil (ejemplo: se acepta @gmail.com)
- No se revisa si hay un string (actor) en la variable “updated\_by” en repository.py

## C — Engineered Context

Prompt: Implement the customer email update functionality.

Follow SPEC.md and AGENTS.md.

Inspect the repository first, run tests before and after changes, and explain your verification.

Results:

Corrigió los errores mencionados de manra similar al experimento A: para convertir el correo del cliente a minúscula,  el agente empleó la función lower(); para que no se introduzcan correos inválidos, el agente introdujo una expresión regular que presentaba la sintáctica correcta y en customer.py revisó que el correo coincida con la expresión usando la función match(); por último, para actualizar el correo correctamente el agente hizo una llamada en repository.py a la función update\_customer\_email() y regresó el valor actualizado usando la misma llamada a update\_customer\_email().

A diferencia del experimento A, el agente no agregó pruebas nuevas.

Human intervention:

|Terminal o archivos donde se intervino|Acción|
| :- | :- |
|repository.py|<p>Corregir repository.py para verificar que haya un string (actor) en la variable “updated\_by”</p><p></p>|

Score: 99

Observations:

- No se revisa si hay un string (actor) en la variable “updated\_by” en repository.py

## Comparative Results

El experimento C (Engineered Context) produjo el mejor resultado obteniendo un score de 99/100, superando a A (95/100) y B (92.1/100). Además, C terminó con 4 pruebas pasando y 0 pruebas fallando, cumplió los 7 requisitos registrados y realizó el menor tiempo de ejecución total, con 116 segundos

La principal razón de su mejor desempeño fue que el agente no recibió únicamente instrucciones sobre qué hacer, sino también información explícita sobre qué comportamiento debía preservar y cómo validar que la tarea estaba terminada. SPEC.md definió requisitos concretos como mantener el customer\_id, preservar created\_by, establecer correctamente updated\_by y rechazar correos sintácticamente inválidos.

Por otra parte, AGENTS.md estableció reglas de ejecución: inspeccionar antes de editar, realizar cambios mínimos, no modificar pruebas, ejecutar pytest antes y después y considerar la tarea terminada solamente cuando las pruebas y la SPEC.md fueran satisfechas.

## Error Analysis

El error más claro que apareció en el experimento A y no en C fue la incorporación de una prueba adicional similar a una prueba que ya existía en el proyecto base. En el experimento C, el agente no agregó nuevas pruebas.

Sin embargo, existe una consideración importante: los experimentos A y C sí compartieron un problema relacionado con updated\_by. En ambos casos, fue necesaria una intervención humana para corregir repository.py y garantizar que updated\_by contuviera un string correspondiente al actor que realiza la actualización. Por lo tanto, sería incorrecto afirmar que C eliminó todos los errores de A.

La diferencia adicional está en el experimento B, el cual presentó una validación de correo demasiado débil, pues simplemente comprobaba la presencia de @, permitiendo casos como @gmail.com. Esto no ocurrió en el experimento A y C, donde se utilizó una expresión regular para validar la sintaxis.

En resumen, C redujo los errores de implementación y produjo una solución más alineada con los requisitos, aunque todavía necesitó intervención humana para completar la validación de updated\_by.

## Context Quality Analysis

La información más útil para proveer el contexto del prompt fue la que describía comportamientos verificables y restricciones concretas con respecto al proyecto de software, especialmente: qué debe ocurrir cuando el correo es válido o inválido; el correo debe quedar en minúsculas; el cliente debe existir; el customer\_id no debe cambiar; el created\_by debe conservarse y el updated\_by debe identificar al actor.

Estos puntos aparecen explícitamente en el archivo SPEC.md el cual aportó el "qué" de la solución: definió el objetivo, los requisitos y los criterios de aceptación. Esto redujo la necesidad de que el agente tuviera que inferir el comportamiento exclusivamente a partir del código, las pruebas y el prompt dado.

AGENTS.md aportó principalmente el "cómo" trabajar: estableció reglas sobre inspección, cambios mínimos, preservación de interfaces, no modificar tests y verificación mediante pruebas.

El experimento también muestra que más contexto no significa necesariamente mejor contexto. En el experimento B, se recibió más instrucciones de inspección que el A, pero obtuvo un resultado inferior de 92.1 frente a 95 en el Context Engineering Score debido a que la validación del correo fue débil en comparación del A; esto pudiera haber sido causado por agregar una restricción en el prompt que le pedía poner solo el código necesario. Igualmente, el contexto dado en el prompt del B contiene información potencialmente redundante como la instrucción de inspeccionar diferentes elementos del repositorio cuando parte de esa información podía obtenerse mediante una inspección general. Por ejemplo, B especificó individualmente leer README.md, archivos fuente relevantes y tests, mientras que C utilizó documentos especializados que concentraban las restricciones importantes.

Ante todo lo anterior, el aprendizaje principal es que la calidad del contexto depende más de su relevancia, estructura y capacidad para reducir ambigüedad que de su cantidad. No solomente se depende del prompt para ofrecer el contexto al agente, sino también de la documentación que la rodea (README.md, SPEC.md, AGENTS.md, etc.)

## Conclusions

Al realizar el experimento, se descubrió que el agente C tuvo el mejor desempeño cuando recibió una especificación clara y reglas explícitas de trabajo realizadas por una persona detrás del monitor lo cual indica que utilizar un agente de código no elimina la responsabilidad del desarrollador, pues este aún debe definir claramente el objetivo, establecer restricciones, proporcionar el contexto relevante y verificar que el resultado realmente cumple con lo esperado. 

Un desarrollador que utiliza agentes necesita aprender Context Engineering, y no solamente mejores prompts, porque un prompt describe principalmente una solicitud, mientras que el contexto puede definir requisitos, arquitectura, restricciones, criterios de aceptación, reglas de modificación y procedimientos de validación. En este experimento, SPEC.md y AGENTS.md ayudaron a convertir información dispersa en instrucciones estructuradas para el agente.

Por lo tanto, el desarrollador no debe limitarse a decirle al agente qué programar. También debe diseñar las condiciones bajo las cuales el agente puede tomar buenas decisiones y establecer cómo comprobar que esas decisiones son correctas.

## What I Would Change

Una de las cosas que cambiaría sería el SPEC.md para hacer algunos requisitos todavía más precisos y verificables. Por ejemplo, actualmente indica que el correo debe ser "sintácticamente válido", pero sería mejor especificar qué estructura mínima debe aceptar y rechazar. Esto es especialmente relevante porque el experimento B demostró que una validación basada únicamente en @ es insuficiente.

Finalmente agregaría criterios de aceptación explícitos para updated\_by, por ejemplo: updated\_by debe ser un str, no debe ser vacío, el valor recibido debe conservarse como el actor de la modificación. Esto habría atacado directamente con el problema que apareció en los tres experimentos.
