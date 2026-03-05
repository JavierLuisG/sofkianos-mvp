# TC US-013

Como experto en pruebas de software, he analizado el contexto de negocio de SofkianOS y la historia de usuario US-013. A continuación, presento el diseño de los casos de prueba estructurados en lenguaje Gherkin, organizados por bloques funcionales de la página de lista de Kudos.

**Técnicas ISTQB Aplicadas:**
- **Partición de Equivalencia (PE)**: Para validar escenarios con datos existentes, lista vacía y combinaciones de filtros.
- **Análisis de Valores Límite (AVL)**: Para validar paginación y límites de visualización.
- **Pruebas de Transición de Estados (TE)**: Para validar cambios de filtros, navegación y sincronización con URL.
- **Pruebas de Seguridad**: Para validar enmascaramiento e integridad de identificadores.
- **Pruebas de Experiencia de Usuario (UX)**: Para validar estados de carga y responsividad.

---

## Casos de Prueba en Lenguaje Gherkin

### 1. Visualización y Estados de la Interfaz

**Caso de Prueba 01: Visualización exitosa de la lista de Kudos**

**Técnica: PE**

#### Escenario (Gherkin)

```gherkin
Given el empleado de Sofka accede a la sección de historial de Kudos
When el sistema carga los datos desde la API exitosamente
Then el sistema debe mostrar la tabla `KudoTable` con las columnas de remitente, destinatario, categoría y mensaje
And los correos electrónicos deben mostrarse enmascarados (ej: j***z@domain.com)
And cada kudo debe mostrar el icono representativo de su categoría (Innovation, Teamwork, Passion, Mastery)
```

**Caso de Prueba 02: Visualización del estado de carga (Skeleton)**

**Técnica: UX / TE**

#### Escenario (Gherkin)

```gherkin
Given el empleado de Sofka solicita ver la lista de Kudos
When la respuesta de la API está en proceso (latencia de red)
Then el sistema debe mostrar componentes de carga (Skeletons) en lugar de la tabla
And el usuario no debe ver una pantalla en blanco
```

**Caso de Prueba 03: Manejo de lista vacía**

**Técnica: PE**

#### Escenario (Gherkin)

```gherkin
Given el empleado de Sofka accede a la lista de Kudos
When la API responde con un arreglo vacío (sin registros)
Then el sistema debe mostrar un mensaje informativo indicando que no se encontraron reconocimientos
And no debe mostrarse la estructura de la tabla ni la paginación
```

**Caso de Prueba 04: Manejo de error en la petición**

**Técnica: TE**

#### Escenario (Gherkin)

```gherkin
Given el empleado de Sofka intenta visualizar los Kudos
When la API responde con un error (ej: HTTP 500 o 503)
Then el sistema debe mostrar un mensaje de error claro al usuario
And debe permitir al usuario intentar recargar la información
```

### 2. Filtrado y Búsqueda

**Caso de Prueba 05: Filtrado por categoría predefinida**

**Técnica: PE**

#### Escenario (Gherkin)

```gherkin
Given el empleado se encuentra en la vista de Kudos
When selecciona la categoría "Innovation" en el filtro
Then la tabla debe actualizarse para mostrar únicamente los Kudos de esa categoría
And la URL del navegador debe incluir el parámetro `category=Innovation`
```

**Caso de Prueba 06: Búsqueda por texto (Search) con resultados**

**Técnica: PE**

#### Escenario (Gherkin)

```gherkin
Given existen Kudos con el mensaje "Excelente trabajo en el sprint"
When el empleado escribe "sprint" en el campo de búsqueda
Then el sistema debe filtrar la tabla para mostrar solo los registros que coincidan con ese texto
And la búsqueda debe ser insensible a mayúsculas y minúsculas
```

**Caso de Prueba 07: Combinación de filtros y búsqueda**

**Técnica: PE**

#### Escenario (Gherkin)

```gherkin
Given el empleado aplica el filtro de categoría "Mastery"
When escribe un término en el campo de búsqueda que no existe dentro de esa categoría
Then el sistema debe mostrar el estado de "Sin resultados"
And debe mantener los filtros visibles para que el usuario pueda ajustarlos
```

### 3. Paginación y Navegación

**Caso de Prueba 08: Navegación entre páginas**

**Técnica: TE**

#### Escenario (Gherkin)

```gherkin
Given existen más de 10 Kudos registrados (límite por página)
When el empleado hace clic en el botón de la página "2"
Then el sistema debe solicitar a la API el siguiente set de datos
And la URL debe actualizarse con el parámetro `page=2`
And la tabla debe hacer scroll automático hacia la parte superior
```

**Caso de Prueba 09: Persistencia de filtros mediante URL (Deep Linking)**

**Técnica: TE**

#### Escenario (Gherkin)

```gherkin
Given un empleado recibe un enlace con los parámetros `?category=Passion&page=3`
When el empleado abre el enlace en su navegador
Then el sistema debe cargar automáticamente la página 3 de los Kudos filtrados por la categoría "Passion"
And los selectores de la interfaz deben reflejar estos filtros activos
```

### 4. Reglas de Negocio y Seguridad

**Caso de Prueba 10: Validación de privacidad (Enmascaramiento)**

**Técnica: Seguridad / PE**

#### Escenario (Gherkin)

```gherkin
Given el sistema muestra un Kudo enviado por "juan.perez@sofka.com" a "maria.lopez@sofka.com"
When se renderiza la fila en la `KudoTable`
Then el sistema no debe mostrar los correos completos bajo ninguna circunstancia
And debe mostrar el formato de enmascaramiento definido (ej: j***z@sofka.com y m***z@sofka.com)
```

**Caso de Prueba 11: Integridad de IDs no secuenciales**

**Técnica: Seguridad**

#### Escenario (Gherkin)

```gherkin
Given el empleado inspecciona el código de la página o los enlaces de la tabla
When observa los identificadores de cada Kudo
Then los IDs deben ser hashes no secuenciales para evitar la enumeración de registros
And no debe ser posible predecir el ID del siguiente Kudo cambiando un número secuencial
```

### 5. Responsividad (UX Técnica)

**Caso de Prueba 12: Adaptabilidad a dispositivos móviles**

**Técnica: PE / UX**

#### Escenario (Gherkin)

```gherkin
Given el empleado accede al sistema desde un dispositivo con pantalla menor a 640px (Mobile)
When se carga la lista de Kudos
Then la tabla debe adaptarse para evitar el scroll horizontal desbordado
And el contenido debe seguir siendo legible y los filtros deben ser accesibles
```