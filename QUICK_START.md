# ⚡ QUICK START - OPCIÓN B HIKVISION QR

## ¿Qué se cambió?

```
Archivo: src/controllers/accessController.js

ANTES:
  - 8 líneas repetidas (2 veces)
  - Sin validación
  - Si fallaba, no se sabía

AHORA:
  - 59 líneas en 1 función reutilizable
  - Validación en cada paso
  - Errores claros al usuario
```

---

## 🚀 Iniciar Ahora

```bash
npm install
npm start
```

**Deberías ver:**
```
Server running on port 3000
Connected to MongoDB
```

---

## ✅ Verificación Rápida (2 minutos)

### 1. Generar un Token de Prueba

```bash
curl -X POST http://localhost:3000/api/access/generate-manual \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiYWRtaW4iLCJpYXQiOjE2MjE2NDIwOTB9.X-x_x-x_x-x" \
  -H "Content-Type: application/json" \
  -d '{
    "visitorName": "Test QR User",
    "destination": "Bodega A",
    "hostName": "Admin"
  }'
```

### 2. Ver en Logs del Servidor

Deberías ver:
```
📤 Registrando QR en Hikvision para: Test QR User (Token: 87654321)
✅ Usuario registrado en Hikvision
✅ Tarjeta registrada en Hikvision
✅ QR generado y registrado en Hikvision para token: 87654321
```

### 3. Respuesta Esperada

```json
{
  "success": true,
  "token": "87654321",
  "qrCodeImage": "data:image/png;base64,iVBORw0KGgo...",
  "visitorName": "Test QR User",
  "destination": "Bodega A"
}
```

---

## 🎯 Prueba Completa (5 minutos)

### Opción A: Con Residente Real

1. Abrir: `http://localhost:3000/resident_generator.html`
2. Login: `admin` / `password`
3. Crear solicitud de visitante
4. Aprobar
5. Ver QR generado

### Opción B: Test Directo

```bash
# Endpoint de generación manual
POST http://localhost:3000/api/access/generate-manual

# Header
Authorization: Bearer <JWT_TOKEN>

# Body
{
  "visitorName": "Juan Pérez",
  "destination": "Bodega B",
  "hostName": "Admin",
  "accessType": "single"
}
```

---

## 📋 La Nueva Función en Acción

```javascript
// NUEVA FUNCIÓN (líneas 21-80 en accessController.js)
async function sendQRToHikvision(token, visitorName) {
    // 1. Registra usuario en Hikvision
    // 2. Registra tarjeta virtual (QR)
    // 3. Genera imagen QR
    // 4. Retorna todo empaquetado
}

// AHORA EN generateManual() (línea 110):
const hikResult = await sendQRToHikvision(token, visitorName);
if (!hikResult) {
    return res.status(500).json({ error: 'No se pudo registrar' });
}

// AHORA EN approveRequest() (línea 211):
const hikResult = await sendQRToHikvision(token, pending.visitorName);
if (!hikResult) {
    return res.status(500).json({ error: 'Fallo registrando' });
}
```

---

## 🔧 Configuración (Si Necesitas Cambiar)

```javascript
// Línea 7 de src/controllers/accessController.js
const DEVICE = { 
    ip: "192.168.1.68",           // IP del Hikvision
    user: "admin",                // Usuario Hikvision
    pass: "1Q2w3e4r5t."          // Contraseña Hikvision
};
```

---

## 🚨 Si Algo Falla

### Error: "No se pudo registrar en Hikvision"

```bash
# 1. Verifica conectividad
ping 192.168.1.68

# 2. Accede a Hikvision
http://192.168.1.68

# 3. Reinicia servidor
npm start
```

### Error: "Cannot find module"

```bash
# Reinstala dependencias
npm install

# Reinicia
npm start
```

### Error: MongoDB connection refused

```bash
# Asegúrate que MongoDB corre
mongod

# O con Docker:
docker run -d -p 27017:27017 mongo
```

---

## ✨ Ventajas Principales

1. **Automático**: El Hikvision abre sin conectarse a servidor
2. **Seguro**: Token expira automáticamente (10 horas)
3. **Robusto**: Manejo completo de errores
4. **Simple**: El visitante solo escanea QR
5. **Auditable**: Logs en Hikvision + MongoDB

---

## 📈 Flujo (30 segundos)

```
1. Residente aprueba solicitud
   └─ Clic "ACEPTAR"

2. Sistema genera token + registra en Hikvision
   └─ sendQRToHikvision() hace toda la magia

3. Se muestra QR
   └─ Imprime o envía al visitante

4. Visitante escanea en entrada
   └─ Lee código con scanner Hikvision

5. ✅ LISTO - Torniquete abre automáticamente
   └─ Sin intervención del servidor
```

---

## 🧪 Test Mínimo (30 segundos)

```bash
# 1. Generar token
curl -X POST http://localhost:3000/api/access/generate-manual \
  -H "Authorization: Bearer token_aqui" \
  -H "Content-Type: application/json" \
  -d '{"visitorName":"Test","destination":"Bodega"}'

# 2. Ver respuesta (debe incluir qrCodeImage)
# ✅ Si ves "qrCodeImage" → FUNCIONA

# 3. Verificar en Hikvision
# Ir a http://192.168.1.68 → Buscar usuario "Token"
# ✅ Si lo encuentras → REGISTRADO CORRECTAMENTE
```

---

## 🎯 Casos de Uso Comunes

### Caso 1: Visitante Temporal
```
Nombre: Juan Pérez
Duración: 10 horas
Acceso: Una entrada y salida
QR: Escanea → Abre
```

### Caso 2: Proveedor Frecuente
```
Nombre: Técnico XYZ
Duración: 10 horas
Acceso: Múltiples entradas (hasta 10 veces)
QR: Escanea → Abre (cada vez)
```

### Caso 3: Pase Sin Solicitud
```
Residente genera directamente (sin visitante pidiendo)
Nombre: Entregador
Duración: 10 horas
QR: Listo para imprimir/enviar
```

---

## 📞 Soporte Rápido

```
Problema              | Solución
──────────────────────┼──────────────────────
No abre torniquete    | Verifica Hikvision UI
Error en logs         | Reinicia: npm start
QR no genera          | Instala: npm install
Hikvision no responde | Ping: 192.168.1.68
```

---

## ✅ Checklist de Verificación

- [ ] Código actualizado (`src/controllers/accessController.js`)
- [ ] Servidor corre: `npm start` (sin errores)
- [ ] Puede generar token (curl test)
- [ ] Logs muestran "✅ Usuario registrado"
- [ ] Logs muestran "✅ Tarjeta registrada"
- [ ] QR genera correctamente (data:image/png...)
- [ ] Hikvision recibe usuario nuevo
- [ ] Escanear abre torniquete

---

## 🎉 Listo!

Ahora tienes OPCIÓN B implementada. 

**Próximos pasos:**
1. Lee `TESTING_GUIDE.md` para pruebas completas
2. Revisa `DIAGRAMA_VISUAL.md` para entender la arquitectura
3. Implementa `CAMBIOS_RESUMEN.md` si necesitas más detalles

---

**Versión:** 2.0  
**Fecha:** 21 de Mayo, 2026  
**Estado:** ✅ LISTO PARA USAR
