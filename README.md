<div align="center">

# Principios de Diseño: The Twelve-Factor App en DailyPulse

**Selección estratégica de los 4 factores críticos para garantizar seguridad, escalabilidad y desacoplamiento en la arquitectura de Acme Corp.**

![Metodología](https://img.shields.io/badge/Metodología-12--Factor_App-24292F?style=for-the-badge)
![Sistema](https://img.shields.io/badge/Sistema-DailyPulse-0078D4?style=for-the-badge)
![Enfoque](https://img.shields.io/badge/Enfoque-Cloud_Native_&_Agnóstico-2EA44F?style=for-the-badge)

</div>

---

## Resumen Ejecutivo

Para llevar la arquitectura de **DailyPulse** a producción de forma segura y mantenible en los próximos sprints, se priorizaron **4 factores fundamentales** de *The Twelve-Factor App* que mitigan directamente los riesgos operativos de una integración con Microsoft Teams:

| Factor | Nombre | Riesgo que Mitiga | Impacto Principal en DailyPulse |
| :--- | :--- | :--- | :--- |
| **III** | **Config** | Exposición de secretos | Protege tokens de MS Teams y credenciales entre ambientes sin tocar código. |
| **VI** | **Processes** | Pérdida de estado / Cuellos de botella | Permite escalar el backend ante picos simultáneos de reportes diarios. |
| **IV** | **Backing Services** | Acoplamiento tecnológico | Facilita cambiar motores de BD o mensajería sin reescribir reglas de negocio. |
| **XII** | **Admin Processes** | Bloqueo por tareas pesadas | Aísla los recordatorios programados (*Scheduler*) del flujo web principal. |

---

## Análisis Detallado de los 4 Factores Seleccionados

### 1. III. Config (Configuración en el entorno)
* **¿Qué es?** Separar estrictamente toda configuración que varía entre despliegues (credenciales del Bot de MS Teams, tokens de API, cadenas de conexión a la base de datos, umbrales de estancamiento) del código fuente, almacenándolos en variables de entorno.
* **Ventaja e importancia frente al resto:** Al integrarse con un sistema externo corporativo como Microsoft Teams, manejar secretos y llaves de autenticación de forma segura es crítico. Además, permite que Acme Corp pruebe el bot en un canal piloto de desarrollo y luego pase a producción con todos los equipos cambiando únicamente variables de entorno, sin tocar una sola línea de código ni arriesgar filtraciones en el repositorio.

---

### 2. VI. Processes (Procesos sin estado / *Stateless*)
* **¿Qué es?** Ejecutar la aplicación como uno o más procesos que no guardan estado en memoria local ni en el sistema de archivos; cualquier dato que deba persistir se delega a un servicio de respaldo (la **Base de Datos de Dailies**).
* **Ventaja e importancia frente al resto:** En los niveles C2 y C3 diseñamos un **Motor de Dailies (API Core)** que recibe *webhooks* de Teams en cualquier momento y necesita evaluar un historial de 3 días. Si el backend es *stateless*, puede escalar horizontalmente (levantar múltiples instancias si 50 equipos envían su reporte a las 9:00 AM en punto) y recuperarse ante un reinicio sin perder reportes en curso ni romper la detección de estancamientos.

---

### 3. IV. Backing Services (Servicios de respaldo como recursos adjuntos)
* **¿Qué es?** Tratar a la base de datos y a servicios externos (como la API de mensajería de Microsoft Teams) como recursos conectados mediante URLs o credenciales configurables, desacoplados del núcleo de la aplicación.
* **Ventaja e importancia frente al resto:** En nuestro diseño evitamos atarnos a tecnologías específicas desde el inicio. Este factor garantiza que, cuando pasemos a la implementación en los próximos sprints, podamos cambiar de una base de datos local de pruebas a una base de datos gestionada en la nube (o reemplazar un proveedor de notificaciones) sin reescribir la lógica del **Orquestador de Reportes** ni del **Detector de Estancamientos**.

---

### 4. XII. Admin Processes (Procesos de administración)
* **¿Qué es?** Ejecutar las tareas administrativas o periódicas (migraciones de base de datos, limpieza de datos o disparos de tareas programadas) como procesos aislados que usan exactamente el mismo código y configuración que la aplicación principal.
* **Ventaja e importancia frente al resto:** Es el respaldo directo de nuestro contenedor **Planificador de Recordatorios (Scheduler)** en C2. En lugar de incrustar temporizadores frágiles dentro del servidor web que atiende los *webhooks* de Teams, aislar los procesos programados (*"Iniciar Daily"* y *"Auditar Faltantes"*) evita que una carga pesada de notificaciones bloquee la recepción de formularios de los usuarios, manteniendo la operación limpia y predecible.

---

## ¿Por qué estos 4 superan a los demás en este proyecto?

> [!IMPORTANT]
> Mientras que factores como **Concurrency (VIII)** o **Port Binding (VII)** hoy en día vienen resueltos casi por defecto en cualquier plataforma moderna o contenedor, **Config (III)**, **Backing Services (IV)**, **Processes (VI)** y **Admin Processes (XII)** atacan directamente los **4 riesgos reales de DailyPulse**:
> 
> 1. **Seguridad** en la integración con MS Teams (*Config*).
> 2. **Flexibilidad** al no haber definido tecnologías todavía (*Backing Services*).
> 3. **Confiabilidad** en picos de envío simultáneo a la hora del *daily* (*Stateless Processes*).
> 4. **Estabilidad** en las tareas programadas de recordatorio y auditoría (*Admin Processes*).
