# Plan
Planeación del contenido y de apps

# 💚 KoraDevs

> Comunidad de desarrolladores donando conocimiento mediante software 100% libre, sin publicidad ni rastreadores, orientado a niños, microempresas y causas sociales.

---

## 🚀 ¿Cómo funciona nuestra comunidad?

1. **💡 Proponer una App:** Ve a la pestaña **Issues** ➔ **New issue** ➔ Selecciona **🚀 Propuesta de Nueva Aplicación**.
2. **📌 Tablero de Gestión:** El avance global se monitorea en la pestaña **Projects** en el proyecto **GestionApps**.
3. **🛠️ Flujo de Trabajo:**
   - Nadie sube cambios directos a la rama principal (`main`).
   - Cada desarrollo se hace en una rama con el formato: `feature/nombre-tarea` o `fix/nombre-arreglo`.
   - Se crea un **Pull Request (PR)** y requiere al menos 1 aprobación del equipo para fusionarse.

---

## 📜 Principios Clave
- **Sin Publicidad:** Cero anuncios y cero monetización invasiva.
- **Open Source:** Código abierto, público y colaborativo.
- **Package ID Oficial para Apps:** `org.koradevs.[nombre_app]`


1. Incorporación y Roles de los Miembros
 * Invitar a la organización:
   * Asígnar rol correspondiente
 * Crear Equipos (Teams):
   * crea grupos funcionales:
     * @KoraDevsOrg/core (arquitectos y administradores).
     * @KoraDevsOrg/reviewers (encargados de revisar PRs).
     * @KoraDevsOrg/contributors (todos los desarrolladores).
    
2. Flujo de Trabajo para Nuevas Apps
1. Proponer Idea (Issue/Ficha) 
   ➔ 2. Crear Repositorio de la App 
   ➔ 3. Asignar Lead Dev (Maintainer) 
   ➔ 4. Desarrollo vía Ramas & PRs

 * Recepción de la Idea:
   * Utilizar el repositorio base, Plan
   * El proponente presenta el objetivo social, plataforma (Web/Android/iOS) y stack técnico.
 * Creación del Repositorio Específico:
   * Se crea un repositorio público nuevo dentro de la organización (ejemplo: KoraDevsOrg/cuentos-interactivos o KoraDevsOrg/pos-microempresa).
 * Asignación de Responsabilidades:
   * En Settings > Collaborators and teams del nuevo repositorio, se le da permiso de Maintainer al autor de la propuesta.
   * El líder define la arquitectura base, crea el backlog de tareas (Issues) y coordina el proyecto.

3. Reglas de Desarrollo y Calidad (Branching Policy)
Para mantener el código limpio y seguro:
 * Protección de la rama main:
   * Configurar en cada repositorio que main requiera al menos 1 Pull Request review antes de hacer merge.
   * Nadie hace git push directo a main.
 * Nomenclatura de ramas para los desarrolladores:
   * feature/[nombre-tarea] (ej. feature/login-ui, feature/motor-audio)
   * fix/[nombre-bug] (ej. fix/calculo-iva, fix/crash-pantalla)
   * docs/[seccion] (ej. docs/instalacion-local)
 * Kanban Board:
   * En la pestaña Projects de la organización, creen un tablero global tipo Kanban con 4 columnas:
     * Propuestas / Backlog
     * En Desarrollo
     * En Pruebas / Code Review
     * Publicadas / Listas

Bienvenidos.
