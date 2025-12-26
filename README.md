# AWS Enterprise Network: From Manual Deep Dive to IaC Automation 🚀

![Architecture Diagram](arquitectura-vpc.png)

## 1️⃣ Historia del Proyecto

En el mundo Cloud, es fácil ejecutar un script y ver cómo se crea la magia. Pero como Ingeniera Cloud, creo firmemente en la regla: **"No automatices lo que no entiendes"**.

Este proyecto, **aws-vpc-network-iac**, constó de dos fases intensivas:

### 🔍 Fase 1: Manual Deep Dive
Primero, construí toda la arquitectura de red manualmente desde la consola de AWS, configurando uno a uno:
* Subnets
* Route Tables
* Internet Gateway
* NAT Gateway
* VPC Endpoints

**🎯 Objetivo:**
Comprender el flujo real de los paquetes de red y visualizar la seguridad por capas antes de automatizar.

### ⚙️ Fase 2: IaC Automation (Scale Up)
Una vez dominados los fundamentos, traduje toda la arquitectura a **Infraestructura como Código (IaC)** utilizando **AWS CloudFormation**.

**✅ Resultado:**
Un despliegue que pasó de **45 minutos manuales a ~3 minutos**, totalmente reproducible y sin errores humanos.

---

## 2️⃣ Arquitectura Desplegada

La plantilla `template.yaml` aprovisiona una **VPC de Alta Disponibilidad** diseñada para entornos de producción seguros:

* **VPC Segmentada:** Bloque CIDR `10.0.0.0/16` dividido en zonas.
* **Subredes Públicas y Privadas:**
    * *Públicas:* Para balanceadores o Bastion hosts.
    * *Privadas:* Aislamiento total para bases de datos o backends.
* **Seguridad Perimetral (NAT Gateway):**
    * Permite a las instancias privadas descargar actualizaciones de internet.
    * Bloquea cualquier intento de conexión entrante desde el exterior.
* **Privacidad de Datos (VPC Endpoints):**
    * Implementación de **Gateway Endpoint** para S3.
    * Implementación de **Interface Endpoint** para KMS.
    * *Beneficio:* El tráfico sensible viaja exclusivamente por la red troncal de AWS, nunca toca la internet pública.

---

## 3️⃣ Validación Técnica (Evidence)

No basta con desplegar, hay que validar. Estas son las pruebas de conectividad realizadas:

### ✅ 3.1 Prueba de Salida Segura (NAT Gateway)
Desde una instancia en la **Subnet Privada** (sin IP pública), logramos conexión a internet. Esto confirma que el enrutamiento a través del NAT Gateway funciona correctamente.

![Ping Test](tes-private-ping.PNG)
*(El ping exitoso demuestra salida a internet, mientras la instancia permanece invisible al exterior).*

### .

🔐 3.2 Privacidad DNS (VPC Endpoints)
Validación de resolución DNS interna para AWS KMS. Al usar `dig`, vemos que la respuesta es una IP privada (`10.x.x.x`), confirmando que el tráfico no sale de la red de AWS.

![DNS Dig Test](cli-endpoint-private-kms.PNG)

---

## 4️⃣ Guía de Despliegue (Quick Start)

Si deseas replicar esta infraestructura en tu cuenta AWS:

### 1. Preparar el entorno
Abre tu terminal y navega a la carpeta del proyecto:
```bash
PS C:\proyectos-aws\proyect-04-vpc-networking-cfn-iac> 
```
### 2. Desplegar con CloudFormation
```bash
aws cloudformation deploy --template-file template.yaml --stack-name vpc-networking --capabilities CAPABILITY_IAM
```
## 5️⃣ Test Connectivity (Comandos de Validación)
Una vez desplegada la infraestructura, accedemos a la instancia privada mediante **AWS Systems Manager (SSM)** y ejecutamos las siguientes pruebas para certificar la red:

### 1. Verificar conectividad interna (Ping entre instancias privadas)
```bash
ping 10.0.0.242 -c 5
```

### 2. Confirmar salida a Internet (Validación de NAT Gateway)
Si responde, la instancia privada tiene acceso a internet para actualizaciones.
```bash
ping example.com -c 5
```
### 3. Prueba de latencia hacia IP Pública específica
```bash
ping 52.23.201.228 
```
### 4. Validar VPC Interface Endpoint para KMS
IMPORTANTE: La respuesta debe ser una IP Privada (10.x.x.x), 
confirmando que el tráfico NO sale a la internet pública.
```bash
dig kms.us-east-1.amazonaws.com
```
### 6️⃣ Limpieza de Recursos (Clean Up)
Para eliminar todos los recursos y evitar costos (especialmente del NAT Gateway), ejecuta:
```bash
aws cloudformation delete-stack --stack-name vpc-networking
```
