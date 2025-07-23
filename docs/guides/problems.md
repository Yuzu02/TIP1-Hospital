# 🚨 Problemas Técnicos y Resoluciones durante la Implementación

## Proyecto: Red VoIP Hospital "Salud Integral"

### 📝 **Registro Personal de Incidencias**

Este documento registra los **problemas técnicos reales** que encontré durante la implementación de la red VoIP, las metodologías de diagnóstico utilizadas y las soluciones aplicadas. Sirve como referencia personal para futuras implementaciones y como material de aprendizaje.

### 🎯 **Contexto del Proyecto**

- **Objetivo**: Implementar telefonía IP con segregación de tráfico (VLANs 10/20)
- **Entorno**: Cisco Packet Tracer con Router 2811 y Switches 2960
- **Desafío Principal**: Primera implementación de VoIP con CME

### ⚠️ **Problemas Identificados**

1. **🔴 CRÍTICO: Error de Solapamiento de Red (IP Overlap)**
2. **🟡 MEDIO: Fallo de Registro de Teléfonos IP en Switch Secundario**  
3. **🟢 MENOR: Limitaciones del Router C2811 vs C2911**

---

## 🔴 **PROBLEMA #1: Error de Solapamiento de Red (IP Overlap)**

### 🐛 **Síntomas Detectados**

Durante la configuración del segundo switch, al intentar configurar las subinterfaces en `FastEthernet0/1` del router, el sistema generó este error:

```cisco
HospitalRouter(config-subif)# ip address 192.168.80.1 255.255.255.128
% 192.168.80.0 overlaps with FastEthernet0/0.10
HospitalRouter(config-subif)#
```

### 🔍 **Análisis Técnico**

**Mi error conceptual inicial**: Intenté aplicar el mismo esquema de direccionamiento en dos interfaces físicas diferentes del router, violando principios básicos de enrutamiento.

**Configuración problemática que intenté aplicar**:

```cisco
! Interface F0/0 (hacia Switch1) - YA CONFIGURADA
interface FastEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.80.1 255.255.255.128    ✅ FUNCIONA

! Interface F0/1 (hacia Switch2) - AQUÍ FALLÓ  
interface FastEthernet0/1.10
 encapsulation dot1Q 10
 ip address 192.168.80.1 255.255.255.128    ❌ ERROR: IP OVERLAP
```

### 💡 **Comprensión del Problema**

- **Principio violado**: Un router no puede tener la misma subred en múltiples interfaces
- **Función del router**: Interconectar redes **diferentes**, no duplicar la misma red
- **Mi confusión inicial**: Pensé que necesitaba replicar la configuración en cada interfaz física

### ✅ **Solución Implementada**

**Cambio de arquitectura**: De conectividad punto a punto a topología en cascada

**ANTES (Problemático)**:

```
Router ──── Switch1 (VLANs 10,20)
  └─────── Switch2 (VLANs 10,20) ❌ Duplicación de redes
```

**DESPUÉS (Funcional)**:

```
Router ──── Switch1 ──── Switch2
              ↑           ↑
        Single point    Troncal
        of routing     802.1Q
```

**Comandos de corrección aplicados**:

```cisco
! ROUTER: Desactivar F0/1 (ya no se usa)
interface FastEthernet0/1
 description NO_UTILIZADO_PARA_LAN  
 shutdown

! SWITCH1: Configurar enlace troncal hacia Switch2
interface GigabitEthernet0/2
 description TRONCAL_SWITCH2
 switchport mode trunk
 switchport trunk allowed vlan 10,20

! SWITCH2: Configurar enlace troncal hacia Switch1  
interface GigabitEthernet0/2
 description TRONCAL_SWITCH1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
```

### 📚 **Lección Aprendida**

- Las VLANs se **extienden** a través de enlaces troncales, no se **duplican** en interfaces de router
- Un solo punto de enrutamiento simplifica la gestión y evita conflictos
- La topología en cascada es más escalable para redes empresariales

---

## 🟡 **PROBLEMA #2: Teléfonos IP No Se Registran en Switch2**

### 🐛 **Síntomas Detectados**

Después de solucionar el problema de arquitectura:

- ✅ **PCs en Switch2**: Obtenían IP de VLAN 10 correctamente
- ❌ **Teléfonos en Switch2**: Quedaban en "Configuring IP..." indefinidamente  
- ✅ **Teléfonos en Switch1**: Funcionaban perfectamente

### 🔍 **Proceso de Diagnóstico que Seguí**

**1. Verificación de troncales**:

```cisco
Switch2-Hospital# show interfaces trunk

Port        Mode         Encapsulation  Status        Native vlan
Gi0/2       on           802.1q         trunking      1

Port        Vlans allowed on trunk  
Gi0/2       10,20                    ✅ CORRECTO
```

**2. Verificación de VLANs**:

```cisco
Switch2-Hospital# show vlan brief

VLAN Name                Status    Ports
10   DATOS_HOSPITAL      active    
20   VOZ_HOSPITAL        active    Fa0/1,Fa0/2,Fa0/3,Fa0/4  ❌ PROBLEMA!
```

**3. ¡Eureka!** Los puertos estaban mal configurados

### 💡 **Causa Raíz Identificada**

**Mi error en Switch2**: Configuré los puertos de acceso como VLAN 20 en lugar de VLAN 10

```cisco
! CONFIGURACIÓN INCORRECTA QUE APLIQUÉ
interface range FastEthernet0/1-4
 switchport mode access
 switchport access vlan 20    ❌ ERROR: Puse la VLAN de voz como acceso
 switchport voice vlan 20
```

**Por qué falló**:

- El puerto de **acceso** (datos/PC) debe estar en VLAN 10
- El puerto de **voz** (teléfono) debe estar en VLAN 20  
- Al tener ambos en VLAN 20, creé un conflicto de configuración

### ✅ **Solución Implementada**

**Corrección de configuración en Switch2**:

```cisco
! ANTES (Incorrecto)
interface range FastEthernet0/1-4
 switchport mode access
 switchport access vlan 20    ❌ ERROR
 switchport voice vlan 20

! DESPUÉS (Correcto)  
interface range FastEthernet0/1-4
 switchport mode access
 switchport access vlan 10    ✅ DATOS/PC
 switchport voice vlan 20     ✅ VOZ/TELÉFONO
 spanning-tree portfast
```

**Resultado**: Los teléfonos obtuvieron IP inmediatamente y se registraron en CME

### 📚 **Lección Aprendida**

- Los puertos de teléfonos requieren **dos VLANs**: acceso para datos (PC) y voz para teléfono
- El comando `show vlan brief` es clave para diagnosticar asignaciones incorrectas
- Siempre verificar la configuración puerto por puerto, no asumir que está correcta

---

## 🟢 **PROBLEMA #3: Limitaciones del Router C2811 vs C2911**

### 🐛 **Problema Inicial**

**Planeaba usar**: Router C2911 (más moderno)
**Descubrí**: El C2911 no soporta `telephony-service` en Packet Tracer

**Error encontrado**:

```cisco
Router(config)# telephony-service
               ^
% Invalid input detected at '^' marker.
```

### 🔍 **Investigación Realizada**

- **C2911**: No incluye el módulo de telefonía en Packet Tracer
- **C2811**: Sí incluye soporte completo para CME (Call Manager Express)
- **Decisión**: Cambiar toda la implementación al C2811

### ⚙️ **Adaptaciones Necesarias**

**1. Cambio de interfaces**:

```cisco
! PLANIFICADO (C2911)         IMPLEMENTADO (C2811)
GigabitEthernet0/0         →   FastEthernet0/0
GigabitEthernet0/1         →   FastEthernet0/1
```

**2. Limitaciones encontradas**:

```cisco
! COMANDO NO FUNCIONA EN C2811
ephone-dn 1
 number 101
 name "Recepcion-101"    ❌ No soportado

! SOLUCIÓN APLICADA  
ephone-dn 1
 number 101              ✅ Solo número
```
