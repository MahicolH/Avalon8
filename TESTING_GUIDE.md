# 🧪 GUÍA TESTING - OPCIÓN B HIKVISION QR

## ✅ Verificación Rápida

### Test 1: Sintaxis del Código
```bash
cd "c:\Users\F Hurtado jimenez\Downloads\park-acces-control (4) (1)\park-acces-control (3)\park-acces-control\park-acces-control"

node -c src/controllers/accessController.js
```

**Resultado Esperado:**
- ✅ Sin errores (no output = éxito)
- ✅ Si hay error: muestra línea específica

---

## 🚀 Test 2: Iniciar el Servidor

```bash
npm start
```

**Logs Esperados:**
```
Server running on port 3000
Connected to MongoDB
```

**Verificar Hikvision accesible:**
```bash
curl -v http://192.168.1.68:8000
```

---

## 🧑‍💼 Test 3: Flujo Completo (Manual)

### Paso A: Crear Token Manual

```bash
# Asumir JWT_TOKEN válido
JWT_TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."

curl -X POST http://localhost:3000/api/access/generate-manual \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "visitorName": "Test User QR",
    "destination": "Bodega A",
    "hostName": "Admin",
    "accessType": "single",
    "visitType": "cargo",
    "phone": "3105551234"
  }'
```

**Logs Esperados en Servidor:**
```
📤 Registrando QR en Hikvision para: Test User QR (Token: 87654321)
✅ Usuario registrado en Hikvision
✅ Tarjeta registrada en Hikvision
✅ QR generado y registrado en Hikvision para token: 87654321
```

**Response Esperado:**
```json
{
  "success": true,
  "token": "87654321",
  "qrCodeImage": "data:image/png;base64,iVBORw0KGgo...",
  "visitorName": "Test User QR",
  "destination": "Bodega A",
  "accessType": "single"
}
```

### Verificación en Hikvision

1. Acceder a interfaz Hikvision: `http://192.168.1.68`
2. Ir a: **User Management** → **Users**
3. Buscar usuario: "87654321"
4. Verificar datos:
   - ✅ employeeNo: 87654321
   - ✅ name: Test User QR
   - ✅ doorRight: 1
   - ✅ Valid: Habilitado

5. Ir a: **Card Management** → **Cards**
6. Buscar tarjeta: "87654321"
7. Verificar:
   - ✅ cardNo: 87654321
   - ✅ isEnable: true

---

## 📱 Test 4: Flujo Visitante → Residente (Real)

### Paso 1: Visitante solicita acceso

1. Abrir: `http://localhost:3000/visitor_request.html`
2. Llenar formulario:
   ```
   Tipo de visita: Carga
   Tipo de acceso: Temporal (10 hrs)
   Empresa: Acme Corp
   Nombre: Carlos García
   Teléfono: 3101234567
   Anfitrión: Admin
   Destino: Bodega B
   Placa: ABC1234
   ```
3. Clic: **SOLICITAR ACCESO**

**Resultado:**
- ✅ Se guarda en MongoDB
- ✅ Residente recibe notificación WebSocket
- ✅ Visitante ve mensaje: "Solicitud enviada"

### Paso 2: Residente aprueba

1. Abrir: `http://localhost:3000/resident_generator.html`
2. Login: `admin` / `password`
3. Ver sección **SOLICITUDES PENDIENTES**
4. Clic en **ACEPTAR** en la solicitud

**Logs Esperados:**
```
📤 Registrando QR en Hikvision para: Carlos García (Token: 34567890)
✅ Usuario registrado en Hikvision
✅ Tarjeta registrada en Hikvision
✅ QR generado y registrado en Hikvision para token: 34567890
```

**Resultado en Pantalla:**
- ✅ Se muestra QR código
- ✅ Se muestra token: 34567890
- ✅ Mensaje: "Acceso Aprobado"

### Paso 3: Visitante escanea en Hikvision

1. Ir a entrada con QR
2. Colocar QR frente a scanner Hikvision
3. Hikvision lee código: "34567890"

**Resultado Esperado:**
- ✅ Hikvision busca CardInfo con cardNo="34567890"
- ✅ Encuentra tarjeta
- ✅ **TORNIQUETE ABRE AUTOMÁTICAMENTE**
- ✅ Sin intervención del servidor

---

## 📊 Test 5: Verificación en MongoDB

```bash
# Conectar a MongoDB
mongo mongodb://localhost:27017/AccessControlDB

# Ver colecciones
db.getCollectionNames()

# Buscar tokens
db.accesstokens.find().pretty()

# Resultado esperado:
{
  "_id": ObjectId(...),
  "token": "87654321",
  "visitorName": "Test User QR",
  "status": "approved",
  "accessType": "single",
  "createdAt": ISODate("2026-05-21T12:20:55Z"),
  ...
}
```

---

## 🔍 Test 6: Buscar Errores Comunes

### Error: "No se pudo registrar en Hikvision"

**Posibles Causas:**
1. Hikvision no responde
2. IP incorrecta (192.168.1.68)
3. Credenciales inválidas
4. Firewall bloqueando puerto

**Debugging:**
```bash
# Verificar conectividad
ping 192.168.1.68

# Acceder directo a Hikvision
curl -v http://192.168.1.68:8000

# Ver logs del servidor
# Deberías ver línea:
# ❌ Error registrando usuario en Hikvision
```

### Error: "QR no abre torniquete"

**Posibles Causas:**
1. Tarjeta no se guardó en Hikvision
2. doorRight incorrecto
3. planTemplateNo no existe
4. Torniquete no configurado

**Debugging:**
```bash
# Verificar en Hikvision que la tarjeta existe
http://192.168.1.68 → CardInfo → Buscar token

# Verificar permisos de la tarjeta
http://192.168.1.68 → User Management → Buscar usuario

# Probar apertura manual desde Hikvision
http://192.168.1.68 → Control de acceso → Abrir puerta
```

---

## 📈 Test 7: Prueba de Carga

### Generar 10 tokens en paralelo

```bash
for i in {1..10}; do
  curl -X POST http://localhost:3000/api/access/generate-manual \
    -H "Authorization: Bearer $JWT_TOKEN" \
    -H "Content-Type: application/json" \
    -d "{
      \"visitorName\": \"User $i\",
      \"destination\": \"Bodega A\",
      \"hostName\": \"Admin\"
    }" &
done
wait

echo "10 tokens generados"
```

**Verificar:**
- ✅ Todos los tokens se crean
- ✅ Todos se registran en Hikvision
- ✅ No hay duplicados
- ✅ No hay errores

---

## 🔐 Test 8: Seguridad

### Test A: JWT inválido

```bash
curl -X POST http://localhost:3000/api/access/generate-manual \
  -H "Authorization: Bearer INVALID_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "visitorName": "Test",
    "destination": "Bodega"
  }'
```

**Resultado Esperado:**
```json
{
  "error": "Unauthorized"
}
```

### Test B: Sin JWT

```bash
curl -X POST http://localhost:3000/api/access/generate-manual \
  -H "Content-Type: application/json" \
  -d '{
    "visitorName": "Test",
    "destination": "Bodega"
  }'
```

**Resultado Esperado:**
```json
{
  "error": "No token provided"
}
```

### Test C: Expiración

1. Crear token
2. Esperar 10 horas
3. Intentar escanear QR

**Resultado Esperado:**
- ✅ Hikvision rechaza (fuera de Valid period)
- ✅ Torniquete no abre

---

## ✨ Test 9: Casos Extremos

### Caso 1: Nombre muy largo

```bash
curl -X POST http://localhost:3000/api/access/generate-manual \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "visitorName": "ABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZ",
    "destination": "Bodega"
  }'
```

**Resultado Esperado:**
- ✅ Se trunca a 31 caracteres
- ✅ Sin errores

### Caso 2: Caracteres especiales

```bash
curl -X POST http://localhost:3000/api/access/generate-manual \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "visitorName": "José María O'\''Brien",
    "destination": "Bodega"
  }'
```

**Resultado Esperado:**
- ✅ Se guarda sin problemas
- ✅ Se envía a Hikvision escapado correctamente

### Caso 3: Múltiples aprobaciones

1. Crear 3 solicitudes del mismo visitante
2. Aprobar las 3

**Resultado Esperado:**
- ✅ 3 tokens diferentes
- ✅ 3 QR diferentes
- ✅ 3 usuarios en Hikvision
- ✅ Cada uno con su propio cardNo

---

## 📋 Checklist Final

```
ANTES DE DESPLEGAR A PRODUCCIÓN:

Funcionalidad:
☐ Test 1: Sintaxis correcta
☐ Test 2: Servidor inicia
☐ Test 3: Token se crea
☐ Test 4: Token se registra en Hikvision
☐ Test 5: QR se genera
☐ Test 6: Visitante puede escanear
☐ Test 7: Torniquete abre

Errores:
☐ Validación de JWT funciona
☐ Campos requeridos validan
☐ Errores de Hikvision se manejan
☐ MongoDB está accesible

Performance:
☐ Respuesta en <500ms
☐ No hay memory leaks
☐ Puede manejar 10 requests paralelos

Seguridad:
☐ JWT válido requerido
☐ Tokens no son predecibles
☐ Expiración automática funciona
☐ Logs de auditoría se guardan

Documentación:
☐ Código comentado
☐ README actualizado
☐ API docs completa
☐ Deployment guide listo
```

---

## 🆘 Soporte Rápido

Si algo falla:

```
1. Revisa LOGS en consola del servidor
2. Busca líneas con: ❌ Error
3. Identifica en qué paso falló
4. Verifica configuración:
   - IP Hikvision: 192.168.1.68
   - Usuario: admin
   - Contraseña: 1Q2w3e4r5t.
5. Reinicia servidor:
   npm start
```

---

**Testing Guide v2.0**  
**Última actualización:** 21 de Mayo, 2026  
**Status:** Listo para Producción ✅
