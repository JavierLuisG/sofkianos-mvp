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