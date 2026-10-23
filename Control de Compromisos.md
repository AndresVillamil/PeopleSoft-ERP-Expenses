# Control de Compromisos y excepciones (TA y hoja de gastos), en PeopleSoft Expenses

En PeopleSoft Expenses, el Control de Compromisos (Commitment Control) y su gestión de excepciones es el mecanismo que valida los gastos reales y proyectados frente a un presupuesto financiero predefinido en la organización.
Se aplica principalmente sobre dos documentos clave:

- TA (Travel Authorization / Autorización de Viaje): Gastos estimados y planificados antes de realizar un viaje.
- Hoja de Gastos (Expense Report / Informe de Gastos): Los gastos reales que el empleado ya realizó y que solicita reembolsar o registrar.

A continuación se detalla en qué consiste cada concepto dentro del módulo:
---

1. ¿Qué es el Control de Compromisos?

Es un proceso de verificación presupuestaria automatizado (conocido como Budget Checking). Su función varía según el tipo de documento:
- En Autorizaciones de Viaje (TA): Funciona como una reserva de fondos o precompromiso (Encumbrance). El sistema aparta el dinero del presupuesto del departamento para asegurar que habrá fondos suficientes una vez que se haga el viaje.
- En Hojas de Gastos: Registra el gasto real (Expense). Si previamente existía una TA ligada, el sistema libera el fondo que estaba retenido (precompromiso) y lo convierte en un gasto oficial definitivo de la cuenta.


2. ¿Qué son las Excepciones?

Las excepciones ocurren cuando un documento (TA o Hoja de Gastos) falla alguna de las reglas de validación del presupuesto al ejecutar el Budget Checking. El sistema asigna un estado de línea específico y divide las excepciones en dos grandes categorías:

🚫 Errores (Error / Código E1-E99)

Son fallas críticas. El sistema bloquea el documento impidiendo que continúe el flujo de aprobación o de pago hasta que se resuelva la situación. Los motivos más comunes son:
- Exceder el presupuesto: El departamento no tiene dinero suficiente para cubrir el viaje o los gastos ingresados (Exceeds Control Budget).
- Presupuesto no existente: La combinación de segmentos contables (ChartFields) o el año fiscal introducido no se encuentra registrado en el sistema (No Budget Exists).
- Presupuesto cerrado o en espera: El fondo contable está bloqueado de forma temporal o definitiva por la administración.

⚠️ Advertencias (Warning / Código W1-W64)

Son alertas informativas. El sistema detecta una anomalía (por ejemplo, el gasto entra dentro de un margen de tolerancia permitido o hay discrepancias de fechas menores), pero sí permite que la transacción se registre y continúe su flujo.

3. ¿Cómo se gestionan estas Excepciones?

Cuando una TA o una Hoja de Gastos queda retenida por un error presupuestario, los administradores, aprobadores o usuarios con permisos especiales tienen las siguientes opciones para solucionar la excepción:

1. Modificar el documento: Editar los montos de la hoja de gastos/TA, cambiar las fechas, o corregir los códigos de cuenta (ChartFields) erróneos, y volver a procesar la verificación.
2. Ajustar el presupuesto: Solicitar al área financiera una transferencia de fondos o una ampliación presupuestaria para la cuenta afectada.
3. Anulación manual (Override): Si el usuario cuenta con los permisos de seguridad adecuados, puede ingresar a la pantalla de Excepciones de Control de Compromisos y forzar la aprobación de la transacción a pesar del error de fondos.

Si tienes un documento específico retenido en este momento, indícame qué código o mensaje de error te aparece (por ejemplo, No Budget Exists o Exceeds Budget Tolerance) para poder guiarte paso a paso en su solución.

<img width="846" height="326" alt="image" src="https://github.com/user-attachments/assets/0836dc73-10aa-4525-be81-8631b41dfae7" />

En la imagen adjunta se observa la pantalla de Excepciones de Hoja de Gastos de PeopleSoft. El informe de gastos afectado tiene el ID Informe: 0000002343 y presenta un bloqueo crítico.
A continuación, te explico el diagnóstico del error y cómo resolverlo:

Diagnóstico del Error

El sistema muestra dos excepciones de tipo Error con los siguientes datos comunes:
- Business Unit: ACHCO
- Grupo Contable: CC_GTO_CTR
- Excepción: Presupuesto Inexistente
Este error específico significa que no hay un presupuesto configurado o abierto en el módulo de Control de Compromisos (Commitment Control) para la combinación de códigos contables (ChartFields), el año fiscal o el periodo que se introdujo en las líneas de esta hoja de gastos. El sistema no puede validar si hay o no dinero porque la estructura del presupuesto para esa cuenta directamente no existe.

---
Pasos para Resolverlo

Tienes tres rutas posibles para corregir este problema, dependiendo de tus permisos en el sistema:

Opción 1: Validar y corregir los códigos contables (ChartFields)

Es muy común que el error ocurra simplemente por haber seleccionado un departamento, proyecto, cuenta o fondo equivocado que no está presupuestado.
1. Haz clic en la pestaña "Claves Contables Ppto" (junto a Sustitución Ppto). Ahí verás exactamente qué cuentas contables se ingresaron en las líneas 1 y 2.
2. Si ves algún código erróneo, regresa a la Hoja de Gastos 0000002343, corrige la contabilidad con los códigos válidos de tu área y vuelve a ejecutar el proceso de Budget Checking.

Opción 2: Solicitar la creación o apertura del presupuesto (Vía Finanzas)

Si los códigos contables ingresados son 100% correctos y válidos para este gasto:
1. Deberás contactar al administrador de presupuestos o al equipo de Finanzas/Gl de tu organización.
2. Infórmales que necesitas que se cree o se active el presupuesto en el Grupo Contable CC_GTO_CTR para la Business Unit ACHCO bajo los códigos contables que aparecen en este reporte.
3. Una vez que ellos lo den de alta o abran el periodo presupuestario, podrás volver a procesar tu informe de gastos.

Opción 3: Forzar la transacción (Anulación / Override manual)

Si cuentas con un perfil de aprobador financiero o administrador con permisos elevados de Control de Compromisos, puedes saltarte el bloqueo directamente desde esta pantalla:
1. En la tabla inferior, marca la casilla de verificación bajo la columna "Sustituir Presupuesto" para ambas líneas con error.
2. Alternativamente, puedes marcar la casilla superior "Sustituir Transacción" para autorizar el reporte completo.
3. Guarda los cambios. El estado cambiará a "Anulado" (Overridden) y el documento podrá continuar su flujo normal de aprobación y pago.


---
Como usuario que reporta el gasto, no tienes los permisos en el sistema para forzar el presupuesto (Override) ni para crear cuentas financieras. Tu objetivo es corregir el documento o escalar el problema al área correcta.
Sigue estos pasos ordenados para solucionar el problema:

1. Revisa si te equivocaste de cuenta o centro de costos

Es la causa más común de este error. Si ingresaste un código incorrecto, el sistema busca un presupuesto que no existe.
• Haz clic en la pestaña "Claves Contables Ppto" que ves en tu pantalla.
• Anota los códigos que aparecen allí (como Departamento, Cuenta, Proyecto o Fondo).
• Compara esos códigos con los que te autorizó tu jefe o el área de finanzas para este viaje o gasto.

2. Cómo corregir la Hoja de Gastos (Si el código estaba mal)

Si confirmas que te equivocaste en un número o código:
1. Sal de esa pantalla de excepciones e ingresa a Modificar Hoja de Gastos (Modify Expense Report).
2. Busca tu informe 0000002343.
3. Ve a los detalles de las líneas 1 y 2, y corrige los campos contables (ChartFields) con los datos correctos.
4. Guarda los cambios y vuelve a presionar el botón de Verificación de Presupuesto (Budget Checking) o vuelve a enviar el informe para que el sistema se actualice.

3. Qué hacer si los códigos están bien (Escalar a Finanzas)

Si revisas los códigos y estás 100% seguro de que son los que te corresponden, significa que el área de Finanzas olvidó cargar o abrir el presupuesto para este año/periodo en el sistema.
Debes levantar un ticket de soporte o enviar un correo al administrador de presupuestos / área de Finanzas de tu empresa con los siguientes datos exactos:
"Hola equipo de Finanzas/Soporte. Mi Hoja de Gastos ID: 0000002343 se encuentra retenida por un error de Presupuesto Inexistente. El error ocurre en la Business Unit ACHCO y Grupo Contable CC_GTO_CTR. Los códigos contables que estoy utilizando son [Inserta aquí los códigos de Cuenta/Departamento que viste en la pestaña]. ¿Podrían por favor verificar si hace falta activar o cargar este presupuesto en el sistema?"
Una vez que ellos configuren el presupuesto del lado financiero, te avisarán para que entres de nuevo a tu reporte, ejecutes la verificación y puedas enviarlo a aprobación normalmente.
Si quieres que verifiquemos si los códigos se ven bien, haz clic en la pestaña Claves Contables Ppto, toma una captura y compártela. ¿Sabes si este gasto corresponde a un proyecto específico o a tu departamento habitual?

<img width="881" height="695" alt="image" src="https://github.com/user-attachments/assets/e5722e14-19d9-44cc-94af-180c1c7dd1cf" />

---
Al realizar la búsqueda en Detalles de Presupuesto, el sistema arroja el mensaje en la parte inferior: "No hay valores coincidentes".
Esto significa que la combinación exacta de códigos contables (ChartFields) que estás intentando cargar en tu hoja de gastos no tiene un presupuesto activo o creado en el sistema para ese periodo específico.
Analizando los criterios de tu búsqueda, el conflicto principal se debe a uno de los siguientes dos motivos:
1. Error en el Periodo Presupuesto (2026M09): Estás intentando registrar el gasto en el periodo Septiembre de 2026. Es muy probable que el equipo de Finanzas aún no haya cargado de forma oficial los techos presupuestarios para este mes en el grupo CC_GTO_CTR, o bien, que la fecha de tu gasto esté cayendo en un mes incorrecto por error de dedo al digitar el informe.
2. Combinación errónea de códigos: La combinación de la Cuenta (5190350005), Departamento (1000400100) y la Referencia de Presupuesto (ON0177) no está dada de alta en conjunto.

¿Qué debes hacer ahora?

Como eres el usuario que reporta, debes escalar esto de inmediato con Finanzas o el Administrador de Presupuestos, ya que la solución depende completamente de ellos.
Envía un correo o abre un ticket adjuntando estas capturas de pantalla y copia el siguiente texto:
*"Buen día. Estoy intentando enviar mi Hoja de Gastos, pero el sistema me arroja un error de 'Presupuesto Inexistente'. Al consultar los Detalles de Presupuesto para validar la combinación, el sistema indica que 'No hay valores coincidentes'.
Los datos ingresados son:
- Business Unit: ACHCO
- Grupo Contable: CC_GTO_CTR
- Cuenta: 5190350005
- Departamento: 1000400100
- Referencia Presupuesto: ON0177
- Periodo Presupuesto: 2026M09

¿Podrían verificar si hace falta abrir este periodo presupuestario o habilitar esta combinación de claves contables para que pueda procesar mi informe? Quedo atento a su confirmación."*

Una alternativa rápida que puedes verificar tú mismo:

Antes de enviar el correo, regresa a tu Hoja de Gastos (0000002343) y revisa la fecha de los recibos/gastos que pusiste en las líneas 1 y 2. Si por error escribiste una fecha de septiembre y el gasto era de otro mes (por ejemplo, agosto), cámbiala a la fecha correcta. Al guardarlo, el sistema cambiará el periodo (ej. a 2026M08) y es posible que ese mes sí tenga presupuesto activo.

Si la fecha de tus recibos es realmente de septiembre, no hay duda de que Finanzas debe activarlo. ¿Esta combinación de Departamento (1000400100) y Referencia (ON0177) es la que usas normalmente todos los meses sin problema?

---
### Procedimiento que debe seguir el Administrador o quien tenga los permisos adecuados

A continuación, se detalla el procedimiento técnico estándar en PeopleSoft Financials (Commitment Control) para que el Administrador de Presupuestos o el usuario con permisos adecuados resuelva esta excepción.
Este perfil cuenta con dos caminos: la solución definitiva (corregir o cargar el presupuesto faltante) o la solución rápida (autorizar la excepción manualmente si el gasto es válido).

Procedimiento para el Administrador de Control de Compromisos


# Opción A: Solución Definitiva (Cargar o Modificar el Presupuesto)

Si la combinación de ChartFields es válida y el gasto debe proceder contra el presupuesto de la organización, el administrador debe habilitar los fondos en el sistema.
1. Navegación en PeopleSoft:
- Ve a: Control de compromisos (Commitment Control) > Presupuestos de Terceros (Budget Journals) > Escribir asientos de presupuesto (Enter Budget Journals).

2. Crear un nuevo Asiento de Presupuesto:
- Haz clic en la pestaña Añadir un nuevo valor.
- Selecciona la Business Unit (ACHCO) y el Grupo Contable (CC_GTO_CTR).
- En tipo de asiento, selecciona Original (si es un presupuesto nuevo) o Ajuste (si se van a transferir fondos).

3. Ingresar las Líneas del Presupuesto:
- En la pestaña de líneas, introduce la combinación exacta que arrojó el error:
		- Cuenta: 5190350005
		- Departamento: 1000400100
		- Referencia Presupuesto: ON0177
		- Periodo Presupuesto: 2026M09
- Asigna el Monto de presupuesto necesario en la línea.

4. Procesar y Postear el Asiento:
- En el menú desplegable de procesos del asiento, selecciona Verificación de Presupuesto (Budget Check) y haz clic en Ejecutar.
- Una vez verificado exitosamente, cambia el proceso a Postear Asiento (Post Journal) para que impacte el libro mayor de Commitment Control.

5. Notificar al usuario: Informa al empleado para que vuelva a ingresar a su reporte de gastos y ejecute el Budget Checking.

# Opción B: Solución Rápida (Anulación manual / Override)

Si el presupuesto no se va a cargar en el sistema inmediatamente, pero el área financiera autoriza que este gasto específico se procese, se puede forzar la aprobación desde la pantalla de excepciones.

1. Navegación en PeopleSoft:
- Ve a: Gastos (Travel and Expenses) > Gestionar Gastos (Manage Expenses) > Excepciones control compromisos (Commitment Control Exceptions) > Hoja de Gastos (Expense Report).

2. Buscar el Documento Afectado:
- Introduce el ID Informe: 0000002343 y haz clic en Buscar.

3. Aplicar la Anulación (Override):
- Para el documento completo: En la parte superior, marca la casilla Sustituir Transacción (Override Transaction).
- Para líneas específicas: En la tabla inferior Presupuesto con Excepciones, ve a las líneas 1 y 2 que muestran "Presupuesto Inexistente" y marca de forma individual la casilla bajo la columna Sustituir Presupuesto (Override Budget).

4. Guardar los Cambios:
- Haz clic en el botón Guardar en la parte inferior de la página. El estado de la excepción cambiará automáticamente de Error a Anulado (Overridden).

5. Liberar el trámite: Con esto, la Hoja de Gastos se desbloquea al instante y continuará de manera normal hacia el siguiente aprobador o la fila de pago de Tesorería.

