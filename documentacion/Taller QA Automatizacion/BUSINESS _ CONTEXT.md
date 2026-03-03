# Plantilla para Diligenciar Contexto de Negocio

## 1. Descripción del Proyecto:

● **Nombre del Proyecto:** SofkianOS - Sistema Distribuido de Reconocimientos  

● **Objetivo del Proyecto:**  
SofkianOS transforma la identidad Sofkian en reconocimientos tangibles mediante Kudos. El sistema permite a los empleados de Sofka enviar reconocimientos instantáneos a sus compañeros a través de categorías de gamificación (Innovation, Teamwork, Passion, Mastery), fortaleciendo la cultura de reconocimiento en equipos geográficamente distribuidos. El objetivo central es procesar miles de reconocimientos de forma asíncrona sin bloqueos, garantizando respuestas inmediatas al usuario (HTTP 202) mientras el procesamiento de gamificación ocurre en segundo plano mediante mensajería.

## 2. Flujos Críticos del Negocio:

● **Principales Flujos de Trabajo:**  
- **Flujo de Envío de Kudo (Asíncrono):** El empleado ingresa desde la interfaz web (React SPA), completa el formulario de Kudo especificando remitente, destinatario, categoría y mensaje. El Producer API valida la información, responde inmediatamente con 202 Accepted y publica el evento en RabbitMQ. El Consumer Worker escucha la cola, procesa la lógica de gamificación y persiste el Kudo en PostgreSQL. Este flujo garantiza que el usuario nunca experimenta esperas visibles.

- **Flujo de Consulta de Kudos:** Los usuarios pueden visualizar el historial de Kudos enviados y recibidos mediante consultas paginadas. El sistema enmascara los emails para proteger la privacidad (ej: j***z@domain.com) y utiliza IDs hasheados no secuenciales para evitar enumeración de registros.

- **Flujo de Validación y Restricciones:** Todas las entradas son validadas en múltiples capas: frontend (Zod schemas), API REST (Jakarta Validation) y dominio (inmutabilidad mediante Java Records). Las reglas incluyen emails válidos obligatorios, mensaje entre 10-500 caracteres, categorías predefinidas y prohibición de auto-reconocimiento.

● **Módulos o Funcionalidades Críticas:**  
- **Producer API:** Gateway REST que actúa como punto de entrada único para recepción de Kudos. Implementa el patrón API Gateway, valida payloads mediante Jakarta Bean Validation y delega la entrega de mensajes al broker mediante Spring AMQP.

- **Consumer Worker:** Procesador asíncrono que implementa el patrón Event-Driven Consumer mediante @RabbitListener. Escucha la cola kudos.queue, ejecuta la lógica de gamificación (asignación de puntos según categoría) y persiste el estado final. Implementa Dead Letter Queue (DLQ) para manejo de errores con reintento automático.

- **Sistema de Mensajería (RabbitMQ):** Infraestructura crítica que desacopla Producer y Consumer. Utiliza Topic Exchange (kudos.exchange) con routing key (kudos.key) para enrutamiento flexible. Las colas implementan persistencia de mensajes (durable:true) para garantizar que los Kudos no se pierdan en caso de reinicio del broker.

- **Capa de Gamificación:** Lógica de negocio que asigna puntos según la categoría del Kudo: Innovation, Teamwork, Passion y Mastery. Esta capa reside en el Consumer Worker y es independiente del transporte de datos.

## 3. Reglas de Negocio y Restricciones:

● **Reglas de Negocio Relevantes:**  
- **Validación de Participantes:** Los campos "from" y "to" deben ser correos electrónicos válidos. El sistema verifica el formato mediante expresiones regulares y anotaciones @Email.

- **Prohibición de Auto-Reconocimiento:** Un empleado no puede enviarse un Kudo a sí mismo. Esta regla se valida tanto en frontend (Zod refine) como en el backend.

- **Categorías Cerradas:** El sistema solo acepta cuatro categorías predefinidas (Innovation, Teamwork, Passion, Mastery) representadas como enum en el dominio (KudoCategory). Cualquier valor fuera de este conjunto es rechazado con HTTP 400.

- **Longitud de Mensaje:** El mensaje de reconocimiento debe contener entre 10 y 500 caracteres. Esta restricción garantiza que los reconocimientos sean significativos sin abusar del almacenamiento.

- **Asincronía como Principio Fundamental:** El Producer API debe responder con HTTP 202 Accepted en menos de 100ms. El procesamiento completo (incluyendo persistencia) ocurre de forma asíncrona mediante la cola de mensajes, garantizando que el usuario nunca experimenta bloqueos.

- **Idempotencia Pendiente:** Actualmente el sistema no implementa idempotencia, lo que representa una deuda técnica identificada. Múltiples envíos del mismo Kudo pueden generar duplicados.

● **Regulaciones o Normativas:**  
- **Privacidad de Datos:** Los correos electrónicos se enmascaran en las consultas públicas para proteger la identidad de los participantes. Solo se muestra información parcial (ej: j***z@sofka.com).

- **Seguridad Pendiente:** El sistema actualmente opera sin autenticación ni autorización (API pública), lo cual está documentado como deuda técnica crítica. La implementación futura debe incluir OAuth 2.0 o JWT para proteger los endpoints.

- **Trazabilidad:** Aunque no existe tracing distribuido formal, todos los eventos son registrados mediante logs estructurados (SLF4J) que permiten seguir el flujo de un Kudo desde su entrada hasta su persistencia.

## 4. Perfiles de Usuario y Roles:

● **Perfiles o Roles de Usuario en el Sistema:**  
- **Empleado de Sofka (Único Rol Actual):** Todos los usuarios del sistema son empleados de Sofka que pueden tanto enviar como recibir Kudos. No existe diferenciación de roles, permisos o jerarquías en la versión actual del MVP.

● **Permisos y Limitaciones de Cada Perfil:**  
- **Empleado de Sofka:**
  - **Puede:** Enviar Kudos a cualquier otro empleado, visualizar el historial de Kudos (con datos enmascarados), seleccionar cualquiera de las cuatro categorías disponibles.
  - **No Puede:** Enviarse Kudos a sí mismo, modificar o eliminar Kudos enviados (inmutabilidad del reconocimiento), acceder a información completa de correos de otros usuarios (privacidad), administrar configuraciones del sistema (no existe rol de administrador en MVP).

**Nota:** El sistema actual carece de capa de autenticación, por lo que no existe validación real de identidad de usuario. Esta es una limitación conocida del MVP que debe ser resuelta en fases posteriores mediante la implementación de un sistema de autenticación y gestión de sesiones.

## 5. Condiciones del Entorno Técnico:

● **Plataformas Soportadas:**  
- **Frontend:** Navegadores web modernos (Chrome, Firefox, Safari, Edge) con soporte de ES2022 y módulos ESNext. La aplicación React está compilada con Vite y es responsiva mediante Tailwind CSS. Desplegada en Vercel con CDN global.

- **Backend (Producer API):** Contenedor Docker basado en OpenJDK 17, ejecutándose en instancias AWS EC2. Requiere mínimo 512MB de RAM y expone puerto 8082. Salud verificable mediante endpoint /api/v1/health.

- **Backend (Consumer Worker):** Contenedor Docker basado en OpenJDK 17, ejecutándose en instancias AWS EC2. Requiere mínimo 512MB de RAM y expone puerto 8081 (solo para health checks, no recibe tráfico HTTP de usuarios).

- **Infraestructura:** AWS EC2 para servicios backend, RabbitMQ (imagen oficial 3-management), PostgreSQL para persistencia. Orquestación mediante Docker Compose en tres configuraciones (development, test, production).

● **Tecnologías o Integraciones Clave:**  
- **Frontend:** React 19.2.0, TypeScript 5.9.3, Vite 7.2.4, Tailwind CSS 3.4.19, React Hook Form 7.71.1, Zod 4.3.6 para validación, Axios 1.13.4 para HTTP, React Router Dom 7.13.0.

- **Backend (Java):** Spring Boot 3.3.5, Java 17, Spring AMQP para RabbitMQ, Jakarta Bean Validation, Maven como build tool, Springdoc OpenAPI para documentación Swagger.

- **Mensajería:** RabbitMQ 3-management con Topic Exchange, configuración de Dead Letter Queue (DLQ) para manejo de errores, persistencia de mensajes habilitada (durable queues).

- **Base de Datos:** PostgreSQL con migraciones gestionadas mediante scripts SQL (futura integración con Flyway o Liquibase planificada).

- **Containerización & Orquestación:** Docker con multi-stage builds, Docker Compose para ambientes local/test/producción, redes bridge (sofkian-net) para comunicación entre servicios.

- **Infraestructura como Código:** Terraform para provisioning en AWS (VPC, EC2, Security Groups, User Data scripts).

- **Observabilidad:** Dozzle para visualización en tiempo real de logs de contenedores, RabbitMQ Management UI para monitoreo de colas, health checks en todos los servicios.

- **Testing:** JUnit 5 para backend, Vitest + Testing Library para frontend, Gatling para pruebas de carga (simulaciones de stress en consumer), cobertura de código mediante JaCoCo (objetivo >80% en próximas iteraciones).

- **CI/CD:** Jenkins con pipelines declarativos (Jenkinsfile) para build, test y deploy. GitHub como repositorio de código con protección de ramas (main y develop).

## 6. Casos Especiales o Excepciones (Opcional):

● **Escenarios Alternos o Excepciones que Deben Considerarse:**  

- **Indisponibilidad de RabbitMQ:** Si el broker de mensajes no está disponible, el Producer API debe responder con HTTP 503 Service Unavailable. Los Kudos enviados durante este período se pierden ya que no hay mecanismo de retry en el productor (deuda técnica identificada).

- **Falla en Consumer Worker:** Si el consumidor falla al procesar un mensaje (ej: error de validación, excepción de base de datos), el mensaje es reenviado automáticamente a la Dead Letter Queue (kudos.dlq). RabbitMQ reintenta la entrega según la configuración de TTL y max-retries. Los mensajes en DLQ requieren intervención manual o procesamiento especial.

- **Duplicados de Mensajes:** Debido a la ausencia de idempotencia, si un usuario envía el mismo Kudo múltiples veces (ej: double-click en el botón de envío), el sistema creará registros duplicados. Esta situación está identificada como deuda técnica y requiere implementación de claves de idempotencia o deduplicación basada en hash del contenido.

- **Volumen Extremo (Picos de Tráfico):** Durante eventos masivos de reconocimiento (ej: fin de sprint, celebraciones), la cola de RabbitMQ puede acumular miles de mensajes. El sistema está diseñado para escalar horizontalmente agregando instancias adicionales del Consumer Worker que procesan mensajes en paralelo desde la misma cola (competing consumers pattern).

- **Validación Fallida:** Si un payload enviado al Producer API no cumple con las validaciones (email inválido, categoría desconocida, mensaje muy corto), el sistema responde con HTTP 400 Bad Request incluyendo un objeto ApiError estructurado con timestamp, status, reason, message y path. Los errores de validación no generan eventos en RabbitMQ.

- **Concurrencia en Base de Datos:** En escenarios de alta concurrencia, múltiples consumers pueden intentar escribir simultáneamente en PostgreSQL. El sistema confía en las garantías ACID de la base de datos y el aislamiento de transacciones (nivel READ_COMMITTED por defecto). No se implementan locks optimistas ni pesimistas en el MVP.

- **Datos Enmascarados en Frontend:** Las consultas de Kudos retornan emails enmascarados (j***z@domain.com) para proteger privacidad. Los usuarios pueden no reconocer fácilmente a quién se envió o de quién recibieron un Kudo. Esta es una decisión de diseño intencional que prioriza privacidad sobre usabilidad.
