# 09 · Active Directory Integration

![Okta](https://img.shields.io/badge/Okta-Workforce_Identity-007DC1?style=flat-square&logo=okta&logoColor=white)
![Category](https://img.shields.io/badge/Category-Directory_Services-37474F?style=flat-square)
![Level](https://img.shields.io/badge/Level-Advanced-F44336?style=flat-square)
![Feature](https://img.shields.io/badge/Feature-Okta_AD_Agent-FF6B35?style=flat-square)

---

## Why this matters

The majority of enterprises run on Active Directory. It's been the backbone of on-premises identity for decades users, computers, group policies, all managed through AD. Moving to the cloud doesn't mean ripping that out; it means extending it.

This lab covers how to connect an existing Active Directory to Okta using the **Okta AD Agent**, so that AD becomes a delegated source for user authentication and Okta becomes the bridge to the cloud. Users log in with their Windows credentials, Okta authenticates against AD, and SSO flows to every cloud app from there. This is the most common integration pattern for enterprise Okta deployments.

---

## Architecture

```mermaid
flowchart LR
    subgraph On-Premises
        AD[Active Directory\nDomain Controller]
        Agent[Okta AD Agent\nWindows Server]
        AD <-->|LDAP| Agent
    end
    subgraph Cloud
        Okta[Okta\nIdentity Cloud]
    end
    subgraph SaaS Apps
        M365[Microsoft 365]
        SF[Salesforce]
        GW[Google Workspace]
    end
    Agent <-->|HTTPS outbound only| Okta
    Okta -->|SSO| M365 & SF & GW
    User[Corporate User] -->|AD Credentials| Okta
    Okta -->|Delegated Auth via Agent| AD
```

---

## Prerequisites

- Windows Server 2016+ with Active Directory Domain Services
- Service account in AD with read access to the directory
- Okta org (any edition)
- Network connectivity between the AD Agent server and the internet (HTTPS outbound)

---

## Lab Walkthrough

### Step 1 · Download and install the Okta AD Agent

In Okta Admin Console, go to **Directory → Directory Integrations → Add Directory → Active Directory**. Download the AD Agent installer.

![Step 1 — Download AD Agent from Okta console](./screenshots/01-download-ad-agent.png)
*The AD Agent requires only outbound HTTPS to Okta, no inbound firewall rules, no VPN, no change to your AD infrastructure.*

---

### Step 2 · Run the agent installer on the Windows Server

Install the agent on a domain-joined Windows Server. During setup, authenticate with your Okta org credentials to register the agent.

![Step 2 — AD Agent installation wizard](./screenshots/02-agent-installation.png)
*Run the installer as a local administrator. The agent creates a Windows service that runs continuously and maintains a persistent connection to Okta.*

---

### Step 3 · Configure the Active Directory connection in Okta

After installation, complete the setup in Okta by specifying the AD domain, Base DN (where users live in AD), and service account credentials.

![Step 3 — AD connection configuration in Okta](./screenshots/03-ad-connection-config.png)
*The Base DN scopes which part of AD Okta will import use an OU that contains only the users you want in Okta, not the entire directory.*

---

### Step 4 · Map AD attributes to Okta Universal Directory

Under **Profile Editor → Active Directory**, configure the attribute mappings. Map `sAMAccountName` to `login`, `mail` to `email`, `givenName` to `firstName`, etc.

![Step 4 — AD to Okta attribute mappings](./screenshots/04-attribute-mappings.png)
*AD uses different attribute names than Okta these mappings normalize them. Custom AD attributes (like extension attributes) can also be mapped to Okta custom profile fields.*

---

### Step 5 · Run the initial user import

Click **Import Now** to do a full import of AD users into Okta. Review the import results and confirm the accounts are being mapped correctly.

![Step 5 — Initial AD user import results](./screenshots/05-user-import.png)
*The first import is manual after that, incremental syncs happen on a schedule. New AD users and changes propagate to Okta automatically.*

---

### Step 6 · Confirm users in Okta with AD as the profile master

Check a user's profile in Okta. You'll see the AD icon indicating that AD is the **profile master**  Okta inherits and syncs from AD, not the other way around.

![Step 6 — User profile showing AD as profile master](./screenshots/06-ad-profile-master.png)
*If AD is the profile master, changes to the user in Okta (like editing the department field) will be blocked the change must happen in AD first.*

---

### Step 7 · Enable delegated authentication to AD

In the **Directory Integration** settings, enable **Delegated Authentication**. This means Okta will pass the user's password to AD for verification, so users log in with their Windows password.

![Step 7 — Delegated authentication enabled](./screenshots/07-delegated-auth.png)
*Delegated auth means Okta never stores or manages the AD password the AD Agent proxies the authentication challenge to the domain controller.*

---

### Step 8 · Test end-to-end login with AD credentials

Open a private browser, go to your app, and sign in using an AD username and password. Confirm the login succeeds and that the Okta session is established.

![Step 8 — Successful login using Active Directory credentials](./screenshots/08-ad-login-success.png)
*From the user's perspective, nothing changes they use the same password they use for Windows. Okta handles the bridge to all cloud apps.*

---
## What I Learned

**El AD Agent solo necesita conectividad saliente en el puerto 443.** No requiere abrir puertos en el firewall, configurar una DMZ, ni modificar la infraestructura de AD. El agente crea una conexión persistente hacia Okta. Okta nunca inicia conexiones hacia el entorno on-premise. Esto es lo que hace que la integración sea aceptable para equipos de seguridad conservadores.

**La diferencia entre Profile Master y Delegated Authentication es fundamental.** El Profile Master define quién controla los atributos del usuario, si AD es el profile master, los cambios en Okta son sobrescritos en el siguiente sync. La Delegated Authentication define quién verifica la contraseña, si está activa, Okta pasa la autenticación al Domain Controller y nunca almacena ni gestiona la contraseña de Windows.

**El Base DN es el scope de lo que Okta importa.** Si configuras el Base DN en la raíz del dominio (`DC=corp,DC=acme,DC=local`), Okta importará todos los objetos incluyendo cuentas de servicio, cuentas de administrador, y cuentas deshabilitadas. La práctica correcta es apuntar a la OU que contiene solo los usuarios que deben tener acceso a Okta.

**El attribute mapping es donde viven los errores más difíciles de debuggear.** Si `sAMAccountName` no está mapeado a `login`, los usuarios no pueden autenticarse. Si `mail` no está mapeado a `email`, las notificaciones de Okta van a ningún lado. Revisar los mappings antes del primer import es el paso más importante del lab.

**Los usuarios importados tienen AD como profile master, Okta no puede editarlos.** Si intentas cambiar el departamento de un usuario importado desde AD directamente en Okta, el cambio será sobrescrito en el siguiente sync. El cambio tiene que hacerse en Active Directory, Okta recoge el cambio en el siguiente ciclo de sincronización.

**La sincronización incremental después del import inicial es automática.** El primer import es manual y trae todos los usuarios del scope. Después, el agente sincroniza cambios cada 4 horas por defecto nuevos usuarios en AD aparecen en Okta automáticamente, cambios de atributos se propagan, y usuarios deshabilitados en AD se desactivan en Okta.

---

## Troubleshooting

| Error | Causa | Fix |
|---|---|---|
| AD Agent no aparece como activo después de instalarlo | El agente no puede alcanzar Okta en el puerto 443 | Verificar que el servidor Windows tiene salida HTTPS sin restricciones revisar proxy corporativo y firewall |
| "Failed to connect to AD" durante la configuración | El Base DN es incorrecto o la cuenta de servicio no tiene permisos | Verificar el Base DN con `dsquery` en el servidor AD, y que la cuenta de servicio tiene "Read" en la OU objetivo |
| Usuarios importados sin email | El atributo `mail` no está poblado en AD | Completar el atributo `mail` en los objetos de usuario en AD antes de hacer el import |
| Delegated auth falla usuarios no pueden autenticarse | La cuenta de servicio del agente no tiene permisos para verificar credenciales | La cuenta de servicio necesita el permiso "Validate credentials" en el dominio |
| Sync no propaga cambios hechos en AD | El ciclo de sync de 4 horas no ha corrido aún | Forzar un sync manual desde el tab Import → Import Now en el Admin Console |
| Usuarios de AD aparecen duplicados en Okta | Ya existían usuarios en Okta con el mismo email antes del import | Usar la función de "Match" durante el import para vincular usuarios existentes con sus cuentas de AD en vez de crear nuevos |
| Agent se desconecta periódicamente | El servicio de Windows del agente se detiene por actualizaciones o reinicios | Configurar el servicio "Okta AD Agent" para reinicio automático en el Administrador de servicios de Windows |

---


## Real-World Applications

- Extending existing AD-based authentication to cloud apps without migrating to cloud-only identity
- Giving remote employees SSO to Salesforce and Microsoft 365 using their existing AD password, without a VPN
- Phased migration from on-prem to cloud: Okta bridges the gap while AD remains the source of truth during transition

---

## Resources

- [Okta Active Directory Integration guide](https://help.okta.com/en-us/content/topics/directory/ad-agent-main.htm)
- [Okta AD Agent prerequisites](https://help.okta.com/en-us/content/topics/directory/ad-agent-prerequisites.htm)
- [Universal Directory overview](https://help.okta.com/en-us/content/topics/directory/dir-profile-editor-about.htm)

