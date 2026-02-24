# User Stories — Laundry ERP (MVP v1)

## Convención

Cada historia sigue el formato:
> **Como** [rol], **quiero** [acción], **para** [beneficio].

Los criterios de aceptación usan el formato **Gherkin (Given / When / Then)**:
- **Dado que** → contexto previo o condición inicial
- **Cuando** → acción que ejecuta el usuario
- **Entonces** → resultado esperado del sistema
- **Y** → condición adicional encadenada

---

## Módulo 1: Autenticación

---

### US-01 — Iniciar sesión
**Como** usuario del sistema (administrador u operador), **quiero** iniciar sesión con mi correo y contraseña, **para** acceder a las funcionalidades según mi rol.

**Criterios de aceptación:**

**Escenario 1: Inicio de sesión exitoso como administrador**
```gherkin
Dado que el usuario tiene una cuenta con rol administrador
Cuando ingresa su correo y contraseña correctos
Entonces el sistema lo redirige al dashboard completo
Y tiene acceso a todas las funcionalidades del sistema
```

**Escenario 2: Inicio de sesión exitoso como operador**
```gherkin
Dado que el usuario tiene una cuenta con rol operador
Cuando ingresa su correo y contraseña correctos
Entonces el sistema lo redirige a la vista simplificada de órdenes
Y solo tiene acceso a crear órdenes, registrar pagos y cambiar estados
```

**Escenario 3: Credenciales incorrectas**
```gherkin
Dado que el usuario se encuentra en la pantalla de login
Cuando ingresa un correo o contraseña incorrectos
Entonces el sistema muestra un mensaje de error
Y no permite el acceso al sistema
```

**Escenario 4: Persistencia de sesión**
```gherkin
Dado que el usuario inició sesión desde su celular
Cuando cierra el navegador y vuelve a abrir la aplicación
Entonces el sistema mantiene la sesión activa sin solicitar nuevamente las credenciales
```

---

### US-02 — Cerrar sesión
**Como** usuario, **quiero** poder cerrar sesión, **para** proteger el acceso al sistema cuando no lo estoy usando.

**Criterios de aceptación:**

**Escenario 1: Cerrar sesión exitosamente**
```gherkin
Dado que el usuario se encuentra en cualquier pantalla del sistema
Cuando presiona el botón de cerrar sesión
Entonces el sistema invalida la sesión activa
Y redirige al usuario a la pantalla de login
```

---

## Módulo 2: Clientes

---

### US-03 — Registrar cliente
**Como** operador o administrador, **quiero** registrar un nuevo cliente con sus datos básicos, **para** asociarlo a sus órdenes de lavado.

**Criterios de aceptación:**

**Escenario 1: Registro exitoso**
```gherkin
Dado que el usuario se encuentra en el formulario de nuevo cliente
Cuando ingresa el nombre completo y presiona guardar
Entonces el sistema registra al cliente
Y lo deja disponible para ser seleccionado al crear una orden
```

**Escenario 2: Teléfono duplicado**
```gherkin
Dado que ya existe un cliente registrado con un número de teléfono
Cuando otro usuario intenta registrarse con el mismo número
Entonces el sistema muestra un mensaje indicando que el teléfono ya está registrado
Y no permite guardar el registro
```

**Escenario 3: Registro sin teléfono**
```gherkin
Dado que el usuario se encuentra en el formulario de nuevo cliente
Cuando ingresa solo el nombre sin teléfono y presiona guardar
Entonces el sistema registra al cliente correctamente
Y el campo teléfono queda vacío
```

---

### US-04 — Buscar cliente
**Como** operador o administrador, **quiero** buscar un cliente por nombre o teléfono, **para** encontrarlo rápidamente al crear una orden.

**Criterios de aceptación:**

**Escenario 1: Búsqueda con resultados**
```gherkin
Dado que el usuario se encuentra en el buscador de clientes
Cuando escribe al menos 2 caracteres del nombre o teléfono
Entonces el sistema muestra en tiempo real los clientes que coinciden con la búsqueda
```

**Escenario 2: Búsqueda sin resultados**
```gherkin
Dado que el usuario realiza una búsqueda
Cuando no existe ningún cliente que coincida con el texto ingresado
Entonces el sistema muestra un mensaje de "Cliente no encontrado"
Y ofrece la opción de registrar un nuevo cliente con ese nombre
```

---

### US-05 — Ver historial de un cliente
**Como** administrador, **quiero** ver el historial de órdenes de un cliente, **para** conocer sus servicios previos y saldos pendientes.

**Criterios de aceptación:**

**Escenario 1: Ver historial**
```gherkin
Dado que el administrador selecciona un cliente de la lista
Cuando accede a su perfil
Entonces el sistema muestra todas sus órdenes de más reciente a más antigua
Y muestra el estado de cada orden, el total cobrado y el saldo pendiente acumulado
```

---

## Módulo 3: Órdenes

---

### US-06 — Crear orden de lavado por prenda
**Como** operador o administrador, **quiero** registrar una orden de lavado por prenda, **para** llevar el control de las prendas que dejó el cliente.

**Criterios de aceptación:**

**Escenario 1: Crear orden exitosamente**
```gherkin
Dado que el usuario seleccionó un cliente y agregó al menos una prenda
Cuando presiona "Guardar orden"
Entonces el sistema genera un número de orden único
Y registra la orden con estado "Recibido"
Y calcula el total automáticamente según los precios del catálogo
```

**Escenario 2: Precio editable por prenda**
```gherkin
Dado que el usuario está agregando una prenda a la orden
Cuando modifica manualmente el precio unitario
Entonces el sistema usa el precio modificado para calcular el total
Y no sobreescribe el cambio con el precio del catálogo
```

**Escenario 3: Prenda marcada como lavado al seco**
```gherkin
Dado que el usuario está agregando una prenda a la orden
Cuando marca la opción "Lavado al seco"
Entonces el sistema habilita el campo "Costo al tercero"
Y calcula automáticamente la ganancia neta (precio público − costo tercero)
```

---

### US-07 — Crear orden de lavado por kilo
**Como** operador o administrador, **quiero** registrar una orden de lavado por kilo, **para** cobrar según el peso de la ropa entregada.

**Criterios de aceptación:**

**Escenario 1: Crear orden por kilo exitosamente**
```gherkin
Dado que el usuario seleccionó un cliente e ingresó el peso en kilogramos
Cuando presiona "Guardar orden"
Entonces el sistema genera un número de orden único
Y calcula el total multiplicando el peso por el precio por kilo configurado en el catálogo
Y registra la orden con estado "Recibido"
```

**Escenario 2: Precio por kilo editable**
```gherkin
Dado que el usuario está creando una orden por kilo
Cuando modifica manualmente el precio por kilo
Entonces el sistema usa el precio modificado para calcular el total
```

---

### US-08 — Seguimiento de prendas en lavado al seco
**Como** administrador, **quiero** hacer seguimiento de las prendas enviadas al tercero, **para** saber cuáles están pendientes de retorno y en qué fecha fueron enviadas.

**Criterios de aceptación:**

**Escenario 1: Ver prendas pendientes de retorno**
```gherkin
Dado que el administrador accede al módulo de lavado al seco
Cuando visualiza el listado
Entonces el sistema muestra todas las prendas enviadas al tercero que aún no han sido retornadas
Y cada prenda muestra: cliente, tipo de prenda, fecha de envío y días transcurridos
```

**Escenario 2: Registrar envío al tercero**
```gherkin
Dado que una orden tiene prendas marcadas como lavado al seco
Cuando el operador o administrador registra el envío
Entonces el sistema solicita seleccionar la fecha de envío (lunes, miércoles o viernes)
Y actualiza el estado de la prenda a "Enviado al tercero"
```

**Escenario 3: Registrar retorno del tercero**
```gherkin
Dado que una prenda se encuentra en estado "Enviado al tercero"
Cuando el operador registra su retorno
Entonces el sistema actualiza el estado de la prenda a "Retornado"
Y actualiza automáticamente el estado de la orden asociada a "Listo"
```

**Escenario 4: Alerta por demora**
```gherkin
Dado que una prenda fue enviada al tercero
Cuando han pasado más de 2 días hábiles sin registrar su retorno
Entonces el sistema resalta la prenda como pendiente de seguimiento en el listado
```

---

### US-09 — Adjuntar foto del ticket físico a una orden
**Como** operador o administrador, **quiero** tomar una foto al ticket de papel y adjuntarla a la orden, **para** tener un respaldo visual del registro manual.

**Criterios de aceptación:**

**Escenario 1: Adjuntar foto al crear una orden**
```gherkin
Dado que el usuario está creando o editando una orden
Cuando selecciona la opción de adjuntar foto
Entonces el sistema permite tomar una foto con la cámara del celular o seleccionarla desde la galería
Y guarda la foto asociada a la orden
```

**Escenario 2: Ver foto adjunta**
```gherkin
Dado que una orden tiene una foto adjunta
Cuando el usuario accede al detalle de la orden
Entonces el sistema muestra la foto del ticket como referencia visual
```

**Escenario 3: Orden sin foto**
```gherkin
Dado que el usuario está creando una orden
Cuando no adjunta ninguna foto y presiona guardar
Entonces el sistema guarda la orden correctamente sin foto
Y no bloquea el proceso por la ausencia de imagen
```

---

### US-10 — Ver detalle de una orden
**Como** operador o administrador, **quiero** ver el detalle completo de una orden, **para** conocer las prendas, el estado, los pagos realizados y el saldo pendiente.

**Criterios de aceptación:**

**Escenario 1: Ver detalle completo**
```gherkin
Dado que el usuario selecciona una orden de la lista
Cuando accede a su detalle
Entonces el sistema muestra: cliente, tipo de servicio, detalle de prendas o peso, total, historial de pagos, saldo pendiente y estado actual
Y si tiene foto adjunta la muestra disponible para visualizar
```

---

### US-11 — Cambiar estado de una orden
**Como** operador o administrador, **quiero** actualizar el estado de una orden, **para** reflejar en qué etapa del proceso se encuentra la ropa del cliente.

**Criterios de aceptación:**

**Escenario 1: Avanzar estado exitosamente**
```gherkin
Dado que una orden se encuentra en cualquier estado excepto "Entregado"
Cuando el usuario selecciona avanzar al siguiente estado
Entonces el sistema actualiza el estado de la orden
Y registra la fecha y hora del cambio
```

**Escenario 2: No se puede retroceder estado**
```gherkin
Dado que una orden se encuentra en un estado avanzado
Cuando el usuario intenta retroceder al estado anterior
Entonces el sistema no permite la acción
Y muestra un mensaje indicando que el estado no puede retrocederse
```

**Escenario 3: Alerta al llegar a "Listo"**
```gherkin
Dado que una orden avanza al estado "Listo"
Entonces el sistema muestra la orden resaltada en el dashboard como lista para entregar
```

---

### US-12 — Listar órdenes con filtros
**Como** administrador, **quiero** ver la lista de órdenes aplicando filtros, **para** encontrar rápidamente las órdenes que necesito gestionar.

**Criterios de aceptación:**

**Escenario 1: Filtrar órdenes**
```gherkin
Dado que el administrador se encuentra en el listado de órdenes
Cuando aplica uno o más filtros (estado, fecha, tipo de servicio)
Entonces el sistema muestra únicamente las órdenes que coinciden con los filtros aplicados
Y cada resultado muestra: número de orden, cliente, total, estado y fecha de ingreso
```

**Escenario 2: Sin resultados**
```gherkin
Dado que el administrador aplica filtros
Cuando no existe ninguna orden que coincida
Entonces el sistema muestra un mensaje de "No se encontraron órdenes"
```

---

## Módulo 4: Pagos

---

### US-13 — Registrar pago adelantado al crear una orden
**Como** operador o administrador, **quiero** registrar un adelanto al momento de crear la orden, **para** dejar constancia de lo que el cliente ya pagó.

**Criterios de aceptación:**

**Escenario 1: Registrar adelanto parcial**
```gherkin
Dado que el usuario está creando una orden
Cuando ingresa un monto de adelanto menor al total y selecciona el método de pago
Entonces el sistema registra el adelanto
Y calcula y muestra el saldo pendiente automáticamente
```

**Escenario 2: Orden sin adelanto**
```gherkin
Dado que el usuario está creando una orden
Cuando deja el monto de adelanto en S/0 y guarda la orden
Entonces el sistema registra la orden con saldo pendiente igual al total
```

**Escenario 3: Pago completo al crear la orden**
```gherkin
Dado que el usuario está creando una orden
Cuando ingresa un adelanto igual al total de la orden
Entonces el sistema marca la orden como completamente pagada
Y el saldo pendiente queda en S/0
```

---

### US-14 — Registrar pago de saldo pendiente
**Como** operador o administrador, **quiero** registrar el pago del saldo pendiente de una orden, **para** marcar la deuda como saldada al momento de la entrega.

**Criterios de aceptación:**

**Escenario 1: Pago total del saldo**
```gherkin
Dado que una orden tiene saldo pendiente
Cuando el usuario registra un pago igual al saldo pendiente
Entonces el sistema marca la orden como completamente pagada
Y el saldo pendiente queda en S/0
```

**Escenario 2: Pago parcial del saldo**
```gherkin
Dado que una orden tiene saldo pendiente
Cuando el usuario registra un pago menor al saldo pendiente
Entonces el sistema actualiza el saldo pendiente restando el monto pagado
Y la orden mantiene deuda pendiente
```

---

### US-15 — Ver historial de pagos de una orden
**Como** administrador, **quiero** ver todos los pagos registrados para una orden, **para** tener trazabilidad de cuándo y cuánto pagó el cliente.

**Criterios de aceptación:**

**Escenario 1: Ver historial de pagos**
```gherkin
Dado que el administrador accede al detalle de una orden
Cuando visualiza la sección de pagos
Entonces el sistema muestra cada pago con: monto, método de pago, fecha y hora
Y muestra el total pagado acumulado y el saldo pendiente actual
```

---

## Módulo 5: Catálogo de Servicios

---

### US-16 — Gestionar precios de prendas
**Como** administrador, **quiero** configurar el listado de prendas con sus precios unitarios, **para** que se carguen automáticamente al crear una orden.

**Criterios de aceptación:**

**Escenario 1: Agregar prenda al catálogo**
```gherkin
Dado que el administrador se encuentra en el catálogo de prendas
Cuando agrega una nueva prenda con nombre y precio unitario
Entonces la prenda queda disponible para seleccionarse al crear nuevas órdenes
```

**Escenario 2: Desactivar prenda**
```gherkin
Dado que el administrador desactiva una prenda del catálogo
Entonces la prenda deja de aparecer en el formulario de nuevas órdenes
Y las órdenes anteriores que la contenían conservan su información sin cambios
```

---

### US-17 — Configurar precio por kilo
**Como** administrador, **quiero** configurar el precio del lavado por kilo, **para** que se aplique automáticamente al crear órdenes de este tipo.

**Criterios de aceptación:**

**Escenario 1: Actualizar precio por kilo**
```gherkin
Dado que el administrador accede a la configuración del catálogo
Cuando modifica el precio por kilo y guarda el cambio
Entonces el nuevo precio aplica únicamente a las órdenes creadas a partir de ese momento
Y las órdenes existentes conservan el precio con el que fueron registradas
```

---

## Módulo 6: Dashboard

---

### US-18 — Ver resumen del día
**Como** administrador, **quiero** ver un resumen de las operaciones del día al ingresar al sistema, **para** tener una visión rápida del estado del negocio.

**Criterios de aceptación:**

**Escenario 1: Ver resumen actualizado**
```gherkin
Dado que el administrador accede al dashboard
Cuando la pantalla carga
Entonces el sistema muestra: número de órdenes ingresadas hoy, órdenes listas para entregar, ingresos cobrados hoy e ingresos pendientes de cobro
Y los datos reflejan el estado actual sin necesidad de recargar la página
```

---

### US-19 — Ver ingresos por período
**Como** administrador, **quiero** filtrar los ingresos por período, **para** evaluar el rendimiento del negocio en un rango de fechas determinado.

**Criterios de aceptación:**

**Escenario 1: Filtrar por período**
```gherkin
Dado que el administrador selecciona un período (hoy, esta semana, este mes o rango personalizado)
Cuando aplica el filtro
Entonces el sistema muestra: total cobrado, total pendiente y desglose por tipo de servicio
Y para el lavado al seco muestra también el costo al tercero y la ganancia neta
```

---

### US-20 — Ver órdenes listas para entregar
**Como** operador o administrador, **quiero** ver de forma destacada las órdenes listas para ser recogidas, **para** atender al cliente cuando se acerque al local.

**Criterios de aceptación:**

**Escenario 1: Ver órdenes listas**
```gherkin
Dado que el usuario accede al dashboard
Cuando hay órdenes en estado "Listo"
Entonces el sistema las muestra resaltadas y separadas del resto
Y cada orden indica si tiene saldo pendiente de pago
```

**Escenario 2: Buscar orden de un cliente**
```gherkin
Dado que un cliente se acerca al local a recoger su ropa
Cuando el operador busca por nombre del cliente o número de orden
Entonces el sistema muestra la orden correspondiente con su estado y saldo pendiente
```

---

## Módulo 7: Gestión de Usuarios

---

### US-21 — Crear usuario operador
**Como** administrador, **quiero** crear una cuenta para un empleado con rol operador, **para** que pueda usar el sistema con acceso limitado.

**Criterios de aceptación:**

**Escenario 1: Crear operador exitosamente**
```gherkin
Dado que el administrador accede a la gestión de usuarios
Cuando ingresa nombre, correo y contraseña temporal del nuevo operador y guarda
Entonces el sistema crea la cuenta con rol operador
Y el nuevo usuario puede iniciar sesión con acceso limitado
```

---

### US-22 — Desactivar usuario
**Como** administrador, **quiero** desactivar la cuenta de un empleado, **para** revocar su acceso cuando ya no trabaje en el local.

**Criterios de aceptación:**

**Escenario 1: Desactivar operador**
```gherkin
Dado que el administrador accede al perfil de un operador activo
Cuando selecciona la opción de desactivar cuenta y confirma la acción
Entonces el sistema desactiva la cuenta
Y el usuario no puede iniciar sesión desde ese momento
Y las órdenes y pagos registrados por ese usuario se conservan sin cambios
```

---

## Product Backlog — Priorización

| ID | Historia | Prioridad | Módulo |
|---|---|---|---|
| US-01 | Iniciar sesión | Alta | Autenticación |
| US-03 | Registrar cliente | Alta | Clientes |
| US-04 | Buscar cliente | Alta | Clientes |
| US-06 | Crear orden por prenda | Alta | Órdenes |
| US-07 | Crear orden por kilo | Alta | Órdenes |
| US-13 | Registrar adelanto al crear orden | Alta | Pagos |
| US-14 | Registrar pago de saldo pendiente | Alta | Pagos |
| US-11 | Cambiar estado de orden | Alta | Órdenes |
| US-18 | Ver resumen del día | Alta | Dashboard |
| US-20 | Ver órdenes listas para entregar | Alta | Dashboard |
| US-16 | Gestionar precios de prendas | Media | Catálogo |
| US-17 | Configurar precio por kilo | Media | Catálogo |
| US-10 | Ver detalle de orden | Media | Órdenes |
| US-08 | Seguimiento de lavado al seco | Media | Órdenes |
| US-09 | Adjuntar foto del ticket | Media | Órdenes |
| US-05 | Ver historial de cliente | Media | Clientes |
| US-12 | Listar órdenes con filtros | Media | Órdenes |
| US-15 | Ver historial de pagos | Media | Pagos |
| US-19 | Ver ingresos por período | Media | Dashboard |
| US-21 | Crear usuario operador | Baja | Usuarios |
| US-22 | Desactivar usuario | Baja | Usuarios |
| US-02 | Cerrar sesión | Baja | Autenticación |