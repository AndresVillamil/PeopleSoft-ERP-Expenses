# Historial de Consulta Técnico - PeopleSoft Expenses & AWE

Este documento contiene la memoria técnica de las sesiones de soporte sobre **Control de Compromisos (Commitment Control)** y el **Motor de Aprobaciones (AWE)** en PeopleSoft Expenses.

---

## 1. Conceptos Base: Control de Compromisos y Excepciones

### ¿Qué es el Control de Compromisos?
Es el mecanismo de verificación presupuestaria automatizada (*Budget Checking*) dentro de PeopleSoft. Su comportamiento varía según el documento de gastos:
* **TA (Travel Authorization / Autorización de Viaje):** Actúa como una reserva de fondos o precompromiso (**Encumbrance**). El sistema aparta el presupuesto estimado.
* **Hoja de Gastos (Expense Report):** Registra el gasto contable real (**Expense**). Si existía una TA ligada, libera el precompromiso y ejecuta el cargo real definitivo.

### Excepciones Presupuestarias
Ocurren cuando una transacción falla las reglas de validación contable y de fondos financieros:
* **Errores (`Error` / Bloqueante):** Impiden que el documento continúe el flujo de aprobación o pago hasta su resolución (ej. *Exceeds Control Budget*, *No Budget Exists*).
* **Advertencias (`Warning` / Informativo):** Anomalías menores (como variaciones leves de tolerancia) que permiten el avance del trámite.

---

## 2. Diagnóstico: Error "Presupuesto Inexistente" (No Budget Exists)

### Contexto de la Casuística
* **ID Informe:** `0000002343`
* **Business Unit:** `ACHCO`
* **Grupo Contable:** `CC_GTO_CTR`
* **Mensaje en Consulta de Detalles:** *"No hay valores coincidentes"*

### Origen Técnico
Falta de configuración física o apertura del periodo en las tablas de control del presupuesto. En este caso particular, el conflicto se originó en el campo **Periodo Presupuesto (`2026M09`)**, correspondiente a Septiembre de 2026, el cual no contaba con techos presupuestarios cargados o activos para la combinación de *ChartFields*:
* **Cuenta:** `5190350005`
* **Departamento:** `1000400100`
* **Referencia Presupuesto:** `ON0177`

### Rutas de Solución

#### Para el Administrador de Presupuestos (Finanzas)
1. **Opción A (Solución Definitiva):** Generar un Asiento de Presupuesto Original o Ajuste en la ruta `Commitment Control > Budget Journals > Enter Budget Journals` para la combinación y periodo indicados, realizando el posterior *Budget Check* y *Post* del diario.
2. **Opción B (Solución Rápida):** Realizar una sustitución manual o forzado de la transacción (**Override**) desde la pantalla de excepciones de Hoja de Gastos marcando la opción *Sustituir Presupuesto / Transacción*.

#### Para el Usuario Final (Que reporta)
* Corregir el informe modificando las claves contables en caso de error de captura o verificar la fecha de los recibos para cerciorarse de que pertenezcan al periodo fiscal abierto actual.

---

## 3. Fallo Lógico en el Flujo de Aprobación (AWE) y Auto-Aprobación

### Síntoma
Al intentar realizar una reasignación administrativa sobre un informe con presupuesto inválido (ej. ID `0000001957`), el sistema saltaba el flujo humano y marcaba el informe automáticamente como **Aprobado** a nivel superior, dejándolo congelado internamente.

### Causa Raíz
Al revisar la **Definición de Criterios (Proceso `ERApproval`, Definición `ACHCO`)**, se identificó un fallo de lógica condicional en la configuración de la ruta y los pasos:
* **Condición Errónea:** `EX_ER_HDR_AW_VW.TOTAL_REIMB_AMT` **Menor Que** `0`

Como la gran mayoría de las hojas de gastos poseen montos positivos (mayores a cero), el motor de PeopleSoft evaluaba la condición como **Falsa** y determinaba que la ruta de aprobación humana no aplicaba para el documento. Al estar encendida la casilla **Aprobación Auto Usuario (Auto Approve)** en las opciones del proceso, el sistema ejecutaba su lógica de contingencia y aprobaba el informe de forma inmediata.

### Plan de Corrección en AWE
1. Activar la edición mediante el botón **Corregir Historial**.
2. Modificar el criterio del campo del monto total a: **Mayor Que `0`**.
3. **Seguridad adicional:** Desmarcar la casilla de *Aprobación Auto Usuario* para que los errores queden en la bandeja del Administrador del Sistema (*System Administrator*) en lugar de autoaprobarse.
4. Agregar un segundo criterio que condicione el avance a un estado presupuestario válido: `BUDGET_HDR_STATUS` **Igual A** `'V'`.

---

## 4. Estructura de Tablas del Motor de Aprobaciones y Consultas SQL

El XML de una instancia en ejecución del AWE expone la jerarquía lógica de los pasos, pero difiere ligeramente de las tablas físicas relacionales en la base de datos de PeopleSoft.

### Mapeo XML vs Tabla Física
* Etiqueta lógica `<userstep>` $ightarrow$ Tabla física **`PS_EOAW_USERINST`**
* Campo `EOAWTHREAD_ID` $ightarrow$ Tabla física **`PS_EOAW_STEPINST`** (Relacionadas por `EOAWSTEP_INSTANCE`)

### Consulta SQL: Identificar Usuarios Aprobadores en Espera
Esta consulta realiza el puente correcto entre las tablas de instancias de pasos y de usuarios finales para extraer los IDs y nombres de los aprobadores asignados a un hilo del AWE (ejemplo con los rangos provistos del XML):

```sql
SELECT 
    S.EOAWTHREAD_ID      AS "ID Hilo",
    U.EOAWUSTEP_INST_ID  AS "ID Instancia Usuario",
    U.OPRID              AS "ID Aprobador",
    B.OPRDEFNDESC        AS "Nombre del Aprobador",
    U.EOAWSTEP_STATUS    AS "Estado del Paso"
FROM 
    PS_EOAW_STEPINST S
INNER JOIN 
    PS_EOAW_USERINST U ON S.EOAWSTEP_INSTANCE = U.EOAWSTEP_INSTANCE
INNER JOIN 
    PSOPRDEFN B ON U.OPRID = B.OPRID
WHERE 
    S.EOAWTHREAD_ID = 28304
    AND U.EOAWUSTEP_INST_ID BETWEEN 890207 AND 890216;
```

### Comportamiento de Multi-Registros por Hilo (`PS_EOAW_STEPINST`)
Al hacer una consulta general en `PS_EOAW_STEPINST` para un hilo específico (ej. `28294`), es normal obtener múltiples filas ya que el sistema guarda el registro histórico de transición de estados:
1. **Fila con `EOAWSTEP_TYPE = N` (Normal) y `STATUS = K` (Skipped):** Indica que el paso de aprobación humana regular fue omitido de forma directa (debido al error de criterio *"Menor que 0"*).
2. **Fila con `EOAWSTEP_TYPE = E` (Error Step), `STATUS = A` (Approved) y `EOAWADHOC_BY = System`:** Indica el sub-paso decimal (`.01`) de contingencia creado de forma automática por el servidor de aplicaciones (*System*) para auto-aprobar la transacción ante la falta de una ruta válida aplicable.

---
*Nota: Este documento sirve como especificación técnica para revisiones de soporte interno con el equipo de infraestructura o mesa de control de PeopleSoft Financials.*
