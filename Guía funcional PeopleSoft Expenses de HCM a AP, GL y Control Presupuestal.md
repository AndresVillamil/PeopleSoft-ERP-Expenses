# Guía funcional PeopleSoft Expenses: de HCM a AP, GL y Control Presupuestal

Oct 23, 2026 · @Andres

## 1. Objetivo y alcance

Esta guía acompaña la capacitación funcional del módulo Expenses de PeopleSoft FSCM 9.1 y recorre el ciclo completo: carga de empleados desde HCM, autorización de viaje, anticipo de caja, informe de gastos, aprobación, pago por EFT en Cuentas por Pagar (AP) y contabilización en General Ledger (GL).

El objetivo práctico es que cada participante pueda reproducir en un ambiente de pruebas los casos de uso UC01 a UC38 y validar el flujo de punta a punta.

- **Dentro del alcance:** integración con HCM, Travel Authorization, Cash Advance, Expense Report, AWE, Commitment Control, Stage Payments, Pay Cycle por EFT, Post Payments y Journal Generator.
- **Fuera del alcance:** configuración técnica de la interfaz bancaria, personalizaciones y otros métodos de pago distintos de EFT.

**Cómo usar la guía.** Siga los módulos en orden, porque cada uno deja datos que el siguiente consume. Cada módulo indica el objetivo, la ruta de menú, los pasos, los controles a validar y los casos de uso asociados.

> Las rutas de menú y los nombres de campos corresponden a la navegación estándar de FSCM 9.1. Valídelos en su instancia, porque pueden variar por personalización o por versión de paquete de mantenimiento (PUM).

## 2. Flujo de punta a punta, roles y datos de prueba

**Expenses no paga.** Cuando un informe de gastos o un anticipo queda aprobado, Expenses lo deja en tablas de staging con el proceso Stage Payments (EX\_TRAN\_PRCS). El Pay Cycle de AP toma esos pagos por la interfaz de pagos de Payables, sin crear vouchers, y los liquida por EFT. Después Expenses ejecuta Post Payments y el Journal Generator lleva los asientos a GL.

| Paso | Qué se ejecuta | Quién | Resultado |
| --- | --- | --- | --- |
| 1 | Configuración de BU, tipos de gasto, aprobaciones y seguridad | Administrador Expenses | Ambiente listo |
| 2 | Carga de empleados, organización y bancos desde HCM | Administrador / integración | Perfiles de empleado válidos |
| 3 | Autorización de Viaje, Anticipo de Caja, Informe de Gastos y envío | Empleado | Transacciones enviadas |
| 4 | Aprobación | Gerente / aprobador | Transacciones aprobadas |
| 5 | Stage Payments y Post Liabilities | Administrador Expenses | Pagos en staging y pasivos contabilizados |
| 6 | Pay Cycle en AP, pago EFT, archivo al banco | Analista de AP | Pago EFT generado |
| 7 | Post Payments y conciliación | Administrador / Tesorería | Pago reflejado en Expenses y conciliado |
| 8 | Journal Generator, Edit, Post y consulta en GL | Analista de GL | Diarios posteados |
| 9 | Control de Compromisos y excepciones (TA y hoja de gastos) | Presupuesto / Administrador | Excepciones resueltas |
| 10 | Consulta final del empleado | Empleado | Estado y pago verificados |

### Usuarios y datos de prueba requeridos

| Rol de prueba | Qué debe tener |
| --- | --- |
| Empleado (viajero) | Perfil válido, supervisor, cuenta bancaria y tarjeta corporativa asociada |
| Gerente / aprobador | Asignado como aprobador del empleado (Approver Assignment) |
| Administrador de Expenses | Acceso a Travel and Expenses > Process Expenses y a Set Up |
| Analista de AP | Acceso a Pay Cycle Manager; pay cycle con fuente Expenses y método EFT |
| Analista de GL | Acceso a Journals (Generate, Edit, Post) |
| Tesorería / Banco | Cuenta bancaria origen, layout EFT y extracto de prueba |

## 3. Módulo 1: integración con HCM y perfil del empleado

Expenses no mantiene la fuente de verdad del empleado: la recibe de HCM. Un perfil incompleto es la causa más común de que una transacción no se pueda crear, aprobar o pagar.

**Datos que viajan desde HCM**

- Datos personales: nombre, identificación, correo.
- Datos del puesto: departamento, ubicación, supervisor, moneda base, estado activo.
- Cuentas bancarias: banco, cuenta y método de pago EFT.

**Cómo llegan.** Mediante componentes de integración de negocio (Component Interfaces) o mensajes de sincronización, según la implementación. La carga inicial es completa y luego se actualiza por cambios.

**Perfil organizacional del empleado (EX\_EE\_PROFILE2).** Ruta típica: Travel and Expenses > Employee Profile. Se revisan y completan:

1. Datos organizacionales: Business Unit de Expenses, departamento y ChartFields por defecto.
2. Preferencias de pago: método EFT y cuenta bancaria vigente.
3. Aprobadores asignados (Approver Assignment) y tarjeta corporativa asociada.
4. Moneda y reglas de gasto aplicables al empleado.

**Controles a validar**

- El empleado aparece activo y con supervisor.
- La cuenta bancaria está marcada como cuenta de pago por EFT.
- Una actualización en HCM (por ejemplo, cambio de departamento) se refleja en el perfil.

**Casos de uso:** UC01 a UC06.

## 4. Módulo 2: Autorización de Viaje y Control Presupuestal

La Autorización de Viaje (Travel Authorization, TA) es la solicitud previa que estima el gasto del viaje y, si Commitment Control está activo, reserva presupuesto antes de que el empleado viaje.

**Crear y modificar una TA.** Ruta típica: Travel and Expenses > Travel Authorization > Create/Modify.

1. Ingrese el empleado y el encabezado: propósito, fechas, destino y Business Unit.
2. Agregue líneas de gasto estimado: tipo de gasto, monto, moneda y ChartFields.
3. Guarde para más tarde o ejecute la verificación presupuestal.
4. Envíe la TA a aprobación (Submit).
5. Para modificar, abra la TA desde Modify; una TA aprobada que cambia vuelve a aprobación.

**Cómo funciona la validación presupuestal**

La TA es una transacción fuente de Commitment Control. Al verificar presupuesto, el sistema compara el monto estimado contra el presupuesto disponible de cada combinación de ChartFields.

| Elemento | Qué hace |
| --- | --- |
| Budget Check online | Se ejecuta desde la propia TA y responde al instante: válido o con excepción |
| Budget Check por lotes | Procesa muchas TAs con el Budget Processor (FS\_BP) |
| Compromiso (encumbrance) | Reserva presupuesto por el monto estimado; se libera al liquidar la TA contra el informe de gastos |
| Budget Exceptions | Registro de líneas que no pasaron la validación, con el motivo |

**Gestionar excepciones.** Ruta típica: Commitment Control > Review Budget Check Exceptions. Para cada excepción:

1. Identifique la causa: presupuesto insuficiente, ChartField sin control o período cerrado.
2. Corrija en la TA (monto o ChartFields) o solicite ajuste de presupuesto.
3. Vuelva a ejecutar el Budget Check hasta que el estado sea válido.

**Controles a validar:** el compromiso aparece en la consulta de Commitment Control; una TA con excepción no avanza a aprobación hasta corregirse.

**Casos de uso:** UC07 a UC13.

## 5. Módulo 3: Anticipo de Caja

El anticipo entrega fondos al empleado antes del gasto. Se asocia a una TA vigente y se amortiza después contra el informe de gastos.

**Crear un anticipo.** Ruta típica: Travel and Expenses > Cash Advance > Create/Modify.

1. Seleccione el empleado y la TA vigente a la que se asocia.
2. Ingrese monto, moneda, motivo y fecha requerida.
3. Confirme el método de pago EFT y la cuenta bancaria del empleado.
4. Envíe a aprobación.

**Modificar, eliminar y cerrar**

- Solo un anticipo en estado pendiente se puede eliminar; uno ya enviado se cancela o se devuelve desde la aprobación.
- Un anticipo pagado se modifica solo antes del pago; después se concilia contra el informe.
- Cierre y reconciliación: el anticipo se aplica en el informe de gastos; si sobra saldo, el empleado debe devolverlo, y si el gasto supera el anticipo, Expenses paga solo la diferencia.

**Ejemplo de amortización**

| Concepto | Monto |
| --- | --- |
| Anticipo pagado | 1.000.000 |
| Gastos reales del informe | 1.350.000 |
| Anticipo aplicado | 1.000.000 |
| Pago al empleado por EFT | 350.000 |

**Controles a validar:** el anticipo aprobado pasa por Stage Payments; el saldo del anticipo queda en cero tras aplicarlo.

**Casos de uso:** UC14 a UC17.

## 6. Módulo 4: Informe de Gastos

El informe de gastos (Expense Report) registra los gastos reales, aplica el anticipo, liquida la TA y dispara el pago al empleado.

**Crear un informe.** Ruta típica: Travel and Expenses > Expense Report > Create/Modify.

1. Ingrese el empleado y el encabezado: propósito, fechas y moneda.
2. Asocie la TA de origen, si el viaje la tiene.
3. Registre cada línea: fecha, tipo de gasto, monto, tipo de pago (efectivo, tarjeta corporativa o pagado por el empleado) y ChartFields.
4. Adjunte los recibos que exija el tipo de gasto.
5. Aplique el anticipo pendiente, si existe.
6. Revise el resumen y envíe a aprobación.

**Excepciones de hoja de gastos.** El sistema marca líneas que incumplen reglas, como gasto sobre el límite, recibo faltante o posible duplicado. El empleado las corrige o las justifica antes de enviar, y el aprobador las ve al revisar.

**Cierre del ciclo de gastos de viaje**

- Amortización del anticipo: se aplica contra los gastos reales y el neto define si se paga o si el empleado devuelve saldo.
- Liquidación de la TA: al asociar el informe final, la TA se liquida y el compromiso remanente se libera para presupuesto.

**Controles a validar:** total del informe = gastos menos anticipo aplicado; la TA queda cerrada y el saldo presupuestal liberado.

**Casos de uso:** UC18 a UC24.

## 7. Módulo 5: Flujo de aprobación (AWE)

El motor de flujo de aprobación (Approval Workflow Engine, AWE) decide quién aprueba cada transacción según reglas de monto, departamento o tipo. Las tres transacciones de Expenses, TA, anticipo e informe, usan el mismo motor con definiciones separadas.

**Cómo se define el flujo**

| Pieza | Función |
| --- | --- |
| Proceso de aprobación | Definición por transacción (TA, anticipo, informe) |
| Etapas, rutas y pasos | Orden de aprobación y condiciones de entrada |
| Listas de usuarios | Quiénes pueden aprobar en cada paso |
| Approver Assignment | Aprobador asignado a cada empleado |

**Operar una aprobación.** El aprobador entra a la página de aprobaciones de la transacción y puede:

- Aprobar, y la transacción avanza al siguiente paso o queda aprobada.
- Denegar, con motivo, y el empleado ve el rechazo.
- Devolver para corrección, y el empleado edita y reenvía.
- Reasignar a otro aprobador mediante la reasignación de trabajo, útil en vacaciones o ausencias.

**Controles a validar:** una transacción sin aprobador asignado queda detenida; el historial de aprobación muestra quién actuó y cuándo; lo aprobado queda disponible para Stage Payments.

**Casos de uso:** UC25 a UC29.

## 8. Módulo 6: Cierre financiero, AP por EFT y GL

Este módulo une Expenses con AP y GL. El orden importa: primero se prepara el pago, luego AP lo liquida y al final se contabiliza.

> **Modelo de pago adoptado:** el Pay Cycle toma los pagos directamente de la interfaz de pagos de Payables, sin crear vouchers. El temario menciona vouchers de reembolso en el punto 7.1.2; esta guía sigue el modelo sin vouchers descrito en la sección de pago. Conviene confirmar cuál aplica antes de dictar la sesión.

**Liquidación y pago**

1. **Stage Payments (EX\_TRAN\_PRCS).** Toma informes y anticipos aprobados y los deja en staging, listos para AP.
2. **Post Liabilities.** Genera las líneas contables de gasto y pasivo en Expenses.
3. **Pay Cycle en AP.** Ruta típica: Accounts Payable > Payments > Pay Cycle Manager. Con fuente Expenses y método EFT, selecciona, crea el pago y genera el archivo para el banco.
4. **Post Payments (Expenses).** Actualiza el estado de pago de las transacciones pagadas.
5. **Conciliación.** Se cruza el pago EFT contra el extracto del banco.

**Contabilización en GL**

- **Distribución contable.** Cada línea de gasto toma sus ChartFields del perfil del empleado o de la línea y define dónde se registra el gasto.
- **Asientos.** Gasto contra pasivo al contabilizar el informe; pasivo contra banco al contabilizar el pago.
- **Journal Generator (FS\_JGEN).** Crea los diarios desde las líneas contables; luego se editan y se postean en GL.

| Momento | Débito | Crédito |
| --- | --- | --- |
| Informe contabilizado (Post Liabilities) | Gasto por ChartFields | Pasivo con el empleado |
| Pago contabilizado (Post Payments) | Pasivo con el empleado | Banco |

**Controles a validar:** el monto del pago EFT coincide con el neto del informe; el diario posteado cuadra débitos y créditos; el empleado ve el informe como pagado.

**Casos de uso:** UC30 a UC38.

## 9. Parámetros de los procesos de gastos

Los nombres exactos de las páginas de petición de proceso varían por instalación; la tabla recoge los parámetros que se explican en la capacitación.

| Proceso | Parámetro | Para qué sirve | Error común |
| --- | --- | --- | --- |
| Stage Payments (EX\_TRAN\_PRCS) | Run Control ID | Identifica la petición y guarda los valores | Reutilizar un control con valores de otra prueba |
|  | Business Unit | Limita a la BU de Expenses | BU distinta a la del informe |
|  | Opción de proceso | Elige la acción: stage de pagos, pasivos o post de pagos | Ejecutar Post Payments antes de que AP pague |
|  | Empleado o ID de informe | Acota la selección a una o varias transacciones | Dejarlo vacío y procesar más de lo previsto |
|  | Fecha de pago | Fecha que arrastra el pago a AP | Fecha en período contable cerrado |
| Pay Cycle (AP) | Pay Cycle ID y fecha | Identifica la corrida de pago | Fecha anterior al staging |
|  | Fuente de pagos | Incluye Expenses entre los orígenes | Sin la fuente, el pago no se selecciona |
|  | Método de pago | Debe ser EFT | Método distinto, con el pago sin generar |
|  | Banco y cuenta origen | Cuenta desde la que se paga | Cuenta sin layout EFT |
| Journal Generator (FS\_JGEN) | Business Unit y fecha contable | Define dónde y cuándo se contabiliza | Fecha en período cerrado |
|  | Plantilla y origen del diario | Mapea las líneas contables a diarios | Plantilla sin ChartFields completos |
|  | Edit y Post | Valida y postea los diarios | Diario con error de edición sin corregir |
| Budget Processor (FS\_BP) | Transacción fuente y rango | Verifica presupuesto por lotes de TAs | Rango que excluye las TAs pendientes |

## 10. Estados de Autorización, Anticipo e Informe de Gastos

Cada transacción avanza por estados que indican qué sigue y quién debe actuar. Los valores corresponden a los típicos de Expenses; confírmelos en las tablas de valores de su instancia.

### Estado de aprobación (común a las tres transacciones)

| Estado | Significado | Quién actúa | Siguiente paso |
| --- | --- | --- | --- |
| Pendiente (Pending) | En edición, aún no enviada | Empleado | Enviar |
| Enviado (Submitted) | En ruta de aprobación | Aprobador | Aprobar, denegar o devolver |
| Aprobado (Approved) | Aprobada por todos los pasos | Administrador | Stage Payments, si genera pago |
| Denegado (Denied) | Rechazada por el aprobador | Empleado | Corregir o cerrar |
| Devuelto para corrección | Regresada para ajustes | Empleado | Editar y reenviar |
| Cancelado (Canceled) | Anulada antes del pago | Empleado | Ninguno |

### Estados adicionales por transacción

| Transacción | Estado adicional | Qué indica |
| --- | --- | --- |
| Autorización de Viaje | Estado presupuestal | Válida, con excepción o sin verificar |
| Autorización de Viaje | Liquidada o cerrada | Se asoció el informe final y se liberó el compromiso |
| Anticipo de Caja | Estado de pago | Sin pagar, en staging o pagado |
| Anticipo de Caja | Estado de aplicación | Abierto, aplicado parcial o conciliado |
| Informe de Gastos | Estado de pago | Sin pagar, en staging o pagado |
| Informe de Gastos | Estado contable | Sin contabilizar, con pasivo contabilizado o con pago contabilizado |

### Cómo se mueven entre sí

1. Pendiente pasa a Enviado cuando el empleado envía.
2. Enviado pasa a Aprobado, Denegado o Devuelto según el aprobador.
3. Aprobado pasa a **En staging** con Stage Payments y a **Pagado** con Post Payments.
4. El estado contable avanza con Post Liabilities y con el Journal Generator.

**Regla útil para la sesión:** si una transacción no avanza, mire primero el estado de aprobación, luego el de pago y al final el contable, en ese orden.

## 11. Casos de uso y seguimiento

La capacitación se evalúa con 38 casos de uso, UC01 a UC38, agrupados por módulo. El detalle de cada caso (rol, precondición, pasos, resultado esperado y evidencia) y el control de avance están en la matriz de seguimiento en Excel que acompaña esta guía.

| Módulo | Casos | Cantidad |
| --- | --- | --- |
| 1. Configuración e integración HCM | UC01 a UC06 | 6 |
| 2. Autorización de Viaje y Control Presupuestal | UC07 a UC13 | 7 |
| 3. Anticipo de Caja | UC14 a UC17 | 4 |
| 4. Informe de Gastos | UC18 a UC24 | 7 |
| 5. Flujo de aprobación (AWE) | UC25 a UC29 | 5 |
| 6. Proceso de gastos, AP (EFT) y GL | UC30 a UC38 | 9 |

**Cómo se hace el seguimiento.** Cada caso pasa por los estados Pendiente, En ejecución, Ejecutado OK, Con defecto y Bloqueado. Un caso se cierra cuando tiene evidencia (captura o número de transacción) y el resultado coincide con el esperado. El tablero de la hoja de resumen calcula el avance por módulo.

## 12. Errores frecuentes y glosario

| Síntoma | Causa probable | Qué revisar |
| --- | --- | --- |
| No se puede crear la TA o el informe | Perfil del empleado incompleto | Perfil: BU, supervisor, ChartFields por defecto |
| La TA no pasa a aprobación | Excepción presupuestal abierta | Budget Exceptions y presupuesto disponible |
| La transacción queda detenida tras enviar | Sin aprobador asignado | Approver Assignment y listas de usuarios del AWE |
| Stage Payments no toma el informe | Informe sin aprobar o BU distinta | Estado de aprobación y BU de la petición |
| El Pay Cycle no selecciona el pago | Fuente Expenses o método EFT mal configurados | Criterios de selección del Pay Cycle |
| El pago no figura como pagado en Expenses | Post Payments sin ejecutar | Estado de pago y proceso Post Payments |
| El diario no se genera | Plantilla o ChartFields incompletos | Journal Generator y fecha contable |

| Término | Definición |
| --- | --- |
| TA | Autorización de Viaje (Travel Authorization) |
| AWE | Motor de flujo de aprobación (Approval Workflow Engine) |
| Staging | Tablas intermedias donde Expenses deja los pagos para AP |
| EFT | Transferencia electrónica de fondos |
| Pay Cycle | Corrida de pagos de AP que selecciona, crea y envía pagos |
| Encumbrance | Compromiso que reserva presupuesto |
| ChartFields | Segmentos contables: cuenta, centro de costo, proyecto y otros |
| Journal Generator | Proceso que crea diarios de GL desde líneas contables de subsistemas |
