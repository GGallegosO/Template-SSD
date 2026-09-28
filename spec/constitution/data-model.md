# data-model.md — Modelo de Datos y Diagrama de Base de Datos

> **Propósito de este archivo:** Registrar la decisión de esquema de datos del proyecto: entidades,
> relaciones, claves y diagrama. Esta es la **fuente única de verdad (single source of truth)** del
> modelo de datos. Cualquier feature que lea o escriba datos debe respetar lo aquí definido; si una
> feature necesita cambiar el esquema, primero se actualiza este archivo (y se registra en `doc/`).
>
> **¿Dónde ubicar esta información?** Aquí, en `spec/constitution/data-model.md`, porque el modelo
> de datos es una **decisión estructural transversal** (afecta a todo el sistema), no el detalle de
> una feature individual. Las decisiones *de selección tecnológica* (qué motor de BD y por qué)
> viven en `tech-stack.md`; el *esquema concreto* vive aquí. El código de migraciones/DAL es solo
> la implementación reflejada de este documento.
>
> **Regla de sincronización:** Si las migraciones reales y este documento divergen, hay un bug de
> documentación. Se corrige este archivo en el mismo PR que modifica el esquema.

---

## 1. Resumen de la Decisión

| Campo                        | Valor                                        |
|------------------------------|----------------------------------------------|
| Motor de persistencia        | [MOTOR_DE_BD] (referencia a su fila en `tech-stack.md`) |
| Modelo de datos              | [RELACIONAL / DOCUMENTAL / CLAVE-VALOR / GRAFO / MIXTO] |
| Fecha de aprobación          | [AAAA-MM-DD]                                 |
| Aprobado por                 | [RESPONSABLE_O_EQUIPO]                       |
| Tarea que originó la decisión| [NNN-nombre-tarea] (archivo en `/doc/`)      |
| Estado del esquema           | [BORRADOR / CONGELADO PARA v1 / EN EVOLUCIÓN] |

> *Guía:* Una sola línea basta para justificar "por qué este esquema". Ejemplo de estilo
> (rellenar con datos ficticios): "Se prioriza integridad referencial sobre flexibilidad de
> esquema porque el dominio tiene relaciones estables verificadas con [ROL_DE_USUARIO]."

---

## 2. Diagrama Entidad-Relación (código como fuente de verdad)

> *Guía:* Dibuja el diagrama en **Mermaid** dentro de este bloque. No pegues imágenes binarias
> como única fuente: el texto versiona, se revisa en PR y los agentes pueden leerlo. Si además
> quieres una imagen renderizada, guárdala en `spec/constitution/diagrams/` y enlázala abajo,
> pero el Mermaid manda.

```mermaid
erDiagram
    [ENTIDAD_A] ||--o{ [ENTIDAD_B] : "VERBO_DE_RELACION"
    [ENTIDAD_A] {
        [tipo] [PK] id
        [tipo] [nombre_campo]
    }
    [ENTIDAD_B] {
        [tipo] [PK] id
        [tipo] [FK] entidad_a_id
        [tipo] [nombre_campo]
    }
```

**Imagen renderizada (opcional, complementaria):** ![Diagrama](diagrams/[NOMBRE_DEL_DIAGRAMA].png)

> *Ejemplo ficticio de cómo se ve relleno (no copiar al proyecto real):*
> ```mermaid
> erDiagram
>     USUARIO ||--o{ PEDIDO : "realiza"
>     PEDIDO ||--|{ LINEA_PEDIDO : "contiene"
>     PRODUCTO ||--o{ LINEA_PEDIDO : "aparece_en"
> ```

---

## 3. Diccionario de Entidades

> *Guía:* Una subsección por entidad. Incluye siempre: propósito (una frase), campos con tipo
> lógico (no sintaxis de un motor concreto → se mantiene agnóstico), claves e índices, y reglas
> de negocio asociadas.

### 3.1 [ENTIDAD_1]

- **Propósito:** [QUE_REPRESENTA_ESTA_ENTIDAD_EN_EL_DOMINIO]
- **Ciclo de vida:** [CREADA_POR_... / TRANSICIONES_VALIDAS / SE_ARCHIVA_CUANDO...]

| Campo          | Tipo lógico            | Clave | Nullable | Regla de negocio / validación      |
|----------------|------------------------|-------|----------|------------------------------------|
| [id]           | [IDENTIFICADOR_UNICO]  | PK    | No       | Generado por [ESTRATEGIA_DE_IDS]   |
| [campo_1]      | [TIPO_1]               | —     | [Sí/No]  | [REGLA_O_VALIDACION]               |
| [campo_2]      | [TIPO_2]               | FK→[OTRA_ENTIDAD] | [Sí/No] | [REGLA_O_VALIDACION]     |

- **Índices:** [LISTA_DE_INDICES_CON_JUSTIFICACION_DE_ACCESO]
- **Eliminación:** [SUAVE_CON_CAMPO_deleted_at / FISICA / NO_SE_ELIMINA] → Justificación: [POR_QUE]

*(Repetir bloque 3.x por cada entidad del dominio.)*

---

## 4. Relaciones y Cardinalidades

| Origen         | Destino        | Cardinalidad | Semántica de la relación        | Integridad referencial |
|----------------|----------------|--------------|---------------------------------|------------------------|
| [ENTIDAD_A]    | [ENTIDAD_B]    | [1:N / N:M]  | [QUE_SIGNIFICA_LA_RELACION]     | [CASCADE / RESTRICT / MANUAL] |

> *Guía:* Para relaciones N:M declara explícitamente la tabla/colección intermedia y quién la
> escribe. Anota toda regla que el motor NO puede imponer (se valida en lógica de negocio).

---

## 5. Decisiones de Diseño del Esquema (ADR-lite)

> *Guía:* Lista numerada. Cada entrada: decisión → alternativas descartadas → justificación →
> consecuencia. Esto es lo que permite a un agente futuro entender *por qué* no cambiar algo.

1. **[DECISION_1_EJ_NOMBRES_EN_SNAKE_CASE]**
   - Alternativas descartadas: [ALTERNATIVA_A], [ALTERNATIVA_B]
   - Justificación: [MOTIVO]
   - Consecuencia: [IMPACTO_A_LARGO_PLAZO]
2. **[DECISION_2_EJ_KEYS_NATURALES_VS_SURROGADAS]**
   - ...

---

## 6. Estrategia de Evolución del Esquema

- **Migraciones:** [HERRAMIENTA_DE_MIGRACION] (coherente con `tech-stack.md` §3).
- **Reglas obligatorias:**
  - [Toda migracion debe ser reversible o documentar por que no]
  - [Cambios destructivos requieren doble fase: agregar → migrar datos → retirar]
  - [El numero de migracion corresponde a la tarea NNN de /doc/ que la origino]
- **Datos semilla (seed):** [ESTRATEGIA_DE_DATOS_INICIALES_PARA_DEV_Y_TEST]

---

## 7. Trazabilidad hacia Features

> *Guía:* Al crear una feature en `spec/features/NNN-nombre/`, referencia en su `plan.md` las
> entidades que toca. Actualiza esta tabla para saber qué partes del esquema están "activas".

| Entidad/Tabla afectada | Feature (carpeta en spec/features/) | Dirección de acceso | Notas |
|------------------------|-------------------------------------|---------------------|-------|
| [ENTIDAD_1]            | [NNN-NOMBRE_FEATURE]                | [lectura / escritura / ambas] | [NOTA] |
