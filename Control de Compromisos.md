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

