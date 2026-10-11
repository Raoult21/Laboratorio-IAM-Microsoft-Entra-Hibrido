# Arquitectura Híbrida de Identidades: Microsoft Entra ID & Windows Server 2022

## 📝 Descripción del Proyecto
Este proyecto documenta la implementación de un entorno empresarial simulado para la gestión de identidades y accesos (IAM). Se diseñó e implementó una infraestructura híbrida conectando un entorno local (On-Premises) en Windows Server 2022 con el Tenant en la nube de Microsoft Entra ID, garantizando la persistencia, seguridad y automatización del ciclo de vida de los usuarios.

---

## 📐 Arquitectura del Laboratorio
*   **Ambiente Cloud:** Microsoft Entra ID (Tenant Corporativo aislado).
*   **Ambiente Local:** Windows Server 2022 (Controlador de Dominio `lab.local`) montado en VirtualBox.
*   **Estructura Organizativa Local:** Unidad Organizativa dedicada (`sync`) para delimitar el alcance del aprovisionamiento.
*   **Mecanismo de Enlace:** Agente moderno Microsoft Entra Cloud Sync (`AADConnectProvisioningAgentSetup.exe`).

---

## 🚀 Hitos Técnicos Implementados

### 1. Gobernanza de Identidades y Carga Masiva (Cloud)
*   Diseño, sanitización y carga masiva de una nómina corporativa utilizando archivos de datos estructurados (**CSV**).
*   Configuración de un modelo inicial de control de accesos basado en roles (**RBAC**) mediante la creación de grupos de seguridad en la nube (ej. `SG-CyberSecurity-Team`).
*   Implementación de directivas de acceso seguro perimetral mediante la activación de **Security Defaults (MFA Obligatorio)**.

### 2. Infraestructura y Servicios de Directorio (Local)
*   Despliegue de Windows Server 2022, promoción a Controlador de Dominio principal y resolución técnica del direccionamiento de red y DNS perimetral.
*   Creación manual de identidades locales utilizando la consola de *Active Directory Users and Computers*.

### 3. Conectividad Híbrida y Sincronización
*   Establecimiento de un túnel seguro de salida hacia los endpoints globales de Microsoft utilizando el agente de aprovisionamiento en la nube.
*   Configuración del alcance (*Scope*) de sincronización restringido estrictamente a la Unidad Organizativa personalizada para optimizar el tráfico de red.

---

## 🛠️ Bitácora de Troubleshooting (Resolución de Problemas Reales)

El mayor valor de este laboratorio radicó en la resolución de bloqueos críticos de infraestructura durante el despliegue:

*   **Error Criptográfico SCHANNEL (0xCAA80000):** Se diagnosticó que Windows Server 2022 bloqueaba las conexiones web de salida del instalador por falta de cifrado fuerte. Se solucionó inyectando llaves de registro por PowerShell para forzar el uso nativo del protocolo **TLS 1.2**.
*   **Interferencia de Windows Hello (Bucle de Llave USB Física / WAM):** El motor moderno intentaba invocar credenciales de hardware inexistentes en el ambiente virtual. Se evadió el bloqueo perimetral desactivando el Administrador de Cuentas Web por comandos de consola.
*   **Restricciones de Navegador (IE ESC):** Los scripts de inicio de sesión de Microsoft eran bloqueados por la configuración de seguridad mejorada de Internet Explorer. Se desactivó el *IE ESC* en el *Server Manager* para permitir la carga fluida de los formularios de autenticación.
*   **Error de Alcance (ResourceNotFound):** El agente reportaba que los nuevos usuarios no existían en el dominio local. Se analizó la estructura de AD DS y se identificó que los usuarios estaban fuera del alcance de escaneo. Al reubicar las cuentas de la carpeta *Users* hacia la OU *sync*, el flujo se normalizó instantáneamente.
*   **Bloqueo por Cuarentena (Provisioning Quarantine):** Debido a apagados inesperados previos registrados en los logs de DFS y del sistema (`dcdiag`), la nube congeló el conector. Se rompió el bloqueo perimetral realizando un reinicio limpio del servidor local y ejecutando un *Restart Provisioning* directo en la API web para devolver la regla a un estado **Healthy**.

---

## 📊 Validación Final del Éxito
El pipeline híbrido se consolidó de forma exitosa. Se verificó en el portal web que el estado del agente se encuentra **Active** y la regla general reporta como **Healthy**. Las identidades locales creadas en el servidor (como los trillizos Gómez) impactaron de forma automatizada en el listado global de usuarios de Microsoft Entra ID, luciendo el atributo de sincronización local habilitada en **`Yes`**.
