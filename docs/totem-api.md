# Documentación API — Verificación de Afiliación SOEA

## Propósito

Este servicio permite verificar si un número de documento corresponde a un afiliado principal o a un familiar de un afiliado de **SOEA** (Sindicato de Obreros y Empleados Aceiteros).

Está diseñado para ser consumido desde un **tótem de autoservicio** ubicado en una sucursal del centro médico, en el proceso de solicitud de turnos.

---

## Endpoint

- **URL completa:**
  ```
  https://agserviciosdesalud.ar/soea/api/affiliate/checkSOEAAffiliation/:docNumber
  ```
- **Método:** `GET`
- **Parámetros:**
  - `:docNumber`: Número de documento nacional (DNI) del usuario a verificar.
    - **Tipo:** `string` (texto)
    - **Formato:** Solo números, sin puntos, guiones ni espacios.
    - **Longitud mínima:** 7 dígitos (ej: `1234567`)
    - **Longitud máxima:** 8 dígitos (ej: `12345678`)
    - **Notas:** No es necesario anteponer ceros a la izquierda.
- **Headers:**
  - `Authorization`: API Key provista por el equipo de desarrollo.
    > La API Key debe incluirse en el header `Authorization` como texto plano (no como `Bearer`).

---

## Autenticación

El servicio está protegido mediante un sistema de validación con **API Key**. Las claves son administradas internamente por el equipo técnico y deben solicitarse para su uso.

Se realiza una comparación segura y se controla además que la clave esté activa. El acceso sin una clave válida será rechazado con los siguientes códigos:

- `401 Unauthorized`: Falta el header `Authorization`.
- `403 Forbidden`: Clave inválida o inactiva.

---

## Respuesta

La respuesta siempre es un objeto JSON con los siguientes campos:

```json
{
  "isAffiliate": true | false,
  "isPrimaryAffiliate": true | false,
  "message": "Descripción textual del resultado"
}
```

---

## Ejemplos de respuesta

### ✅ Afiliado principal

```json
{
  "isAffiliate": true,
  "isPrimaryAffiliate": true,
  "message": "Es afiliado principal"
}
```

### ✅ Familiar de afiliado

```json
{
  "isAffiliate": true,
  "isPrimaryAffiliate": false,
  "message": "Es pariente de afiliado"
}
```

### ✅ Jubilado principal

```json
{
  "isAffiliate": true,
  "isPrimaryAffiliate": true,
  "message": "Es jubilado principal"
}
```

### ✅ Familiar de jubilado

```json
{
  "isAffiliate": true,
  "isPrimaryAffiliate": false,
  "message": "Es pariente de jubilado"
}
```

> **Nota técnica:** Los registros de jubilados no almacenan el número de documento directamente, sino el CUIL. Dado que el DNI está contenido dentro del CUIL (formato: `XX-XXXXXXXX-X`), la verificación se realiza buscando el número de documento como subcadena del CUIL.

### ❌ Documento no encontrado

```json
{
  "isAffiliate": false,
  "isPrimaryAffiliate": false,
  "message": "No hay ninguna coincidencia"
}
```

---

## Tiempo de respuesta promedio

Promedio de **600 ms**, dependiendo de la carga del servidor y la respuesta de la base de datos SQL Server alojada en los servidores de SOEA.

> **Recomendación:** Configurar un timeout del lado del cliente de al menos **5 segundos**, para tener margen ante posibles demoras esporádicas.

---

## Manejo de errores

Las respuestas de error siguen el siguiente formato JSON:

```json
{
  "message": "No hay ninguna coincidencia"
}
```

### Códigos de error posibles

| Código | Estado                | Descripción                                                       |
| ------ | --------------------- | ----------------------------------------------------------------- |
| `400`  | Bad Request           | El número de documento enviado no es válido.                      |
| `401`  | Unauthorized          | La API Key no fue provista o es incorrecta.                       |
| `403`  | Forbidden             | La API Key no tiene permisos para acceder a este recurso.         |
| `409`  | Conflict              | Se detectaron múltiples coincidencias con el número de documento. |
| `500`  | Internal Server Error | Ocurrió un error inesperado al procesar la solicitud.             |

---

## Error 409 — Conflict

El código `409 Conflict` se devuelve cuando existen **múltiples coincidencias** del número de documento en la tabla de afiliados principales, considerando únicamente aquellos registros con estado **activo**.

Este conflicto surge debido a que el campo de número de documento no posee una restricción de unicidad en la base de datos. Por lo tanto, ante más de una coincidencia, no es posible determinar de forma segura una única afiliación principal.

Actualmente, con los datos vigentes y la validación incorporando el estado del afiliado (activo o inactivo), no se detectan casos de conflicto. Sin embargo, la lógica se mantiene activa como medida de seguridad ante posibles inconsistencias futuras.

> **Nota:** El servicio no considera múltiples coincidencias en el grupo familiar como un conflicto, ya que es válido que un mismo número de documento aparezca más de una vez, por ejemplo, un hijo vinculado a más de un afiliado principal.

---

## Límite de uso

Actualmente no hay limitaciones de _rate limit_ (llamadas por unidad de tiempo) impuestas por la API.

Sin embargo, se recomienda no hacer llamadas innecesarias ni en bucles, ya que el acceso está pensado exclusivamente para el tótem y uso controlado.

> Cualquier expansión en el uso debe coordinarse con el equipo técnico.

---

## Logs y auditoría

Actualmente, el servicio **no registra logs** ni realiza auditoría de las consultas realizadas. No se almacena información relacionada con los accesos ni los resultados devueltos por la API.

---

## Notas adicionales

El endpoint está pensado para uso exclusivo del tótem de autoservicio, pero podría ampliarse en el futuro a otras plataformas internas.

La documentación y las credenciales de acceso serán provistas únicamente a servicios autorizados por el equipo técnico, deben mantenerse **confidenciales** y no deben compartirse fuera del entorno autorizado.

---

## Historial de versiones

| Versión | Descripción                                                              |
| ------- | ------------------------------------------------------------------------ |
| `1.0.0` | Versión inicial. Verificación de afiliados principales y sus familiares. |
| `1.0.1` | Se agregó verificación de jubilados principales y sus familiares.        |

---

## Contacto técnico

Esta API ha sido desarrollada y es mantenida por el equipo de desarrollo de **AGSalud**.

Para consultas técnicas, reportes de errores o solicitudes de integración, contactarse por correo a:

- 📧 [ninotell@gmail.com](mailto:ninotell@gmail.com)
- 📧 [franco.mac@gmail.com](mailto:franco.mac@gmail.com)
