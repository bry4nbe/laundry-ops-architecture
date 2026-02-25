# Laundry ERP — Documentación del Proyecto

Sistema de gestión para lavanderías diseñado para reemplazar el registro manual en papel por una plataforma digital accesible desde cualquier dispositivo, incluyendo celulares.

---

## Descripción del Problema

Actualmente, la lavandería opera con tickets en papel autocopiativo: una copia queda en el local y otra se entrega al cliente para que la presente al recoger su ropa. Este proceso genera los siguientes problemas:

- No hay visibilidad en tiempo real de las órdenes pendientes, en proceso o listas para entrega.
- El control de pagos es manual, dificultando saber qué órdenes tienen saldo pendiente.
- No existe historial digital de clientes ni de sus órdenes anteriores.
- La operadora principal del local no cuenta con herramientas digitales adaptadas a su perfil de usuario.

---

## Solución

**Laundry ERP** es una plataforma web con diseño responsive design que permite:

- Registrar órdenes de lavado desde un celular de forma simple y rápida.
- Gestionar pagos parciales (adelantos y saldos pendientes).
- Hacer seguimiento del estado de cada orden.
- Visualizar ingresos y métricas clave desde un dashboard.

---

## Stack Tecnológico

| Capa | Tecnología |
|---|---|
| Frontend | React + Tailwind CSS |
| Backend | Django + Django REST Framework |
| Base de datos | PostgreSQL |
| Autenticación | JWT |
| Deploy Frontend | Vercel |
| Deploy Backend | Railway |
| CI/CD | GitHub Actions |
| Documentación API | Swagger / OpenAPI |

---

## Repositorios

| Repositorio | Descripción |
|---|---|
| [laundry-erp-backend](https://github.com/laundry-erp/laundry-erp-backend) | API REST con Django |
| [laundry-erp-frontend](https://github.com/laundry-erp/laundry-erp-frontend) | Interfaz web con React + Tailwind |
| [laundry-erp-docs](https://github.com/bry4nbe/laundry-erp-docs) | Documentación del proyecto (este repo) |

---

## Estructura de este repositorio

```
laundry-ops-architecture/
├── README.md
├── product/
│   ├── problem-and-solution.md
│   └── user-stories.md
├── technical/
│   ├── stack-decisions.md
│   ├── database-design.md
│   ├── erd.png
│   ├── c4-context.png
│   └── c4-container.png
└── infrastructure/
    ├── deployment.md
    └── ci-cd.md
```

---

## Estado del Proyecto

> 🟡 En desarrollo — Fase de documentación y diseño

---

## Autor

Desarrollado por **Bryan Barba**.
Stack: Django · React · PostgreSQL · Tailwind CSS