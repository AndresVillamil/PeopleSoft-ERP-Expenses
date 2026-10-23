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
