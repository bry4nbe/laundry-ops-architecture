# Stack Tecnológico — Laundry Ops

> Documento de decisiones arquitectónicas. Cada sección responde tres preguntas: cuál es el problema concreto, qué alternativas se evaluaron y por qué se tomó esta decisión específica. El objetivo es que cualquier decisión sea defendible con evidencia, no con preferencias.

---

## Contexto del Proyecto

Sistema de gestión operativa para una lavandería con una sola sucursal, orientado a resolver tres problemas reales del negocio: pérdida de control sobre órdenes en proceso, dificultad para saber cuánto se cobró y cuánto se debe en el día, y ausencia de trazabilidad en el servicio de lavado al seco enviado a terceros.

El sistema tiene dos roles (administrador y operador), opera desde dispositivos móviles en el mostrador y desde desktop en la administración, y es mantenido por un solo desarrollador.

**Sobre la nomenclatura:** El término **Laundry Management Software** se eligió sobre ERP o BMS porque describe con precisión el alcance real del sistema: gestión operativa y financiera específica para una lavandería, sin pretender cubrir módulos de un ERP completo (contabilidad general, inventario de insumos, nómina). Sistemas comerciales de referencia como CleanCloud y Cents operan bajo esta misma categoría.

---

## Decisiones Arquitectónicas Transversales

### Monolito modular sobre microservicios

Para el volumen y equipo de este proyecto (una sucursal, un desarrollador, decenas de órdenes diarias), una arquitectura de microservicios introduce complejidad operativa sin beneficio real: orquestación de servicios, latencia entre llamadas internas, múltiples bases de datos que gestionar. Se eligió un monolito modular donde la separación de responsabilidades se implementa a nivel de código mediante capas (`models`, `services`, `serializers`, `views`), no a nivel de infraestructura. Esta decisión es revisable si el sistema escala a múltiples sucursales.

### Multi-tenancy: exclusión consciente en v1

El sistema está diseñado para una sola sucursal. Escalar a múltiples lavanderías requeriría una estrategia de multi-tenancy que impacta el modelo de datos desde la base: ya sea mediante `tenant_id` en cada tabla o mediante esquemas separados por cliente en PostgreSQL. Implementar esto en v1 sin un caso de uso real validado sería over-engineering. La decisión está documentada aquí para que sea una elección consciente, no una omisión, y para que cualquier trabajo de migración futuro parta de un diagnóstico claro del estado actual.

---

## Backend

### Lenguaje: Python 3.12

**Problema:** El backend necesita un lenguaje con ecosistema maduro para APIs REST, ORMs y migraciones de base de datos, con buena demanda en el mercado laboral peruano.

**Alternativas evaluadas:** Python, JavaScript/Node.js, Java.

**Decisión:** Python. Tiene el ecosistema más completo para sistemas de gestión empresarial en el stack elegido y es el lenguaje nativo de Django. Node.js fue descartado porque para este dominio, con lógica financiera compleja y modelo de datos relacional con múltiples FKs, un ORM tipado como el de Django ofrece mayor seguridad y productividad que el enfoque orientado a I/O de Node. Java fue descartado por curva de aprendizaje alta y menor velocidad de desarrollo en un proyecto de un solo desarrollador.

---

### Framework: Django 5 + Django REST Framework

**Problema:** Se necesita un framework que provea ORM, migraciones, autenticación, sistema de permisos y administración de datos sin construir esas capas desde cero.

**Alternativas evaluadas:**

| Framework | ORM propio | Migraciones | Admin panel | Autenticación incluida |
|---|---|---|---|---|
| **Django + DRF** | ✅ | ✅ automáticas | ✅ | ✅ |
| FastAPI | ❌ SQLAlchemy | ❌ Alembic | ❌ | ❌ |
| Flask | ❌ SQLAlchemy | ❌ Alembic | ❌ | ❌ |
| Spring Boot | ✅ JPA | ✅ Liquibase | ❌ | ✅ Spring Security |

**Decisión:** Django + DRF, por tres razones concretas:

**Django Admin como herramienta operativa real.** El panel `/admin` permite corregir datos mal ingresados, gestionar usuarios y consultar registros sin construir pantallas adicionales. Para un sistema en producción con un solo desarrollador, esto elimina una categoría completa de trabajo de mantenimiento que en otros frameworks requeriría semanas de desarrollo adicional.

**Migraciones automáticas críticas para desarrollo iterativo.** Se generan con `makemigrations` a partir de cambios en los modelos Python, sin SQL manual. FastAPI y Flask requieren configurar Alembic por separado con su propio ciclo de trabajo.

**Arquitectura por capas demostrable.** La lógica de negocio vive en `services.py`, separada de las vistas y los serializadores. Esto mantiene la lógica testeable de forma independiente y demuestra criterio de diseño más allá del uso básico del framework.

FastAPI fue la segunda opción más evaluada. Su rendimiento y documentación Swagger automática son ventajas reales, pero para una lavandería sin miles de requests concurrentes el rendimiento no es el cuello de botella, y el costo de configurar autenticación, ORM y migraciones manualmente no se justifica en este contexto.

---

### Autenticación: JWT con djangorestframework-simplejwt

**Problema:** El sistema tiene dos roles con permisos distintos. Se necesita un mecanismo de autenticación compatible con una SPA en React que no comparte dominio con el backend.

**Alternativas evaluadas:** Django Sessions, JWT con SimpleJWT, OAuth2.

**Decisión:** JWT con `djangorestframework-simplejwt` por razones técnicas concretas:

Django Sessions almacena estado en el servidor y usa cookies de sesión. Esto funciona bien cuando frontend y backend comparten dominio, pero genera problemas de CORS en una SPA servida desde un puerto o dominio distinto. JWT es stateless: el token viaja en el header `Authorization`, lo que simplifica la configuración CORS y es el estándar adoptado por DRF para APIs consumidas por SPAs.

**Flujo implementado:** login devuelve `access_token` (15 minutos de vida) y `refresh_token` (7 días). El frontend almacena el refresh token en una cookie HTTP-only para protegerlo de ataques XSS, y el access token en memoria. Cuando el access token expira, el frontend solicita uno nuevo con el refresh token sin interrumpir al usuario.

Los permisos por rol se implementan con permission classes de DRF validadas en cada endpoint: operadores solo pueden crear y consultar órdenes y pagos, los administradores tienen acceso completo incluyendo catálogo, reportes y gestión de usuarios.

OAuth2 fue descartado por complejidad de configuración sin justificación para un sistema sin login social ni delegación de permisos a terceros.

---

### Base de Datos: PostgreSQL 16

**Problema:** Almacenar órdenes, clientes, pagos y catálogo con integridad referencial garantizada y aritmética monetaria exacta.

**Decisión:** PostgreSQL. El tipo `DECIMAL(10, 2)` garantiza precisión exacta en montos, eliminando los errores de redondeo que ocurren con `FLOAT`. En un sistema financiero, un error de punto flotante acumulado en los totales de caja es un bug con impacto real en el dinero del negocio. PostgreSQL es además el motor estándar en producción Django. SQLite fue descartado por limitaciones de concurrencia. MySQL fue descartado porque PostgreSQL es el estándar de facto en el ecosistema Django.

---

### Almacenamiento de Archivos: Cloudinary

**Problema:** Las órdenes de lavado al seco requieren adjuntar foto del ticket físico. El servidor de aplicaciones no debe gestionar almacenamiento de archivos en producción.

**Decisión:** Cloudinary. Ofrece 25 GB en tier gratuito, suficiente para el volumen del negocio, tiene SDK oficial para Python y provee optimización automática de imágenes. AWS S3 fue descartado porque su configuración (IAM, bucket policies, CORS) introduce complejidad operativa sin beneficio real para una sola sucursal. El almacenamiento local fue descartado porque se pierde al redesplegar el servidor.

---

### Testing: Pytest + pytest-django

**Problema:** El sistema maneja lógica financiera (cálculo de totales, pagos parciales, cambios de estado de orden) que no puede verificarse de forma confiable solo con pruebas manuales. Un error en el cálculo del saldo pendiente de una orden es un bug con impacto directo en la caja del negocio.

**Decisión:** Pytest con el plugin `pytest-django`. Las pruebas se concentran en la capa `services.py` donde vive la lógica de negocio, no en las vistas ni en los serializadores. Esta separación es exactamente lo que justificó la arquitectura por capas: hace que la lógica sea testeable de forma aislada sin levantar el servidor HTTP.

**Casos de prueba prioritarios en v1:**

- Cálculo correcto de `total_amount` en órdenes con múltiples items (por prenda y por kilo).
- Validación de que un pago no puede exceder el saldo pendiente de la orden.
- Transiciones de estado de orden: las transiciones inválidas deben ser rechazadas explícitamente.
- Cálculo del balance diario: total cobrado vs total pendiente en el dashboard.

---

## Infraestructura y Despliegue

### Hosting: Railway (backend) + Vercel (frontend)

**Problema:** El sistema necesita estar disponible con una URL pública para ser validado por el negocio real y demostrado en entrevistas. Un sistema que solo funciona en localhost no existe en producción.

**Decisión:** Railway para el backend Django y PostgreSQL. Vercel para el frontend React.

Railway tiene tier gratuito con $5 de crédito mensual, suficiente para una sucursal con tráfico bajo. Backend y base de datos corren como servicios separados dentro del mismo proyecto, con variables de entorno gestionadas en su panel. Vercel tiene tier gratuito sin restricciones para SPAs estáticas.

**Costo operativo mensual estimado:**

| Servicio | Costo |
|---|---|
| Railway (backend + PostgreSQL) | $0 – $5 USD |
| Vercel (frontend) | $0 |
| Cloudinary (imágenes) | $0 (tier gratuito) |
| **Total mensual** | **$0 – $5 USD** |

Este costo es significativamente menor al sistema actual que reemplaza (S/540/año ≈ $144/año).

---

### Contenedores: Docker + Docker Compose

**Decisión:** El backend incluye `Dockerfile` y `docker-compose.yml` para desarrollo local. Docker garantiza paridad entre el entorno local y producción, eliminando la categoría de bugs "funciona en mi máquina". El `docker-compose.yml` levanta Django y PostgreSQL con un solo comando, y en v2 incluirá Redis para Celery.

---

### CI/CD: GitHub Actions

**Decisión:** Pipeline en GitHub Actions que se ejecuta en cada push a `main`:

1. Instala dependencias y ejecuta `pytest`.
2. Si los tests pasan, construye la imagen Docker.
3. Despliega en Railway mediante su CLI.

Esto garantiza que ningún código que rompa los tests llegue a producción, y documenta el proceso de deploy de forma reproducible y auditable.

---

## Frontend

### Framework: React 19 + TypeScript

**Problema:** La interfaz necesita actualizar múltiples secciones simultáneamente (métricas del dashboard, lista de órdenes, indicadores de pago) sin recargar la página, y manejar un modelo de datos con relaciones entre órdenes, items y pagos.

**Decisión:** React con TypeScript. El modelo de componentes de React permite construir interfaces complejas de forma modular. TypeScript agrega tipado estático que previene errores al trabajar con los tipos del dominio: una `Order` tiene `OrderItem[]` y `Payment[]`, y confundir un `id` de orden con un `id` de pago es exactamente el tipo de error que TypeScript atrapa antes de llegar a producción. Vue.js fue descartado por menor demanda en el mercado peruano. Next.js fue descartado porque el sistema no tiene requerimientos de SEO ni renderizado en servidor; es una aplicación administrativa interna donde Next.js agrega complejidad sin beneficio.

---

### Estilos: Tailwind CSS v4

**Problema:** El sistema necesita diseño responsive adaptado a dos contextos de uso: registro de órdenes desde móvil en el mostrador y revisión del dashboard desde desktop.

**Decisión:** Tailwind CSS v4. Permite diseño completamente personalizado sin las restricciones visuales de Bootstrap o los estilos genéricos de Material UI. El bundle final incluye solo las clases efectivamente utilizadas gracias al purging automático, resultando en archivos CSS más pequeños que Bootstrap 5. Tailwind v4 introduce además mejoras de rendimiento en el proceso de build respecto a v3.

---

### Template Base: TailAdmin (MIT License)

**Problema:** Construir desde cero sidebar, tablas con filtros, modales, formularios y gráficas de dashboard consume semanas de desarrollo que no aportan valor al dominio del problema.

**Decisión:** TailAdmin como punto de partida estructural, por dos razones técnicas:

**Sin dependencias de librerías de componentes externas.** A diferencia de Material UI o Ant Design, TailAdmin construye sus componentes directamente con Tailwind. El código vive en el repositorio del proyecto, puede modificarse sin restricciones de API externas y no tiene riesgo de breaking changes entre versiones de librerías de terceros.

**Stack idéntico al del proyecto.** React 19 + TypeScript + Tailwind CSS v4, sin introducir dependencias adicionales.

Los componentes de TailAdmin fueron adaptados al dominio de lavandería: la tabla de órdenes incluye columnas de estado y saldo pendiente específicas del negocio, el formulario de registro maneja items dinámicos con cálculo de total en tiempo real, y el dashboard muestra las métricas operativas definidas en las user stories. El template provee la estructura; la lógica de dominio es propia.

---

### Gráficas: ApexCharts

**Decisión:** ApexCharts mediante `react-apexcharts`, integrado en TailAdmin. Provee los tipos de visualización requeridos: barras para ingresos por período y donut para distribución de métodos de pago (efectivo vs Yape/Plin vs otros).

---

### Routing: React Router v7

**Decisión:** React Router v7, estándar de facto para routing en SPAs React, ya integrado en TailAdmin. Gestiona la navegación entre módulos (dashboard, órdenes, clientes, catálogo) sin recargar la página.

---

## Herramientas de Desarrollo

### Build Tool: Vite

**Decisión:** Vite, sucesor del deprecado Create React App. Su Hot Module Replacement hace que los cambios se reflejen en el navegador en milisegundos durante el desarrollo.

---

### Control de Versiones: Git + GitHub

**Decisión:** Tres repositorios:

| Repositorio | Contenido |
|---|---|
| `laundry-ops-api` | Backend Django + DRF |
| `laundry-ops-web` | Frontend React + TypeScript |
| `laundry-ops-architecture` | Diagramas C4, ERD, decisiones técnicas |

`laundry-ops-architecture` se mantiene separado del código porque los artefactos de diseño tienen un ciclo de vida distinto al código fuente y deben ser consultables sin necesidad de clonar el proyecto completo.

---

*Última actualización: Febrero 2026*