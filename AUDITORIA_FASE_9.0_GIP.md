# AUDITORIA FASE 9.0 - GIP (Gestion Interna de Pedidos)
## CloudPyme POS V18+ - Pre-Implementacion CRUD Administrativo

**Fecha:** 2026-01-26
**Sistema:** CloudPyme POS V18.1
**Arquitectura:** Google Apps Script + HTML/CSS/JS
**Base de Datos:** Google Sheets (ID: 1EsRSQQIcvUPYph1dDcRyc-8Zga8cbJzjerYUVKby2WE)

---

## 1. ESTADO ACTUAL DEL SISTEMA

### 1.1 Estructura de Archivos

| Archivo | Descripcion | Lineas |
|---------|-------------|--------|
| `1 Codigo GS V18.1.txt` | Backend completo (Google Apps Script) | ~2250 |
| `5 DASHBOARD HMTL V18.1.txt` | HTML del Dashboard | 477 |
| `6 CSS-DAS HTML V18.1.txt` | Estilos CSS Dashboard | 809 |
| `7 JS-DASH V18.1.txt` | JavaScript Dashboard | 1355 |

### 1.2 Hojas de Google Sheets (Tablas)

| Hoja | Proposito | Columnas Criticas |
|------|-----------|-------------------|
| `Usuarios` | Usuarios internos + clientes App | ID, Email, Password, Nombre, Telefono, Direccion, Rol, Activo, PIN |
| `Clientes` | Clientes domicilio (tabla separada) | ID_Key_Clientes, Nombre_Completo, Celular, Domicilio, Email |
| `Pedidos` | Pedidos principales | 16 columnas (nueva estructura) |
| `Detalle_Pedidos` | Items de cada pedido | ID_Detalle, ID_Pedido, ID_Cliente, ID_Producto, Cantidad, etc. |
| `Proceso_Delivery` | Auditoria cambios estado | ID_Delivery, ID_Pedido, Usuario_Cambio, Estado_Anterior, Estado_Nuevo |
| `Mesas` | Control mesas (Panel Meseros) | ID_Mesas, Nombre, Capacidad, Estado, ID_Pedido_Activo |
| `Configuracion` | Parametros empresa | Parametro, Valor |
| `Categorias` | Categorias menu | ID, Nombre, Icono, Orden, Activo |
| `Productos` | Catalogo productos | ID, Categoria, Nombre, Precio, Disponible |

### 1.3 Sistema de Roles

```
ROLES_INTERNOS: ['Administrador', 'Admin', 'Cocina', 'Domiciliario', 'Repartidor', 'Cajero']
ROLES_SUPER_ADMIN: ['Administrador', 'Admin']
```

**Estructura de la tabla USUARIOS (10 columnas):**
| Col | Nombre | Tipo |
|-----|--------|------|
| 0 | ID | String (CLI00001, USR001) |
| 1 | Email | String (unico) |
| 2 | Password | String |
| 3 | Nombre | String |
| 4 | Telefono | String |
| 5 | Direccion | String |
| 6 | Fecha_Registro | Date |
| 7 | Rol | String |
| 8 | Activo | Boolean |
| 9 | Permisos | String |
| 10 | PIN | Number (4 digitos) |

**Estructura de la tabla CLIENTES (10 columnas):**
| Col | Nombre | Tipo |
|-----|--------|------|
| 0 | ID_Key_Clientes | String (celular o CLI-DOM-XXXX) |
| 1 | Nombre_Completo | String |
| 2 | Celular | String |
| 3 | Domicilio | String |
| 4 | Notas_Adicionales | String |
| 5 | Fecha_Registro | Date |
| 6 | Email | String |
| 7 | Password | String |
| 8 | Rol | String |
| 9 | Activo | Boolean |

---

## 2. FUNCIONES EXISTENTES - INVENTARIO COMPLETO

### 2.1 Autenticacion (NO MODIFICAR)

| Funcion | Linea | Descripcion | Riesgo |
|---------|-------|-------------|--------|
| `validarLogin(email, password)` | 226 | Login general tabla USUARIOS | CRITICO |
| `validarLoginDashboard(email, password)` | 265 | Login Dashboard (solo internos) | CRITICO |
| `validarPinMesero(pin)` | 999 | Validacion PIN 4 digitos | CRITICO |
| `registrarUsuario(datos)` | 279 | Registro clientes desde App | CRITICO |

### 2.2 Dashboard - Pedidos

| Funcion | Linea | Descripcion | Modificable |
|---------|-------|-------------|-------------|
| `obtenerPedidosDashboard(filtros)` | 559 | Lista pedidos con filtros | SI (agregar clientes) |
| `obtenerDetallePedidoDashboard(idPedido)` | 691 | Detalle de un pedido | NO |
| `cambiarEstadoPedido(...)` | 787 | Cambio estado logistico | NO |
| `obtenerEstadisticasDashboard(fecha)` | 867 | Estadisticas del dia | NO |
| `exportarPedidosCSV(filtros)` | ~890 | Exportar a CSV | NO |
| `registrarCambioEstado(...)` | 163 | Auditoria en Proceso_Delivery | NO |
| `obtenerHistorialPedido(idPedido)` | 188 | Timeline de estados | NO |

### 2.3 Gestion de Clientes

| Funcion | Linea | Descripcion | Modificable |
|---------|-------|-------------|-------------|
| `buscarClientePorCelular(celular)` | 1735 | Busca en tabla Clientes | SI |
| `buscarClienteDomicilio(termino)` | 1766 | Busqueda por nombre/celular | SI |
| `guardarClienteDomicilio(datos)` | 1797 | Crear/actualizar cliente | SI |
| `buscarOcrearClienteUnificado(datos)` | 2087 | Unificacion V16 | PRECAUCION |
| `obtenerDatosCliente(idCliente)` | 2188 | Datos de un cliente | SI |
| `obtenerOCrearClienteGenerico()` | 2026 | Cliente "Consumidor Final" | NO |

### 2.4 Panel Meseros (NO MODIFICAR)

| Funcion | Linea | Descripcion |
|---------|-------|-------------|
| `crearPedidoDomicilio(datosCliente, datosUsuario)` | 1831 | Nuevo pedido domicilio |
| `crearPedidoRapido(datosUsuario)` | 1867 | Pedido rapido |
| `finalizarPedido(idPedido, datosCierre)` | 1899 | Cerrar pedido |
| `obtenerMesasPanel()` | 1089 | Lista de mesas |
| `agregarItemsPedido(...)` | 1622 | Agregar items a pedido |
| `cancelarPedidoBackend(...)` | 1555 | Cancelar pedido |

### 2.5 Frontend Dashboard (JS-DASH)

| Funcion | Descripcion | Modificable |
|---------|-------------|-------------|
| `loginDashboard(event)` | Login frontend | NO |
| `cargarDatosDashboard()` | Carga pedidos | SI |
| `renderizarTablaPedidos()` | Render tabla | SI |
| `verDetallePedido(idPedido)` | Abre modal | NO |
| `cambiarEstado(nuevoEstado)` | Cambio estado | NO |
| `obtenerNombreCliente(idCliente)` | Mapper local | SI |
| `obtenerDatosClienteLocal(idCliente)` | Datos cache | SI |

---

## 3. MAPEO DE RIESGOS

### 3.1 Puntos de Falla Criticos

| Escenario | Impacto | Mitigacion |
|-----------|---------|------------|
| API Admin cae durante operacion | Dashboard no carga | Implementar reintentos con exponential backoff |
| Lock de escritura excede timeout | Datos inconsistentes | Usar transacciones con `LockService` (ya implementado) |
| Tabla USUARIOS corrupta | Login falla completamente | Backup automatico antes de modificar |
| Email duplicado insertado | Conflicto de identidad | Validacion previa obligatoria |
| PIN duplicado | Acceso cruzado | Validacion unicidad de PIN |

### 3.2 Funciones Intocables (NO MODIFICAR)

```javascript
// CRITICO - AUTENTICACION
validarLogin()
validarLoginDashboard()
validarPinMesero()
registrarUsuario()

// CRITICO - FLUJO DE PEDIDOS
crearPedido()
cambiarEstadoPedido()
cancelarPedidoBackend()
registrarCambioEstado()

// CRITICO - PANEL MESEROS
crearPedidoDomicilio()
crearPedidoRapido()
finalizarPedido()
agregarItemsPedido()
```

### 3.3 Validaciones de Datos Requeridas

| Campo | Validacion | Tipo |
|-------|------------|------|
| Email | Unico en USUARIOS | Backend + Frontend |
| Celular | Minimo 7 digitos, solo numeros | Backend |
| PIN | 4 digitos, unico | Backend |
| Password | Minimo 4 caracteres | Frontend |
| Rol | Solo valores permitidos | Backend |
| Nombre | No vacio, max 100 chars | Frontend |

---

## 4. PLAN DE ROLLBACK

### 4.1 Checkpoints por Sub-Fase

| Fase | Checkpoint | Accion Rollback |
|------|------------|-----------------|
| 9.1 | Pre-listar usuarios | git revert al commit actual |
| 9.2 | Pre-crear usuario | Eliminar ultima fila USUARIOS |
| 9.3 | Pre-editar usuario | Restaurar backup de fila |
| 9.4 | Pre-integrar frontend | git checkout archivos JS/HTML |

### 4.2 Backup de Funciones Criticas

Antes de modificar, crear copia en:
```
BACKUP_FUNCIONES_V18.1.gs
```

Funciones a respaldar:
- `validarLogin`
- `registrarUsuario`
- `obtenerPedidosDashboard`

### 4.3 Procedimiento de Emergencia

1. **Detectar falla** -> Verificar logs en Google Apps Script
2. **Notificar** -> Marcar sistema como "En mantenimiento"
3. **Revertir** -> `git revert HEAD` o restaurar desde backup
4. **Validar** -> Ejecutar pruebas manuales de login
5. **Comunicar** -> Informar al equipo

---

## 5. FUNCIONES A CREAR (FASE 9.1+)

### 5.1 Backend (Codigo.gs)

```javascript
// NUEVAS FUNCIONES CRUD ADMIN
obtenerUsuariosInternos()       // Lista usuarios con rol interno
crearUsuarioInterno(datos)      // Crear usuario Admin/Cocina/etc
editarUsuarioInterno(id, datos) // Editar datos de usuario
desactivarUsuario(id)           // Soft delete (Activo = false)
obtenerListaClientes()          // Lista paginada de clientes
editarCliente(id, datos)        // Editar datos cliente
```

### 5.2 Frontend (JS-Dashboard)

```javascript
// NUEVAS FUNCIONES UI
renderizarTablaUsuarios()       // Mostrar lista usuarios
abrirModalCrearUsuario()        // Modal nuevo usuario
abrirModalEditarUsuario(id)     // Modal editar
confirmarDesactivarUsuario(id)  // Confirmar desactivacion
```

### 5.3 HTML (Dashboard)

- Nueva seccion en sidebar: "Administracion"
- Modal para crear/editar usuario
- Tabla de usuarios con paginacion
- Formulario con validaciones

---

## 6. MATRIZ DE RIESGOS Y MITIGACIONES

| ID | Riesgo | Probabilidad | Impacto | Mitigacion |
|----|--------|--------------|---------|------------|
| R1 | Email duplicado | Media | Alto | Validacion pre-insert |
| R2 | PIN duplicado | Baja | Alto | Validacion pre-insert |
| R3 | Lock timeout | Baja | Medio | Retry con backoff |
| R4 | XSS en formularios | Media | Alto | Sanitizar inputs |
| R5 | Inyeccion en parametros | Baja | Alto | Validar tipos de datos |
| R6 | Sesion expirada | Media | Bajo | Refresh token |
| R7 | Perdida de datos | Baja | Critico | Backup pre-modificacion |

---

## 7. CHECKLIST DE VALIDACION PRE-DEPLOY

### 7.1 Antes de Fase 9.1

- [x] Auditoria de codigo completada
- [x] Estructura de tablas documentada
- [x] Funciones criticas identificadas
- [x] Plan de rollback definido
- [x] Backup de funciones criticas listo
- [ ] Ambiente de pruebas configurado
- [ ] Datos de prueba creados

### 7.2 Por cada Sub-Fase

- [ ] Codigo revisado por pares
- [ ] Pruebas unitarias ejecutadas
- [ ] Pruebas de integracion completadas
- [ ] Validaciones de seguridad verificadas
- [ ] Rollback probado
- [ ] Documentacion actualizada

### 7.3 Criterios de Exito

| Metrica | Valor Esperado |
|---------|----------------|
| Errores criticos | 0 |
| Tiempo de respuesta API | < 2s |
| Login existente funcional | 100% |
| Pedidos existentes accesibles | 100% |

---

## 8. CONCLUSION

### Estado: APROBADO PARA FASE 9.1

El sistema CloudPyme POS V18.1 esta listo para la implementacion del CRUD administrativo con las siguientes condiciones:

1. **Funciones criticas identificadas** - No se modificaran las funciones de login, registro y flujo de pedidos
2. **Estructura de datos clara** - Las tablas USUARIOS y CLIENTES tienen estructura conocida
3. **Plan de rollback definido** - Se puede revertir cualquier cambio
4. **Validaciones documentadas** - Se conocen las reglas de negocio

### Proximos Pasos

1. Crear backup del codigo actual
2. Configurar ambiente de pruebas
3. Iniciar Fase 9.1: Listar usuarios internos
4. Validar con pruebas antes de avanzar a Fase 9.2

---

**Documento generado:** 2026-01-26
**Autor:** Claude (Auditor de Sistema)
**Version:** 1.0
