# Guía de Desarrollo del Ecosistema Kora

Bienvenido al equipo de desarrollo de **KoraDevsOrg**[span_0](start_span)[span_0](end_span)[span_1](start_span)[span_1](end_span). Este documento establece los fundamentos arquitectónicos, convenciones y procedimientos para crear y conectar aplicaciones al ecosistema **Kora**[span_2](start_span)[span_2](end_span).

El objetivo central es construir herramientas **libres de publicidad, 100% funcionales sin conexión (Local-First) y con costo cero de infraestructura ($0 servidores)**[span_3](start_span)[span_3](end_span)[span_4](start_span)[span_4](end_span).

---

## 1. Filosofía y Modelo Arquitectónico

A diferencia de las arquitecturas tradicionales donde cada aplicación administra su propia base de datos o depende de APIs externas[span_5](start_span)[span_5](end_span), el ecosistema Kora funciona como un **ERP Móvil Descentralizado**[span_6](start_span)[span_6](end_span)[span_7](start_span)[span_7](end_span):

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                          DISPOSITIVO DEL USUARIO                            │
│                                                                             │
│   [ Micro-Apps Web HTML/JS ]              [ Apps Nativas Android (APK) ]    │
│   (Quiz Japonés, Francés, etc.)           (Finanzas, Juegos, Utilidades)    │
│              │                                           │                  │
│     window.KoraDB Bridge                         KoraClient SDK             │
│   (kora-web-sdk vía jsDelivr)            (IPC / Signature Permissions)      │
│              │                                           │                  │
│              └─────────────────────┬─────────────────────┘                  │
│                                    ▼                                        │
│                 ┌──────────────────────────────────────┐                    │
│                 │          KORA ADMIN DB (HUB)         │                    │
│                 │  • Base de datos central (SQLite)    │                    │
│                 │  • Visor Web seguro (Runtime)        │                    │
│                 │  • Tienda / Gestor de Paquetes       │                    │
│                 └──────────────────┬───────────────────┘                    │
│                                    │                                        │
│                      ┌─────────────┴─────────────┐                          │
│                      ▼                           ▼                          │
│           [ Tablas del Núcleo ]       [ Módulos ERP Satélite ]              │
│             core_apps_registry             mod_jp_palabras                  │
│             core_state_anchor              mod_fr_vocabulario               │
│                                            mod_fin_gastos                   │
└─────────────────────────────────────────────────────────────────────────────┘
```[span_8](start_span)[span_8](end_span)[span_9](start_span)[span_9](end_span)

### Principios Fundamentales
* **Persistencia Única:** El usuario solo almacena un archivo físico de base de datos (`kora_master.db`), gestionado por `kora-admin-db` en modo WAL (*Write-Ahead Logging*)[span_10](start_span)[span_10](end_span)[span_11](start_span)[span_11](end_span).
* **Modularidad Aditiva:** Cada nueva app que se instala no fragmenta el almacenamiento; inyecta dinámicamente sus propias tablas relacionales (`CREATE TABLE IF NOT EXISTS`)[span_12](start_span)[span_12](end_span)[span_13](start_span)[span_13](end_span).
* **Resiliencia Offline:** La aplicación debe funcionar sin red[span_14](start_span)[span_14](end_span). Si hay conectividad, sincroniza deltas desde GitHub; si está offline, opera sin fallos contra la base de datos local.

---

## 2. Convenciones de Nomenclatura

Para garantizar la integridad y evitar colisiones de tablas entre módulos[span_15](start_span)[span_15](end_span):

| Elemento | Convención | Ejemplo |
| :--- | :--- | :--- |
| **Repositorio** | `kebab-case` en minúsculas[span_16](start_span)[span_16](end_span) | `quiz-japones`, `kora-admin-db` |
| **ID de Paquete** | `org.koradevs.<categoria>.<nombre>`[span_17](start_span)[span_17](end_span)[span_18](start_span)[span_18](end_span) | `org.koradevs.quiz.japon`, `org.koradevs.finanzas`[span_19](start_span)[span_19](end_span) |
| **Tablas del Núcleo** | Prefijo `core_*` (Reservado para Admin DB)[span_20](start_span)[span_20](end_span) | `core_apps_registry`, `core_state_anchor`[span_21](start_span)[span_21](end_span) |
| **Tablas de Apps** | Prefijo `mod_<id_corto>_*`[span_22](start_span)[span_22](end_span) | `mod_jp_palabras`, `mod_jp_srs`, `mod_fin_gastos`[span_23](start_span)[span_23](end_span) |
| **Identificadores (PK)**| Texto unívoco o UUID[span_24](start_span)[span_24](end_span) | `jp_001`, `comer_taberu`, `cat_gramatica` |

---

## 3. Desarrollo de Micro-Apps Web (HTML5 / JavaScript)

Las micro-apps son interfaces ligeras desarrolladas con estándares web y alojadas en GitHub Pages[span_25](start_span)[span_25](end_span)[span_26](start_span)[span_26](end_span). Operan tanto en navegadores estándar como embebidas dentro de `Kora Admin DB`[span_27](start_span)[span_27](end_span).

### Paso 1: Importar el SDK Web
En el `<head>` de tu `index.html`, incluye el motor universal de persistencia desde el repositorio `kora-web-sdk` a través de CDN[span_28](start_span)[span_28](end_span):

```html
<script src="[https://cdn.jsdelivr.net/gh/KoraDevsOrg/kora-web-sdk@main/kora-sync.js](https://cdn.jsdelivr.net/gh/KoraDevsOrg/kora-web-sdk@main/kora-sync.js)"></script>

Paso 2: Configurar KoraSyncEngine
En el punto de entrada JavaScript de tu aplicación (por ejemplo, app.js), inicializa el motor con el esquema DDL y el manejador de inserción:
import { ITEMS_DATA } from "./data/items.js";

document.addEventListener("DOMContentLoaded", async () => {
  let activeData = [...ITEMS_DATA];

  // 1. Detectar si corre dentro del entorno Kora
  if (typeof window.KoraSyncEngine !== "undefined") {
    const engine = new window.KoraSyncEngine({
      pkgName: "org.koradevs.mi.modulo",
      appName: "Kora Mi Módulo",
      tableName: "mod_mi_tabla",
      currentHtmlVersion: "1.0.0",
      tableDdl: `
        CREATE TABLE IF NOT EXISTS mod_mi_tabla (
          id TEXT PRIMARY KEY,
          titulo TEXT,
          descripcion TEXT,
          categoria TEXT
        );
      `,
      insertHandler: (db, item) => {
        const sql = `
          INSERT OR REPLACE INTO mod_mi_tabla (id, titulo, descripcion, categoria)
          VALUES (?, ?, ?, ?);
        `;
        db.execute(sql, JSON.stringify([
          item.id,
          item.titulo || "",
          item.descripcion || "",
          item.categoria || "general"
        ]));
      }
    });

    // 2. Ejecutar ciclo de sincronización (Offline-First)
    await engine.sync(ITEMS_DATA, {
      onStatus: (msg) => console.log("[KoraSync]:", msg),
      onPrompt: (promptMsg) => window.confirm(promptMsg) // Pide autorización si hay novedades
    });

    // 3. Si el visor nativo está presente, cargar desde SQLite
    if (engine.hasBridge) {
      const dbRows = engine.getAll();
      if (dbRows && dbRows.length > 0) {
        activeData = dbRows;
      }
    }
  }

  // 4. Arrancar la lógica del aplicativo con activeData
  inicializarApp(activeData);
});

Paso 3: Registrar la Web App en el Catálogo Global
Para que tu aplicación aparezca en la pestaña Tienda Kora de Kora Admin DB, edita el archivo kora-catalog.json en el repositorio kora-admin-db:
{
  "id": "mi_app_id",
  "name": "Kora Mi Aplicación",
  "description": "Breve resumen funcional de la herramienta.",
  "type": "WEB_APP",
  "packageName": "org.koradevs.mi.modulo",
  "version": "1.0.0",
  "versionCode": 1,
  "sourceUrl": "[https://koradevsorg.github.io/mi-repositorio/](https://koradevsorg.github.io/mi-repositorio/)"
}

4. Desarrollo de Aplicaciones Nativas Android (Kotlin / Java)
Para aplicaciones nativas de alto rendimiento (juegos pesados o herramientas del sistema), la comunicación con kora_master.db se realiza mediante IPC seguro (ContentProvider).
Paso 1: Configurar Permisos
En el AndroidManifest.xml de tu aplicación Android, solicita el permiso protegido por firma del ecosistema:
<manifest xmlns:android="[http://schemas.android.com/apk/res/android](http://schemas.android.com/apk/res/android)">

    <!-- Permiso obligatorio del ecosistema Kora -->
    <uses-permission android:name="org.koradevs.permission.ACCESS_KORA_DB" />

    <application ...>
        <!-- Tu configuración -->
    </application>
</manifest>
```[span_32](start_span)[span_32](end_span)[span_33](start_span)[span_33](end_span)

> **Regla de Seguridad Crítica:** Toda aplicación nativa debe estar firmada con la **misma Keystore maestra (`kora-release-key.jks`)** que `Kora Admin DB`[span_34](start_span)[span_34](end_span)[span_35](start_span)[span_35](end_span). Si la firma digital difiere, Android denegará el acceso al ContentProvider a nivel de kernel[span_36](start_span)[span_36](end_span).

### Paso 2: Uso del SDK Nativo (`KoraClient`)
Importa el módulo `kora-sdk` y utiliza la API unificada[span_37](start_span)[span_37](end_span):

```kotlin
import org.koradevs.sdk.KoraClient

val kora = KoraClient(context)

// 1. Validar que Kora Admin DB esté instalado
if (!kora.isCoreAvailable()) {
    kora.promptInstallCore() // Abre el release de GitHub para instalar el motor
    return
}

// 2. Registrar el módulo e inyectar tablas
kora.registerModule(
    appName = "Kora Finanzas",
    appVersion = 1,
    ddlSql = """
        CREATE TABLE IF NOT EXISTS mod_fin_movimientos (
            id TEXT PRIMARY KEY,
            monto REAL,
            etiqueta TEXT,
            timestamp INTEGER
        );
    """.trimIndent()
)

// 3. Insertar registros
val valores = ContentValues().apply {
    put("id", UUID.randomUUID().toString())
    put("monto", 25000.0)
    put("etiqueta", "Transporte")
    put("timestamp", System.currentTimeMillis())
}
kora.insert("mod_fin_movimientos", valores)

// 4. Consultar registros
val cursor = kora.query("mod_fin_movimientos", selection = "monto > ?", selectionArgs = arrayOf("10000"))
```[span_38](start_span)[span_38](end_span)

---

## 5. Buenas Prácticas y Reglas de Contribución

1. **Migraciones Aditivas (Prohibido el `DROP TABLE`):** En un modelo Local-First distribuido, destruir tablas borra el progreso de usuarios sin posibilidad de rollback[span_39](start_span)[span_39](end_span)[span_40](start_span)[span_40](end_span). Las migraciones deben usar `ALTER TABLE ADD COLUMN` o esquemas compatibles.
2. **Consultas Sanitizadas:** Jamás concatenes cadenas de texto en SQL (`"SELECT * WHERE id = " + id`). Usa siempre parámetros ligados (`?`) y sentencias preparadas para prevenir inyecciones SQL y fallos de escape[span_41](start_span)[span_41](end_span).
3. **Cero Dependencias Pesadas:** Las micro-apps deben mantenerse ligeras, modulares y con Vanilla JS o empaquetadores compactos (Vite)[span_42](start_span)[span_42](end_span). Evita frameworks con sobrecarga de runtime excesiva.
4. **Flujo de Ramas:**
   * La rama `main` debe ser siempre estable y compatible con compilaciones de GitHub Pages.
   * Toda nueva funcionalidad debe desarrollarse en ramas `feature/<nombre>` y fusionarse mediante Pull Requests revisados.

