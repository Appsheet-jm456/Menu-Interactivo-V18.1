# FASE 9.1 - API CRUD ADMINISTRATIVA
## CloudPyme POS V18+ - Servicios Backend

**Fecha:** 2026-01-26
**Archivo modificado:** `1 Codigo GS V18.1.txt`
**Lineas agregadas:** 2244-3038 (~800 lineas)

---

## 1. FUNCIONES IMPLEMENTADAS

### 1.1 adminGestionarUsuarios(accion, datos, usuarioSolicitante)

**Ubicacion:** Linea 2269

**Acciones soportadas:**

| Accion | Descripcion | Retorno |
|--------|-------------|---------|
| `'leer'` | Lista usuarios internos con passwords ocultos | `{success, data: [], total}` |
| `'guardar'` | Crear o actualizar usuario | `{success, data: {id, nombre, rol, accion}}` |
| `'eliminar'` | Soft delete (Activo=FALSE) | `{success, data: {id, nombre, accion}}` |

**Parametros de entrada para 'guardar':**
```javascript
{
  nombre: "Juan Perez",           // Obligatorio, min 2 chars
  email: "juan@empresa.com",      // Obligatorio excepto Mesero
  telefono: "3001234567",         // Opcional, min 7 digitos
  rol: "Mesero",                  // Obligatorio: Administrador|Mesero|Cocina|Domiciliario
  credencial: "1234",             // PIN (4 digitos) o Password (min 4 chars)
  activo: true,                   // Opcional, default true
  direccion: "Calle 123",         // Opcional
  idActual: "USR_123456789"       // Solo para actualizar
}
```

### 1.2 adminGestionarClientes(accion, datos, usuarioSolicitante)

**Ubicacion:** Linea 2713

**Acciones soportadas:**

| Accion | Descripcion | Retorno |
|--------|-------------|---------|
| `'leer'` | Lista clientes con paginacion | `{success, data: [], total, pagina, totalPaginas}` |
| `'guardar'` | Crear o actualizar cliente | `{success, data: {id, nombre, accion}}` |
| `'eliminar'` | Soft delete | `{success, data: {id, nombre, accion}}` |

**Parametros de entrada para 'guardar':**
```javascript
{
  nombre: "Maria Garcia",         // Obligatorio, min 2 chars
  celular: "3009876543",          // Obligatorio, min 7 digitos, sera el ID
  domicilio: "Carrera 45 #12-34", // Opcional
  email: "maria@gmail.com",       // Opcional
  notas: "Cliente VIP",           // Opcional
  id: "3009876543"                // Solo para actualizar
}
```

### 1.3 adminReactivarEntidad(tipo, id, usuarioSolicitante)

**Ubicacion:** Linea 2981

**Funcion auxiliar para reactivar usuarios/clientes desactivados:**
```javascript
// Reactivar usuario
adminReactivarEntidad('usuario', 'USR_123456789', usuarioAdmin);

// Reactivar cliente
adminReactivarEntidad('cliente', '3001234567', usuarioAdmin);
```

---

## 2. VALIDACIONES IMPLEMENTADAS

### 2.1 Validaciones de Usuario

| Campo | Validacion | Mensaje Error |
|-------|------------|---------------|
| Nombre | Obligatorio, min 2 chars | "El nombre es obligatorio (minimo 2 caracteres)" |
| Rol | Debe estar en lista permitida | "Rol no valido. Permitidos: ..." |
| Email | Formato valido (excepto Mesero) | "Email valido es obligatorio para rol X" |
| Email | Unico en tabla | "El email X ya esta registrado" |
| Telefono | Min 7 digitos si se proporciona | "Telefono debe tener minimo 7 digitos" |
| Telefono | Unico en tabla | "El telefono X ya esta registrado" |
| Credencial | Obligatoria para nuevos usuarios | "Credencial obligatoria para nuevo usuario" |
| PIN (Mesero) | Exactamente 4 digitos numericos | "El PIN para Mesero debe ser de 4 digitos numericos" |
| PIN | Unico entre usuarios activos | "El PIN X ya esta asignado a otro usuario" |
| Password | Min 4 caracteres | "La contraseña debe tener minimo 4 caracteres" |
| Auto-eliminacion | No permitida | "No puede desactivar su propia cuenta" |

### 2.2 Validaciones de Cliente

| Campo | Validacion | Mensaje Error |
|-------|------------|---------------|
| Nombre | Obligatorio, min 2 chars | "Nombre es obligatorio (minimo 2 caracteres)" |
| Celular | Obligatorio, min 7 digitos | "Celular es obligatorio (minimo 7 digitos)" |
| Celular | Unico en tabla | "El celular X ya esta registrado" |

### 2.3 Validaciones de Seguridad

- **Permisos:** Solo usuarios con rol `Administrador` o `Admin` pueden ejecutar
- **Passwords ocultos:** Retorna `'******'` en lugar del valor real
- **PINs ocultos:** Retorna `'****'` en lugar del valor real
- **LockService:** Todas las operaciones usan lock para evitar concurrencia
- **Sanitizacion:** Emails convertidos a lowercase, telefonos solo digitos

---

## 3. LOGICA DE CREDENCIALES POR ROL

| Rol | Password (col 2) | Pin_Acceso (col 10) | Login |
|-----|------------------|---------------------|-------|
| Mesero | (vacio) | PIN 4 digitos | Solo PIN |
| Cocina | Password | (vacio) | Email + Password |
| Domiciliario | Password | (vacio) | Email + Password |
| Administrador | Password | PIN (si es 4 digitos) | Ambos metodos |

---

## 4. CASOS DE PRUEBA

### 4.1 Crear Usuario Mesero con PIN

```javascript
// Entrada
adminGestionarUsuarios('guardar', {
  nombre: 'Carlos Mesero',
  telefono: '3101234567',
  rol: 'Mesero',
  credencial: '1234'
}, usuarioAdmin);

// Resultado esperado
{
  success: true,
  data: { id: 'USR_1706XXXXX', nombre: 'Carlos Mesero', rol: 'Mesero', accion: 'creado' },
  message: 'Usuario "Carlos Mesero" creado correctamente'
}
```

### 4.2 Crear Usuario Admin con Password

```javascript
// Entrada
adminGestionarUsuarios('guardar', {
  nombre: 'Ana Administradora',
  email: 'ana@empresa.com',
  telefono: '3201234567',
  rol: 'Administrador',
  credencial: 'MiPassword123'
}, usuarioAdmin);

// Resultado esperado
{
  success: true,
  data: { id: 'USR_1706XXXXX', nombre: 'Ana Administradora', rol: 'Administrador', accion: 'creado' },
  message: 'Usuario "Ana Administradora" creado correctamente'
}
```

### 4.3 Intentar Duplicar Email

```javascript
// Entrada (email ya existe)
adminGestionarUsuarios('guardar', {
  nombre: 'Pedro Prueba',
  email: 'ana@empresa.com', // Ya existe
  rol: 'Cocina',
  credencial: 'password123'
}, usuarioAdmin);

// Resultado esperado
{
  success: false,
  error: 'El email "ana@empresa.com" ya esta registrado'
}
```

### 4.4 Editar Cliente Existente

```javascript
// Entrada
adminGestionarClientes('guardar', {
  id: '3001234567', // ID existente
  nombre: 'Maria Garcia Actualizada',
  celular: '3001234567',
  domicilio: 'Nueva Direccion #123'
}, usuarioAdmin);

// Resultado esperado
{
  success: true,
  data: { id: '3001234567', nombre: 'Maria Garcia Actualizada', accion: 'actualizado' },
  message: 'Cliente "Maria Garcia Actualizada" actualizado correctamente'
}
```

### 4.5 Eliminar Usuario (Soft Delete)

```javascript
// Entrada
adminGestionarUsuarios('eliminar', {
  id: 'USR_1706XXXXX'
}, usuarioAdmin);

// Resultado esperado
{
  success: true,
  data: { id: 'USR_1706XXXXX', nombre: 'Carlos Mesero', accion: 'desactivado' },
  message: 'Usuario "Carlos Mesero" desactivado correctamente'
}

// En la hoja:
// Columna Activo = FALSE
// Columna Fecha_Baja = fecha actual
// Columna Usuario_Que_Elimino = ID y nombre del admin
```

### 4.6 Validacion PIN Duplicado

```javascript
// Entrada (PIN ya asignado a otro usuario)
adminGestionarUsuarios('guardar', {
  nombre: 'Nuevo Mesero',
  rol: 'Mesero',
  credencial: '1234' // Ya existe
}, usuarioAdmin);

// Resultado esperado
{
  success: false,
  error: 'El PIN "1234" ya esta asignado a otro usuario'
}
```

---

## 5. ESTRUCTURA DE COLUMNAS ACTUALIZADA

### 5.1 Tabla USUARIOS (13 columnas)

| Col | Nombre | Tipo | Notas |
|-----|--------|------|-------|
| 0 | ID | String | USR_timestamp o CLI00001 |
| 1 | Email | String | Unico (excepto Mesero) |
| 2 | Password | String | Para Cocina/Domiciliario/Admin |
| 3 | Nombre | String | Obligatorio |
| 4 | Telefono | String | Opcional, unico si existe |
| 5 | Direccion | String | Opcional |
| 6 | Fecha_Registro | Date | Automatico |
| 7 | Rol | String | Ver lista permitida |
| 8 | Activo | Boolean | Soft delete |
| 9 | Permisos | String | Reservado |
| 10 | Pin_Acceso | String | Para Mesero/Admin |
| 11 | Fecha_Baja | Date | Cuando se desactiva |
| 12 | Usuario_Que_Elimino | String | Quien desactivo |

### 5.2 Tabla CLIENTES (10 columnas)

| Col | Nombre | Tipo | Notas |
|-----|--------|------|-------|
| 0 | ID_Key_Clientes | String | Igual al celular |
| 1 | Nombre_Completo | String | Obligatorio |
| 2 | Celular | String | Obligatorio, unico |
| 3 | Domicilio | String | Opcional |
| 4 | Notas_Adicionales | String | Opcional |
| 5 | Fecha_Registro | Date | Automatico |
| 6 | Email | String | Opcional |
| 7 | Password | String | Reservado |
| 8 | Rol | String | Default 'Cliente' |
| 9 | Activo | Boolean | Soft delete |

---

## 6. COMPATIBILIDAD VERIFICADA

| Funcion Existente | Estado | Notas |
|-------------------|--------|-------|
| `validarLogin()` | NO MODIFICADA | Sigue funcionando |
| `validarLoginDashboard()` | NO MODIFICADA | Sigue funcionando |
| `validarPinMesero()` | NO MODIFICADA | Sigue funcionando |
| `registrarUsuario()` | NO MODIFICADA | Sigue funcionando |
| `obtenerPedidosDashboard()` | NO MODIFICADA | Sigue funcionando |
| `cambiarEstadoPedido()` | NO MODIFICADA | Sigue funcionando |

---

## 7. PROXIMOS PASOS (FASE 9.2)

1. **Frontend:** Crear interfaz en Dashboard para gestionar usuarios
2. **Modal:** Formulario crear/editar usuario
3. **Tabla:** Lista de usuarios con acciones
4. **Integracion:** Llamadas a las funciones backend desde JS-Dashboard

---

**Documento generado:** 2026-01-26
**Autor:** Claude (Senior Backend Developer)
**Estado:** LISTO PARA PRUEBAS
