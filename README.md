
# 🌐 Packet Tracer: Configuración de NAT y PAT - Laboratorio Práctico

<div align="center">

**Laboratorio CISCO - Network Address Translation & Port Address Translation**

[![Cisco Packet Tracer](https://img.shields.io/badge/Cisco-Packet_Tracer-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)](https://www.netacad.com)
[![NAT Protocol](https://img.shields.io/badge/Protocol-NAT/PAT-00A86B?style=for-the-badge)](https://www.cisco.com/)
[![CCNA](https://img.shields.io/badge/Certification-CCNA-blue?style=for-the-badge)](https://www.cisco.com/c/en/us/training-events/training-certifications/certifications/associate/ccna.html)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](LICENSE)

[🎯 Objetivos](#-objetivos) • 
[⚙️ Configuración](#️-configuración-paso-a-paso) • 
[🔍 Verificación](#️-verificación) • 
[📊 Resultados](#️-resultados-y-análisis) • 
[👨‍💻 Autor](#️-autor)

</div>

---

## 📋 Descripción del Proyecto
Este laboratorio práctico de Cisco Packet Tracer implementa dos técnicas fundamentales de traducción de direcciones de red: **NAT Dinámico con Sobrecarga (PAT)** y **PAT mediante Interfaz**. El proyecto demuestra cómo múltiples hosts internos pueden compartir direcciones IP públicas para acceder a Internet, optimizando el uso del limitado espacio IPv4.

### 🎯 Objetivos
**Parte 1:** Configurar NAT Dinámico con Sobrecarga utilizando un pool de direcciones  
**Parte 2:** Verificar la implementación de NAT Dinámico con Sobrecarga  
**Parte 3:** Configurar PAT utilizando la dirección IP de una interfaz  
**Parte 4:** Verificar la implementación de PAT mediante interfaz  

---

## 🛠️ Topología y Direccionamiento

### 🔧 Dispositivos y Redes
| Dispositivo | Red Interna | Gateway | Función |
|-------------|-------------|---------|---------|
| **R1** | 172.16.0.0/16 | R1 | Router con NAT Pool |
| **R2** | 172.17.0.0/16 | R2 | Router con PAT por Interfaz |
| **PC1, L1, PC2, L2** | 172.16.0.0/16 | R1 | Clientes de R1 |
| **PC3, L3, PC4, L4** | 172.17.0.0/16 | R2 | Clientes de R2 |
| **Server1** | Internet | - | Servidor Web Externo |

### 🌐 Direccionamiento Público
| Router | Pool NAT | Rango | Máscara |
|--------|----------|-------|---------|
| **R1** | ANY_POOL_NAME | 209.165.200.233-234 | /30 |
| **R2** | Interfaz S0/1/1 | IP de Interfaz | - |

---


## ⚙️ Configuración Paso a Paso

### Parte 1: Configurar NAT Dinámico con Sobrecarga

#### Paso 1: Configurar ACL para Tráfico Permitido
```cisco
! Crear ACL 1 para permitir red 172.16.0.0/16
R1(config)# access-list 1 permit 172.16.0.0 0.0.255.255
```

#### Paso 2: Configurar Pool de Direcciones NAT
```cisco
! Configurar pool NAT con direcciones públicas
R1(config)# ip nat pool ANY_POOL_NAME 209.165.200.233 209.165.200.234 netmask 255.255.255.252
```

#### Paso 3: Asociar ACL con Pool NAT (Overload)
```cisco
! Configurar NAT dinámico con sobrecarga (PAT)
R1(config)# ip nat inside source list 1 pool ANY_POOL_NAME overload
```

#### Paso 4: Configurar Interfaces NAT
```cisco
! Interface externa (hacia Internet)
R1(config)# interface s0/1/0
R1(config-if)# ip nat outside

! Interfaces internas (hacia redes LAN)
R1(config-if)# interface g0/0/0
R1(config-if)# ip nat inside

R1(config-if)# interface g0/0/1
R1(config-if)# ip nat inside
```

### Parte 2: Verificar NAT Dinámico con Sobrecarga

#### Paso 1: Acceso a Servicios Web
- Desde PC1, L1, PC2, L2 acceder a Server1 via navegador web
- Verificar conectividad exitosa

#### Paso 2: Observar Traducciones NAT
```cisco
R1# show ip nat translations
```

**Resultado Esperado:**
```
Pro Inside global      Inside local       Outside local      Outside global
tcp 209.165.200.233:1024 172.16.1.10:1024 209.165.201.1:80 209.165.201.1:80
tcp 209.165.200.233:1025 172.16.1.11:1025 209.165.201.1:80 209.165.201.1:80
```

### Parte 3: Configurar PAT mediante una Interfaz

#### Paso 1: Configurar ACL para R2
```cisco
! Crear ACL 2 para permitir red 172.17.0.0/16
R2(config)# access-list 2 permit 172.17.0.0 0.0.255.255
```

#### Paso 2: Configurar PAT usando Interfaz
```cisco
! Configurar PAT usando dirección IP de interfaz
R2(config)# ip nat inside source list 2 interface s0/1/1 overload
```

#### Paso 3: Configurar Interfaces NAT en R2
```cisco
! Interface externa
R2(config)# interface s0/1/1
R2(config-if)# ip nat outside

! Interfaces internas
R2(config)# interface [INTERFAZ_INTERNA_1]
R2(config-if)# ip nat inside

R2(config)# interface [INTERFAZ_INTERNA_2]
R2(config-if)# ip nat inside
```

### Parte 4: Verificar Implementación de PAT por Interfaz

#### Paso 1: Acceso a Servicios Web
- Desde PC3, L3, PC4, L4 acceder a Server1 via navegador web
- Verificar conectividad exitosa

#### Paso 2: Observar Traducciones NAT en R2
```cisco
R2# show ip nat translations
```

#### Paso 3: Comparar Estadísticas NAT
```cisco
! En R1
R1# show ip nat statistics

! En R2
R2# show ip nat statistics
```

---

## 📊 Resultados y Análisis

### 🔍 Preguntas del Laboratorio

#### Pregunta 1: ¿Todas las conexiones fueron exitosas? (Parte 2)
**Respuesta:** Sí, todas las conexiones desde los dispositivos internos (PC1, L1, PC2, L2) al servidor web externo fueron exitosas. Esto demuestra que el NAT dinámico con sobrecarga está funcionando correctamente.

#### Pregunta 2: ¿Todas las conexiones fueron exitosas? (Parte 4)
**Respuesta:** Sí, todas las conexiones desde los dispositivos internos (PC3, L3, PC4, L4) al servidor web externo fueron exitosas. Esto confirma que PAT mediante interfaz está operativo.

#### Pregunta 3: ¿Por qué R2 no enumera ninguna asignación dinámica?
**Respuesta:** R2 no enumera asignaciones dinámicas en el pool porque está utilizando **PAT mediante interfaz**. En esta configuración:
- No se define un pool de direcciones explícito
- Se utiliza la dirección IP de la interfaz externa (S0/1/1)
- Todas las traducciones usan la misma dirección IP pública
- Las diferenciaciones se hacen únicamente mediante números de puerto

### 📈 Análisis Técnico

#### Comparación de Métodos NAT

| Característica | NAT con Pool (R1) | PAT por Interfaz (R2) |
|----------------|-------------------|----------------------|
| **Direcciones Públicas** | Múltiples (pool) | Una (interfaz) |
| **Eficiencia de IPs** | Alta | Máxima |
| **Configuración** | Más compleja | Más simple |
| **Escalabilidad** | Limitada por pool | Limitada por puertos |
| **Conexiones Simultáneas** | ~65k por IP | ~65k total |

#### Límites Teóricos y Prácticos
- **Límite teórico PAT:** 65,536 conexiones (campo de puerto de 16 bits)
- **Límite práctico:** Limitado por memoria del dispositivo
- **Uso en R1:** 2 IPs × ~65k = ~130k conexiones teóricas
- **Uso en R2:** 1 IP × ~65k = ~65k conexiones teóricas

### 🎯 Observaciones Clave

1. **Eficiencia de Direcciones:** PAT permite que cientos de dispositivos compartan una sola IP pública
2. **Transparencia:** Los usuarios internos no perciben la traducción
3. **Seguridad:** Oculta la topología interna de red
4. **Flexibilidad:** Múltiples métodos de implementación disponibles

---

## 💡 Conceptos Aprendidos

### 🎯 NAT Dinámico con Sobrecarga (PAT)
- **Mecanismo:** Multiplexación basada en puertos TCP/UDP
- **Ventaja:** Conservación máxima de direcciones IPv4
- **Implementación:** Requiere pool de direcciones públicas
- **Aplicación:** Escenarios con múltiples IPs públicas disponibles

### 🌐 PAT mediante Interfaz
- **Mecanismo:** Usa dirección IP de interfaz como dirección pública única
- **Ventaja:** No requiere pool de direcciones adicional
- **Limitación:** Todos los hosts comparten misma IP pública
- **Aplicación:** Pequeñas empresas/redes domésticas

### 🔧 Comandos Clave Dominados
- `access-list`: Definición de tráfico permitido para NAT
- `ip nat pool`: Configuración de pool de direcciones públicas
- `ip nat inside source`: Asociación ACL con método NAT
- `show ip nat translations`: Verificación de traducciones activas
- `show ip nat statistics`: Estadísticas de rendimiento NAT

---

## 🚀 Guía de Solución de Problemas

### Problemas Comunes y Soluciones

#### ❌ No hay traducciones NAT
```cisco
! Verificar configuración NAT
show running-config | section nat
show ip nat translations
debug ip nat

! Verificar ACL
show access-lists
show ip access-lists
```

#### ❌ Conectividad Unidireccional
```cisco
! Verificar rutas
show ip route
show ip interface brief

! Verificar conectividad
ping [destino] source [origen]
traceroute [destino]
```

#### ❌ Servidor No Accesible desde Internet
```cisco
! Verificar NAT estático (si aplica)
show ip nat translations verbose

! Verificar políticas de firewall
show running-config | include access-list
```

### 🔍 Herramientas de Diagnóstico
| Comando | Propósito |
|---------|-----------|
| `debug ip nat` | Trazas en tiempo real de traducciones |
| `show ip nat statistics` | Métricas de rendimiento |
| `clear ip nat translation *` | Limpiar tabla de traducciones |
| `ping` con `source` | Probar conectividad desde IP específica |

---

## 📚 Recursos Adicionales

### Documentación Oficial Cisco
- [Configuración de NAT en Cisco IOS](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/ipaddr_nat/configuration/15-mt/nat-15-mt-book.html)
- [Guía de Comandos NAT](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/ipaddr_nat/command/nat-cr-book.html)
- [Packet Tracer Tutorials](https://www.netacad.com/courses/packet-tracer)

### Libros Recomendados
- "CCNA 200-301 Official Cert Guide" - Wendell Odom
- "Network Address Translation" - K. Holdaway
- "Cisco PAT Configuration Guide" - Cisco Press

### Laboratorios Prácticos
- [Cisco Networking Academy Labs](https://www.netacad.com)
- [Packet Tracer Community Labs](https://www.packettracernetwork.com)
- [GNS3 NAT Labs](https://gns3.com/marketplace/appliances/nat-lab)

---

## 👨‍💻 Autor

<div align="center">

**Darwin Manuel Ovalles Cesar**

<p align="center">
<a href="https://www.linkedin.com/in/darwin-manuel-ovalles-cesar-dev" target="_blank">
<img align="center" src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/linked-in-alt.svg" alt="LinkedIn - Darwin Ovalles" height="40" width="50" />
</a>
<a href="https://github.com/dovalless" target="_blank">
<img align="center" src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/github.svg" alt="GitHub - Darwin Ovalles" height="40" width="50" />
</a>
</p>

💼 **LinkedIn**: [darwin-manuel-ovalles-cesar-dev](https://www.linkedin.com/in/darwin-manuel-ovalles-cesar-dev/)  
🌐 **GitHub**: [@dovalless](https://github.com/dovalless)  
🎓 **Certificaciones**: CCNA, Network+, A+  

*"Este laboratorio demuestra la importancia crítica de NAT/PAT en redes modernas. La habilidad de permitir que múltiples dispositivos compartan direcciones IP públicas es fundamental para la sostenibilidad de Internet y la seguridad de red."*

**#Cisco #PacketTracer #NAT #PAT #CCNA #Networking #IPv4**

</div>

---

## 📄 Licencia

Este proyecto está bajo la Licencia MIT. Consulta el archivo [LICENSE](LICENSE) para más detalles.

```
MIT License
Copyright (c) 2024 Darwin Manuel Ovalles Cesar
```

---

## 🙏 Agradecimientos

- **Cisco Networking Academy** - Por proporcionar Packet Tracer y recursos educativos
- **Instructores de Redes** - Por su guía en conceptos de NAT/PAT
- **Comunidad de Networking** - Por compartir conocimientos y mejores prácticas

<div align="center">

### ⭐ Si este laboratorio te fue útil, compártelo con otros estudiantes de redes ⭐

### 🚀 ¡Próximo paso: Configuración de NAT Estático y Dinámico! 🚀

**Desarrollado con 💚 para la comunidad de networking**

---
*Laboratorio completado: Packet Tracer - Configure PAT*  
*Habilidades demostradas: NAT Dinámico, PAT, ACLs, Troubleshooting*

</div>
```
