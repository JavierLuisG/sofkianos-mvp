# TC US-013

Como experto en pruebas de software, he analizado el contexto de negocio de SofkianOS y la historia de usuario US-013. A continuación, presento el diseño de los casos de prueba estructurados en lenguaje Gherkin, organizados por bloques funcionales de la página de lista de Kudos.

**Técnicas ISTQB Aplicadas:**
- **Partición de Equivalencia (PE)**: Para validar escenarios con datos existentes, lista vacía y combinaciones de filtros.
- **Análisis de Valores Límite (AVL)**: Para validar paginación y límites de visualización.
- **Pruebas de Transición de Estados (TE)**: Para validar cambios de filtros, navegación y sincronización con URL.
- **Pruebas de Seguridad**: Para validar enmascaramiento e integridad de identificadores.
- **Pruebas de Experiencia de Usuario (UX)**: Para validar estados de carga y responsividad.

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

---

**Caso de Prueba 02: Visualización del estado de carga (Skeleton)**

**Técnica: UX / TE**

#### Escenario (Gherkin)

```gherkin
Given el empleado de Sofka solicita ver la lista de Kudos
When la respuesta de la API está en proceso (latencia de red)
Then el sistema debe mostrar componentes de carga (Skeletons) en lugar de la tabla
And el usuario no debe ver una pantalla en blanco
```

---

**Caso de Prueba 03: Manejo de lista vacía**

**Técnica: PE**

#### Escenario (Gherkin)

```gherkin
Given el empleado de Sofka accede a la lista de Kudos
When la API responde con un arreglo vacío (sin registros)
Then el sistema debe mostrar un mensaje informativo indicando que no se encontraron reconocimientos
And no debe mostrarse la estructura de la tabla ni la paginación
```

---

**Caso de Prueba 04: Manejo de error en la petición**

**Técnica: TE**

#### Escenario (Gherkin)

```gherkin
Given el empleado de Sofka intenta visualizar los Kudos
When la API responde con un error (ej: HTTP 500 o 503)
Then el sistema debe mostrar un mensaje de error claro al usuario
And debe permitir al usuario intentar recargar la información
```

---

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

---

**Caso de Prueba 06: Búsqueda por texto (Search) con resultados**

**Técnica: PE**

#### Escenario (Gherkin)

```gherkin
Given existen Kudos con el mensaje "Excelente trabajo en el sprint"
When el empleado escribe "sprint" en el campo de búsqueda
Then el sistema debe filtrar la tabla para mostrar solo los registros que coincidan con ese texto
And la búsqueda debe ser insensible a mayúsculas y minúsculas
```

---

**Caso de Prueba 07: Combinación de filtros y búsqueda**

**Técnica: PE**

#### Escenario (Gherkin)

```gherkin
Given el empleado aplica el filtro de categoría "Mastery"
When escribe un término en el campo de búsqueda que no existe dentro de esa categoría
Then el sistema debe mostrar el estado de "Sin resultados"
And debe mantener los filtros visibles para que el usuario pueda ajustarlos
```

---

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

---

**Caso de Prueba 09: Persistencia de filtros mediante URL (Deep Linking)**

**Técnica: TE**

#### Escenario (Gherkin)

```gherkin
Given un empleado recibe un enlace con los parámetros `?category=Passion&page=3`
When el empleado abre el enlace en su navegador
Then el sistema debe cargar automáticamente la página 3 de los Kudos filtrados por la categoría "Passion"
And los selectores de la interfaz deben reflejar estos filtros activos
```

---

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

---

**Caso de Prueba 11: Integridad de IDs no secuenciales**

**Técnica: Seguridad**

#### Escenario (Gherkin)

```gherkin
Given el empleado inspecciona el código de la página o los enlaces de la tabla
When observa los identificadores de cada Kudo
Then los IDs deben ser hashes no secuenciales para evitar la enumeración de registros
And no debe ser posible predecir el ID del siguiente Kudo cambiando un número secuencial
```

---

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

---

## Tabla de Casos de Prueba - HU-013

| ID | Caso de Prueba generado por la gema | Ajuste realizado por el probador                                                  | ¿Por qué se ajustó?                                                                                    |
|--------|--------------------------------|-----------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------|
| TC-01 | Visualización exitosa de lista   | Agregar validación de ordenamiento por fecha descendente por defecto              | No se valida una regla común para historiales como el registro más reciente debe mostrarse primero. Mejora cobertura funcional. |
| TC-02 | Estado de carga (Skeleton)       | Agregar validación de tiempo máximo de espera antes de mostrar mensaje de error   | No se contempló timeout controlado y por ello se requiere validar comportamiento ante latencias prolongadas.                    |
| TC-03 | Lista vacía                      | Validar código HTTP 200 junto con arreglo vacío                                   | Se debe distinguir entre lista vacía válida y error de servicio. Mejora precisión del caso.                                     |
| TC-04 | Manejo de error API              | Agregar validación de no exposición de detalles técnicos del error (stacktrace)   | Riesgo de seguridad. No deben mostrarse detalles internos al usuario final.                                                     |
| TC-05 | Filtro por categoría             | Validar que el filtro sea persistente al recargar la página                       | Se debe asegurar consistencia de estado mediante parámetros de URL.                                                             |
| TC-06 | Búsqueda por texto               | Reconocer texto con asentos y caracteres especiales                               | Validación de normalización de texto y manejo de encoding.                                                                      |
| TC-07 | Combinación filtros              | Agregar validación de reinicio correcto al limpiar filtros                        | No se contempló el flujo de restauración del estado original.                                                                   |
| TC-08 | Paginación                       | Validar límite inferior y superior (page=0, page negativa, page mayor al total)   | Aplicación de análisis de valores límite según ISTQB.                                                                           |
| TC-09 | Deep Linking                     | Validar comportamiento con parámetros inválidos (category inexistente)            | Cobertura de excepciones y robustez ante manipulación manual de URL.                                                            |
| TC-10 | Enmascaramiento                  | Validar que el correo completo no esté expuesto en el DOM ni en atributos ocultos | Seguridad front-end. Evita fuga de datos sensibles vía inspección.                                                              |
| TC-11 | IDs no secuenciales              | Agregar validación de intento de acceso directo a un ID alterado                  | Cobertura de prueba negativa orientada a seguridad (enumeración).                                                               |
| TC-12 | Responsividad móvil              | Agregar validación en breakpoint tablet (ej: 768px) además de móvil <640px        | Se amplía cobertura responsive a más de un punto de quiebre.                                                                    |

---

# TC HU-016

Como experto en pruebas de software, he analizado el contexto de negocio de SofkianOS y la historia de usuario US-016. A continuación, presento el diseño de los casos de prueba utilizando técnicas de Análisis de Valores Límite, Partición de Equivalencia y Transición de Estados, estructurados en lenguaje Gherkin.

**Técnicas ISTQB Aplicadas:**
- **Análisis de Valores Límite (AVL)**: Para validar el comportamiento de los botones "Anterior" y "Siguiente" en los extremos, y el cálculo del rango mostrado.
- **Partición de Equivalencia (PE)**: Para probar escenarios con 0 páginas, 1 página y múltiples páginas.
- **Pruebas de Transición de Estados**: Para validar el flujo de navegación entre páginas y el cambio en la URL.
- **Pruebas de Accesibilidad**: Validación de etiquetas ARIA y navegación por teclado.

## Casos de Prueba en Lenguaje Gherkin

### 1. Lógica de Navegación y Control

**Caso de Prueba 1: Botón Anterior deshabilitado en el límite inferior**

**Técnica: AVL**

#### Escenario (Gherkin)

```gherkin
Given el empleado está en la página "1" del historial de reconocimientos
When visualiza el componente de paginación
Then el botón "Anterior" debe estar deshabilitado para evitar navegación negativa
```

---

**Caso de Prueba 2: Botón Siguiente deshabilitado en el límite superior**

**Técnica: AVL**

#### Escenario (Gherkin)

```gherkin
Given el empleado está en la última página del historial (página "10" de "10")
When visualiza el componente de paginación
Then el botón "Siguiente" debe estar deshabilitado para indicar el fin de los datos
```

---

**Caso de Prueba 3: Ocultación automática por falta de volumen de datos**

**Técnica: PE**

#### Escenario (Gherkin)

```gherkin
Given el sistema tiene "15" kudos en total y el tamaño de página es "20"
When el empleado carga el historial de reconocimientos
Then el componente de paginación no debe ser visible en la interfaz
```

---

### 2. Representación Visual y Elipsis

**Caso de Prueba 4: Visualización de elipsis para navegación en grandes volúmenes**

**Técnica: PE**

#### Escenario (Gherkin)

```gherkin
Given existen "50" páginas de historial y el empleado se encuentra en la página "1"
When observa la lista de botones numéricos
Then debe ver los botones "1, 2, 3, 4, 5", seguido de una elipsis "..." y el número final "50"
```

---

**Caso de Prueba 5: Estilo visual distintivo para la página activa**

**Técnica: Transición de Estados**

#### Escenario (Gherkin)

```gherkin
Given el empleado navega a la página "3"
When el componente se renderiza
Then el botón numérico "3" debe mostrar el estilo destacado definido en Tailwind CSS diferente a los demás botones
```

---

### 3. Indicador de Rango Dinámico

**Caso de Prueba 6: Cálculo correcto del rango en la primera página**

**Técnica: AVL / Regla de Negocio**

#### Escenario (Gherkin)

```gherkin
Given existen "150" kudos totales y un tamaño de página de "20"
When el empleado está en la página "1"
Then el texto informativo debe decir "Mostrando 1-20 de 150 kudos"
```

---

**Caso de Prueba 7: Cálculo correcto del rango en una página intermedia**

**Técnica: PE**

#### Escenario (Gherkin)

```gherkin
Given existen "150" kudos totales y un tamaño de página de "20"
When el empleado navega a la página "2"
Then el texto informativo debe decir "Mostrando 21-40 de 150 kudos"
```

---

**Caso de Prueba 8: Ajuste de rango en la última página (Límite superior)**

**Técnica: AVL**

#### Escenario (Gherkin)

```gherkin
Given existen "45" kudos totales y un tamaño de página de "20"
When el empleado navega a la página "3"
Then el texto informativo debe decir "Mostrando 41-45 de 45 kudos"
```

---

### 4. Experiencia de Usuario (UX) y URL

**Caso de Prueba 9: Sincronización bidireccional con Query Parameters**

**Técnica: Transición de Estados**

#### Escenario (Gherkin)

```gherkin
Given el empleado hace clic en el botón de la página "4"
When la navegación se completa
Then la URL del navegador debe actualizarse a "?page=4" automáticamente
```

---

**Caso de Prueba 10: Responsividad del componente en móviles**

**Técnica: PE**

#### Escenario (Gherkin)

```gherkin
Given el empleado accede desde un dispositivo con resolución de "375px"
When visualiza el componente de paginación
Then los botones deben ajustar su espaciado y tamaño para ser táctiles sin solaparse
```

---

### 5. Accesibilidad (A11y)

**Caso de Prueba 11: Presencia de atributos ARIA descriptivos**

**Técnica: Prueba de Caja Negra / Accesibilidad**

#### Escenario (Gherkin)

```gherkin
Given el componente muestra el botón para la página "5"
When un lector de pantalla inspecciona el elemento
Then el botón debe tener el atributo aria-label con el valor "Ir a la página 5"
```

---

**Caso de Prueba 12: Navegación por teclado mediante Tabulador y Enter**

**Técnica: Transición de Estados / Accesibilidad**

#### Escenario (Gherkin)

```gherkin
Given el empleado no utiliza mouse y usa la tecla "Tab" para enfocarse en el botón "Siguiente"
When presiona la tecla "Enter" o "Espacio"
Then el sistema debe procesar el cambio a la página siguiente correctamente
```

---

## Tabla de Casos de Prueba - HU-016

| ID    | Caso de Prueba generado por la gema                           | Ajuste del realizado por el probador                                                                 | ¿Por qué se ajustó?                                                                                                              |
| ----- | ------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| TC-01 | Botón Anterior deshabilitado en el límite inferior            | Validar además que no se dispare ninguna llamada HTTP al hacer clic cuando está deshabilitado        | Desde enfoque ISTQB no solo validamos UI sino comportamiento. Evitamos tráfico innecesario y posibles inconsistencias de estado. |
| TC-02 | Botón Siguiente deshabilitado en el límite superior           | Verificar también que el cálculo del total de páginas se base en el total real devuelto por backend  | Riesgo de defecto en integración frontend-backend. Se aplica técnica de valores límite.                                          |
| TC-03 | Ocultación automática por falta de volumen de datos           | Validar comportamiento cuando total=0 (estado vacío con mensaje UX adecuado)                         | No solo se prueba ausencia de paginación, sino experiencia cuando no hay datos (caso alterno real).                              |
| TC-04 | Visualización de elipsis para navegación en grandes volúmenes | Agregar validación de comportamiento dinámico al cambiar tamaño de página (pageSize)                 | Riesgo funcional cuando cambia configuración. Aplicamos prueba combinatoria básica.                                              |
| TC-05 | Estilo visual distintivo para la página activa                | Validar además atributo aria-current="page"                                                          | Se amplía a accesibilidad (WCAG). No solo visual, también semántico.                                                             |
| TC-06 | Cálculo correcto del rango en la primera página               | Validar rango cuando total < pageSize                                                                | Valor límite inferior (ej: 3 registros con pageSize 10).                                                                         |
| TC-07 | Cálculo correcto del rango en una página intermedia           | Probar con dataset grande (>1000 registros simulados)                                                | Basado en riesgo por picos de volumen mencionados en el contexto.                                                                |
| TC-08 | Ajuste de rango en la última página (Límite superior)         | Validar comportamiento cuando último pageSize es parcial (ej: 95 registros, página 10 muestra 91-95) | Aplicación directa de técnica de valores límite ISTQB.                                                                           |
| TC-09 | Sincronización bidireccional con Query Parameters             | Validar persistencia del estado al refrescar navegador (F5) y compartir URL                          | Caso crítico SPA (React). Riesgo alto de inconsistencia de estado.                                                               |
| TC-10 | Responsividad del componente en móviles                       | Agregar prueba en breakpoints específicos (Tailwind: sm, md) y orientación landscape                 | No basta “se ve bien”, debe validarse contra diseño responsivo declarado en stack técnico.                                       |
| TC-11 | Presencia de atributos ARIA descriptivos                      | Validar compatibilidad con lector de pantalla (orden lógico de navegación)                           | Se amplía de prueba estática a prueba dinámica de accesibilidad.                                                                 |
| TC-12 | Navegación por teclado mediante Tabulador y Enter             | Agregar prueba con tecla Space y foco visible                                                        | Cumplimiento de estándares de accesibilidad y UX inclusivo.                                                                      |
