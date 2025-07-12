# Guía : Implementación de Telefonía IP Hospital "Salud Integral"

## 📋 Información General del Proyecto

### Objetivo

Migrar el sistema de telefonía analógica tradicional a una solución de Telefonía sobre IP (VoIP) para el Hospital General "Salud Integral".

### Equipos Necesarios

- 1 Router Cisco 2911 (con capacidad CME)
- 2 Switches Cisco 2960
- 8 Teléfonos IP Cisco 7960
- 4 PCs de escritorio

### Esquema de Direccionamiento

- **Red General**: 192.168.80.0/24
- **VLAN de Datos (VLAN 10)**: 192.168.80.0/25 (DATOS_HOSPITAL)
- **VLAN de Voz (VLAN 20)**: 192.168.80.128/25 (VOZ_HOSPITAL)

### Plan de Numeración Telefónica

- **Recepción y Admisiones**: 101, 102
- **Sala de Emergencias**: 201, 202
- **Consultorios Médicos**: 301, 302
- **Administración**: 401, 402

---

## 🔧 PASO 1: Diseño Físico y Cableado

### 1.1 Configuración del Entorno

1. **Abrir Cisco Packet Tracer**
2. **Seleccionar dispositivos**:
   - Ir a la pestaña "Network Devices"
   - Seleccionar "Routers" → Cisco 2911
   - Seleccionar "Switches" → Cisco 2960 (2 unidades)
   - Ir a "End Devices" → IP Phone 7960 (8 unidades)
   - Ir a "End Devices" → PC Desktop (4 unidades)

### 1.2 Colocación de Dispositivos

#### Distribución Optimizada por Switch (4 PCs + 8 Teléfonos)

**Switch1 (4 teléfonos + 2 PCs):**

- **Recepción**: PC1 + Tel1 (Ext: 101)
- **Admisiones**: PC2 + Tel2 (Ext: 102)
- **Emergencia**: Tel3 (Ext: 201) + Tel4 (Ext: 202)

**Switch2 (4 teléfonos + 2 PCs):**

- **Consultorios**: PC3 + Tel5 (Ext: 301) + Tel6 (Ext: 302)
- **Administración**: PC4 + Tel7 (Ext: 401) + Tel8 (Ext: 402)

### 1.3 Cableado Estructurado

#### Conexiones Router-Switches

1. **Router G0/0 → Switch1 G0/1**
   - Usar cable "Copper Straight-Through"
   - Click en Router → seleccionar G0/0
   - Click en Switch1 → seleccionar G0/1

2. **Router G0/1 → Switch2 G0/1**
   - Usar cable "Copper Straight-Through"
   - Click en Router → seleccionar G0/1
   - Click en Switch2 → seleccionar G0/1

#### Conexiones Teléfonos-Switches

1. **Switch1**: Conectar teléfonos 1-4 a puertos Fa0/1-Fa0/4
2. **Switch2**: Conectar teléfonos 5-8 a puertos Fa0/1-Fa0/4

#### Conexiones PCs-Teléfonos

1. **PC1** → Puerto "Switch" del **Tel1**
2. **PC2** → Puerto "Switch" del **Tel2**
3. **PC3** → Puerto "Switch" del **Tel5**
4. **PC4** → Puerto "Switch" del **Tel7**
5. Usar cables "Copper Straight-Through" para todas las conexiones

---

## 🔧 PASO 2: Configuración de los Switches

### 2.1 Configuración Switch1

1. **Acceder al CLI del Switch1**:
   - Click en Switch1 → pestaña "CLI"
   - Presionar Enter para acceder

2. **Entrar en modo configuración**:

```bash
Switch>enable
Switch#configure terminal
```

3. **Configurar hostname**:

```bash
Switch(config)#hostname Switch1-Hospital
```

4. **Crear las VLANs**:

```bash
Switch1-Hospital(config)#vlan 10
Switch1-Hospital(config-vlan)#name DATOS_HOSPITAL
Switch1-Hospital(config-vlan)#exit

Switch1-Hospital(config)#vlan 20
Switch1-Hospital(config-vlan)#name VOZ_HOSPITAL
Switch1-Hospital(config-vlan)#exit
```

5. **Configurar puertos de acceso (Fa0/1 - Fa0/4)**:

```bash
Switch1-Hospital(config)#interface range FastEthernet0/1-4
Switch1-Hospital(config-if-range)#switchport mode access
Switch1-Hospital(config-if-range)#switchport access vlan 10
Switch1-Hospital(config-if-range)#switchport voice vlan 20
Switch1-Hospital(config-if-range)#spanning-tree portfast
Switch1-Hospital(config-if-range)#exit
```

6. **Configurar puerto troncal (G0/1)**:

```bash
Switch1-Hospital(config)#interface GigabitEthernet0/1
Switch1-Hospital(config-if)#switchport mode trunk
Switch1-Hospital(config-if)#switchport trunk allowed vlan 10,20
Switch1-Hospital(config-if)#exit
```

7. **Guardar configuración**:

```bash
Switch1-Hospital(config)#exit
Switch1-Hospital#write memory
```

### 2.2 Configuración Switch2

1. **Acceder al CLI del Switch2**:
   - Click en Switch2 → pestaña "CLI"
   - Presionar Enter para acceder

2. **Entrar en modo configuración**:

```bash
Switch>enable
Switch#configure terminal
```

3. **Configurar hostname**:

```bash
Switch(config)#hostname Switch2-Hospital
```

4. **Crear las VLANs**:

```bash
Switch2-Hospital(config)#vlan 10
Switch2-Hospital(config-vlan)#name DATOS_HOSPITAL
Switch2-Hospital(config-vlan)#exit

Switch2-Hospital(config)#vlan 20
Switch2-Hospital(config-vlan)#name VOZ_HOSPITAL
Switch2-Hospital(config-vlan)#exit
```

5. **Configurar puertos de acceso (Fa0/1 - Fa0/4)**:

```bash
Switch2-Hospital(config)#interface range FastEthernet0/1-4
Switch2-Hospital(config-if-range)#switchport mode access
Switch2-Hospital(config-if-range)#switchport access vlan 10
Switch2-Hospital(config-if-range)#switchport voice vlan 20
Switch2-Hospital(config-if-range)#spanning-tree portfast
Switch2-Hospital(config-if-range)#exit
```

6. **Configurar puerto troncal (G0/1)**:

```bash
Switch2-Hospital(config)#interface GigabitEthernet0/1
Switch2-Hospital(config-if)#switchport mode trunk
Switch2-Hospital(config-if)#switchport trunk allowed vlan 10,20
Switch2-Hospital(config-if)#exit
```

7. **Guardar configuración**:

```bash
Switch2-Hospital(config)#exit
Switch2-Hospital#write memory
```

---

## 🔧 PASO 3: Configuración del Router (Interfaces y DHCP)

### 3.1 Configuración Básica del Router

1. **Acceder al CLI del Router**:
   - Click en Router → pestaña "CLI"
   - Presionar Enter

2. **Entrar en modo configuración**:

```bash
Router>enable
Router#configure terminal
```

3. **Configurar hostname**:

```bash
Router(config)#hostname HospitalRouter
```

### 3.2 Configuración de Subinterfaces (OPTIMIZADA)

#### Para la interfaz G0/0 (Switch1)

```bash
HospitalRouter(config)#interface GigabitEthernet0/0
HospitalRouter(config-if)#no shutdown
HospitalRouter(config-if)#exit

HospitalRouter(config)#interface GigabitEthernet0/0.10
HospitalRouter(config-subif)#description VLAN de Datos Switch1
HospitalRouter(config-subif)#encapsulation dot1Q 10
HospitalRouter(config-subif)#ip address 192.168.80.1 255.255.255.128
HospitalRouter(config-subif)#exit

HospitalRouter(config)#interface GigabitEthernet0/0.20
HospitalRouter(config-subif)#description VLAN de Voz Switch1
HospitalRouter(config-subif)#encapsulation dot1Q 20
HospitalRouter(config-subif)#ip address 192.168.80.129 255.255.255.128
HospitalRouter(config-subif)#exit
```

#### Para la interfaz G0/1 (Switch2) - MISMAS IPs, diferentes interfaces físicas

```bash
HospitalRouter(config)#interface GigabitEthernet0/1
HospitalRouter(config-if)#no shutdown
HospitalRouter(config-if)#exit

HospitalRouter(config)#interface GigabitEthernet0/1.10
HospitalRouter(config-subif)#description VLAN de Datos Switch2
HospitalRouter(config-subif)#encapsulation dot1Q 10
HospitalRouter(config-subif)#ip address 192.168.80.1 255.255.255.128
HospitalRouter(config-subif)#exit

HospitalRouter(config)#interface GigabitEthernet0/1.20
HospitalRouter(config-subif)#description VLAN de Voz Switch2
HospitalRouter(config-subif)#encapsulation dot1Q 20
HospitalRouter(config-subif)#ip address 192.168.80.129 255.255.255.128
HospitalRouter(config-subif)#exit
```

### 3.3 Configuración del Servidor DHCP (MEJORADA)

#### Pool de Datos (OPTIMIZADO)

```bash
HospitalRouter(config)#ip dhcp pool DATOS_HOSPITAL
HospitalRouter(dhcp-config)#network 192.168.80.0 255.255.255.128
HospitalRouter(dhcp-config)#default-router 192.168.80.1
HospitalRouter(dhcp-config)#dns-server 8.8.8.8
HospitalRouter(dhcp-config)#domain-name hospital.local
HospitalRouter(dhcp-config)#exit
```

#### Pool de Voz (OPTIMIZADO)

```bash
HospitalRouter(config)#ip dhcp pool VOZ_HOSPITAL
HospitalRouter(dhcp-config)#network 192.168.80.128 255.255.255.128
HospitalRouter(dhcp-config)#default-router 192.168.80.129
HospitalRouter(dhcp-config)#dns-server 8.8.8.8
HospitalRouter(dhcp-config)#domain-name hospital.local
HospitalRouter(dhcp-config)#option 150 ip 192.168.80.129
HospitalRouter(dhcp-config)#exit
```

#### Excluir IPs del Gateway

```bash
HospitalRouter(config)#ip dhcp excluded-address 192.168.80.1 192.168.80.3
HospitalRouter(config)#ip dhcp excluded-address 192.168.80.129 192.168.80.131
```

---

## 🔧 PASO 4: Configuración de Telefonía IP (CME)

### 4.1 Habilitar el servicio de telefonía

```bash
HospitalRouter(config)#telephony-service
HospitalRouter(config-telephony)#max-ephones 10
HospitalRouter(config-telephony)#max-dn 10
HospitalRouter(config-telephony)#ip source-address 192.168.80.129 port 2000
HospitalRouter(config-telephony)#auto assign 1 to 10
HospitalRouter(config-telephony)#exit
```

### 4.2 Configurar las Extensiones Telefónicas

```bash
# Recepción y Admisiones
HospitalRouter(config)#ephone-dn 1
HospitalRouter(config-ephone-dn)#number 101
HospitalRouter(config-ephone-dn)#name Recepcion-101
HospitalRouter(config-ephone-dn)#exit

HospitalRouter(config)#ephone-dn 2
HospitalRouter(config-ephone-dn)#number 102
HospitalRouter(config-ephone-dn)#name Admisiones-102
HospitalRouter(config-ephone-dn)#exit

# Sala de Emergencias
HospitalRouter(config)#ephone-dn 3
HospitalRouter(config-ephone-dn)#number 201
HospitalRouter(config-ephone-dn)#name Emergencia-201
HospitalRouter(config-ephone-dn)#exit

HospitalRouter(config)#ephone-dn 4
HospitalRouter(config-ephone-dn)#number 202
HospitalRouter(config-ephone-dn)#name Emergencia-202
HospitalRouter(config-ephone-dn)#exit

# Consultorios Médicos
HospitalRouter(config)#ephone-dn 5
HospitalRouter(config-ephone-dn)#number 301
HospitalRouter(config-ephone-dn)#name Consultorio-301
HospitalRouter(config-ephone-dn)#exit

HospitalRouter(config)#ephone-dn 6
HospitalRouter(config-ephone-dn)#number 302
HospitalRouter(config-ephone-dn)#name Consultorio-302
HospitalRouter(config-ephone-dn)#exit

# Administración
HospitalRouter(config)#ephone-dn 7
HospitalRouter(config-ephone-dn)#number 401
HospitalRouter(config-ephone-dn)#name Administracion-401
HospitalRouter(config-ephone-dn)#exit

HospitalRouter(config)#ephone-dn 8
HospitalRouter(config-ephone-dn)#number 402
HospitalRouter(config-ephone-dn)#name Administracion-402
HospitalRouter(config-ephone-dn)#exit
```

### 4.3 Configuración Manual de Teléfonos (si es necesario)

Si la asignación automática no funciona:

```bash
# Verificar MAC addresses de los teléfonos
HospitalRouter#show ephone

# Asignar manualmente (ejemplo para el primer teléfono)
HospitalRouter(config)#ephone 1
HospitalRouter(config-ephone)#mac-address [MAC_DEL_TELEFONO]
HospitalRouter(config-ephone)#type 7960
HospitalRouter(config-ephone)#button 1:1
HospitalRouter(config-ephone)#exit
```

---

## 🔧 PASO 5: Verificación y Pruebas

### 5.1 Verificar Asignación de IPs

1. **Verificar IPs en PCs**:
   - Click en cada PC → pestaña "Desktop" → "Command Prompt"
   - Ejecutar: `ipconfig`
   - Verificar que las IPs estén en el rango 192.168.80.4-126

2. **Verificar IPs en Teléfonos**:
   - Click en cada teléfono → verificar que muestre IP en pantalla
   - Las IPs deben estar en el rango 192.168.80.132-254

### 5.2 Comandos de Verificación Extendidos (OPTIMIZADO)

En el router:

```bash
# Verificar estado de interfaces
HospitalRouter#show ip interface brief

# Verificar enrutamiento entre VLANs
HospitalRouter#show ip route

# Verificar configuración DHCP
HospitalRouter#show ip dhcp binding
HospitalRouter#show ip dhcp pool

# Verificar telefonía
HospitalRouter#show telephony-service
HospitalRouter#show ephone registered
HospitalRouter#show ephone summary
HospitalRouter#show ephone-dn
```

En los switches:

```bash
# Verificar VLANs
Switch1-Hospital#show vlan brief
Switch2-Hospital#show vlan brief

# Verificar puertos troncales
Switch1-Hospital#show interface trunk
Switch2-Hospital#show interface trunk

# Verificar estado de puertos
Switch1-Hospital#show interface status
Switch2-Hospital#show interface status
```

### 5.3 Realizar Pruebas de Llamadas

1. **Preparar para pruebas**:
   - Cambiar a modo "Realtime" en Packet Tracer
   - Click en un teléfono IP

2. **Realizar llamadas**:
   - Levantar el auricular (click en el teléfono)
   - Marcar extensión de destino (ejemplo: 201)
   - Verificar conexión exitosa

3. **Pruebas sugeridas según departamentos**:
   - 101 → 201 (Recepción a Emergencia)
   - 301 → 401 (Consultorio a Administración)
   - 102 → 302 (Admisiones a Consultorio)
   - 202 → 402 (Emergencia a Administración)
   - 101 → 301 (Recepción a Consultorio)
   - 201 → 401 (Emergencia a Administración)

4. **Pruebas de conectividad de datos**:
   - Desde PC1: `ping 192.168.80.130` (gateway de voz)
   - Desde PC2: `ping 192.168.80.1` (gateway de datos)
   - Verificar conectividad entre PCs en diferentes switches

---

## 📊 Comandos de Verificación Importantes

### Verificar configuración de VLANs

```bash
Switch1-Hospital#show vlan brief
Switch2-Hospital#show vlan brief
```

### Verificar estado de interfaces

```bash
HospitalRouter#show ip interface brief
HospitalRouter#show ip route
```

### Verificar pools DHCP

```bash
HospitalRouter#show ip dhcp binding
HospitalRouter#show ip dhcp pool
HospitalRouter#show ip dhcp conflict
```

### Verificar telefonía

```bash
HospitalRouter#show telephony-service
HospitalRouter#show ephone registered
HospitalRouter#show ephone-dn
HospitalRouter#show ephone summary
```

### Verificar conectividad

```bash
# Desde router
HospitalRouter#ping 192.168.80.4
HospitalRouter#ping 192.168.80.132

# Desde PCs
PC>ping 192.168.80.1
PC>ping 192.168.80.129
PC>tracert 192.168.80.130
```

---

## 🚨 Solución de Problemas Comunes

### Problema 1: Teléfonos no se registran

**Síntomas**: Los teléfonos no aparecen en `show ephone registered`

**Solución**:

1. Verificar que option 150 esté configurada correctamente
2. Verificar conectividad entre VLANs: `ping 192.168.80.129`
3. Reiniciar el servicio telephony:

   ```bash
   HospitalRouter(config)#no telephony-service
   HospitalRouter(config)#telephony-service
   ```

4. Verificar que los teléfonos obtengan IP de la VLAN 20

### Problema 2: No hay comunicación entre VLANs

**Síntomas**: Los PCs no pueden hacer ping al gateway de voz

**Solución**:

1. Verificar configuración de subinterfaces:

   ```bash
   HospitalRouter#show ip interface brief
   ```

2. Verificar que los switches tengan configurado el trunking:

   ```bash
   Switch1-Hospital#show interface trunk
   ```

3. Verificar que las VLANs estén creadas en ambos switches:

   ```bash
   Switch1-Hospital#show vlan brief
   ```

### Problema 3: DHCP no asigna IPs

**Síntomas**: PCs o teléfonos no obtienen dirección IP

**Solución**:

1. Verificar que las interfaces estén "up":

   ```bash
   HospitalRouter#show ip interface brief
   ```

2. Verificar que no haya conflictos en los rangos de IP:

   ```bash
   HospitalRouter#show ip dhcp conflict
   ```

3. Verificar que el pool DHCP esté configurado correctamente:

   ```bash
   HospitalRouter#show ip dhcp pool
   ```

4. Limpiar conflictos DHCP:

   ```bash
   HospitalRouter#clear ip dhcp conflict *
   ```

### Problema 4: Llamadas no se conectan

**Síntomas**: Los teléfonos están registrados pero no pueden llamar

**Solución**:

1. Verificar que las extensiones estén configuradas:

   ```bash
   HospitalRouter#show ephone-dn
   ```

2. Verificar que los teléfonos estén asociados a las extensiones:

   ```bash
   HospitalRouter#show ephone summary
   ```

3. Verificar conectividad de red entre teléfonos

---

## 📋 Lista de Verificación Final

- [ ] Todos los dispositivos están correctamente cableados según la distribución optimizada
- [ ] VLANs 10 y 20 creadas en ambos switches con nombres descriptivos
- [ ] Puertos de acceso configurados con voice VLAN y portfast
- [ ] Puertos troncales configurados con VLANs permitidas
- [ ] Subinterfaces del router configuradas con IPs unificadas
- [ ] Pools DHCP configurados con DNS secundario y dominio
- [ ] Servicio de telefonía habilitado con parámetros correctos
- [ ] Extensiones telefónicas configuradas con nombres descriptivos
- [ ] Teléfonos registrados exitosamente (verificar con `show ephone registered`)
- [ ] Llamadas funcionando entre todos los departamentos
- [ ] Conectividad de datos verificada entre VLANs
- [ ] Verificación de enrutamiento con `show ip route`
- [ ] Documentación y capturas de pantalla completas

---

## 📝 Entregables para Evaluación

### 1. Archivo de Cisco Packet Tracer

- Archivo `.pkt` con la configuración completa y funcional
- Todos los dispositivos correctamente nombrados
- Verificación de conectividad realizada

### 2. Informe en PDF que incluya

#### Capturas de Pantalla Requeridas

1. **Topología completa** mostrando todos los dispositivos conectados
2. **Configuración de VLANs** en ambos switches (`show vlan brief`)
3. **Asignación de IPs** en PCs y teléfonos (`ipconfig` y pantalla del teléfono)
4. **Registro de teléfonos** (`show ephone registered`)
5. **Pools DHCP** (`show ip dhcp binding`)
6. **Enrutamiento** (`show ip route`)
7. **Pruebas de llamadas** (capturas de llamadas exitosas)
8. **Conectividad entre VLANs** (ping entre redes)

#### Documentación Adicional

- **Tabla de direccionamiento** con IPs asignadas
- **Plan de numeración** con extensiones por departamento
- **Resumen de configuración** por dispositivo
- **Desafíos encontrados** y soluciones implementadas
- **Recomendaciones** para futuras mejoras

### 3. Verificación de Funcionalidad

- Llamadas exitosas entre todas las extensiones
- Conectividad de datos entre PCs en diferentes switches
- Separación correcta del tráfico de voz y datos
- Servicios DHCP funcionando correctamente

---

## 🎯 Criterios de Evaluación Cubiertos

### Correctitud de la Configuración (50%)

- ✅ **VLANs**: Configuradas correctamente con nombres descriptivos
- ✅ **Troncales**: Configurados con VLANs permitidas específicas
- ✅ **Subinterfaces**: Optimizadas con IPs unificadas
- ✅ **DHCP**: Mejorado con DNS secundario y dominio
- ✅ **Servicios de telefonía**: Completamente configurados con nombres

### Funcionalidad de la Red (30%)

- ✅ **Conectividad entre VLANs**: Verificada con comandos extendidos
- ✅ **Llamadas exitosas**: Probadas entre todos los departamentos
- ✅ **Separación de tráfico**: Datos y voz correctamente segmentados
- ✅ **Escalabilidad**: Diseño preparado para futuras expansiones

### Documentación y Entregables (20%)

- ✅ **Capturas de pantalla**: Especificadas detalladamente
- ✅ **Comandos de verificación**: Extensos y organizados
- ✅ **Solución de problemas**: Completa con síntomas y soluciones
- ✅ **Lista de verificación**: Detallada para asegurar completitud

---
