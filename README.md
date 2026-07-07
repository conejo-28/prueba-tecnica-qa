# Prueba Técnica — Ingeniero de Automatización QA

Suite de pruebas automatizadas para la aplicación móvil **SauceLabs My Demo App** (Android),
construida con **Maestro** sobre un dispositivo físico Samsung Galaxy S24 Ultra.

---

## 1. Estrategia y priorización

Esta prueba se abordó bajo una restricción de tiempo real y explícita. Las decisiones de
alcance se tomaron priorizando por **peso en la rúbrica** y por **riesgo de negocio**, no por
comodidad. El criterio fue: asegurar primero lo de mayor valor y menor incertidumbre, y
documentar con transparencia lo que se recortó.

**Prioridad 1 — Capa móvil (foco principal, mayor peso).** Se cubrió el flujo obligatorio
completo: login exitoso, manejo de credenciales inválidas / cuenta bloqueada, navegación
entre pantallas y confirmación de cambio de estado. Se priorizó la **robustez de los
selectores** y la **verificación real de estado** (aserciones que discriminan el estado
logueado), por encima de la cantidad de casos.

**Prioridad 2 — Capa de API.** Documentada como diseño (ver sección 5), no implementada,
por decisión consciente de alcance ante el tiempo disponible. Se prefirió entregar la capa
móvil sólida y verificable antes que una API a medias y no defendible.

**Fuera de alcance (decisión explícita):** integración con Kafka (bonus), y modularización
avanzada (Screenplay). El tiempo se invirtió en cobertura funcional verificada antes que en
refactorización arquitectónica.

---

## 2. Requisitos previos

- **Java 17 (LTS)** — Temurin/Adoptium. Maestro no es compatible con versiones de Java muy
  nuevas; se usó 17 deliberadamente. `JAVA_HOME` debe apuntar al JDK 17.
- **Android Platform Tools (adb)** en el `PATH`.
- **Maestro** — instalado vía Maestro Studio (Windows).
- **Dispositivo Android** físico o emulador (API 29–34). Esta suite se ejecutó contra un
  Samsung Galaxy S24 Ultra físico vía adb.
- **SauceLabs My Demo App** instalada en el dispositivo
  (`com.saucelabs.mydemoapp.android`), descargada desde el repositorio oficial de SauceLabs.

### Verificación del entorno
```bash
java -version        # debe reportar 17
adb devices          # el dispositivo debe aparecer como "device"
```

---

## 3. Estructura del repositorio

```
prueba-tecnica-qa/
├── .gitignore
├── README.md
├── AI_USAGE.md
└── mobile-tests/
    ├── smokeTest.yaml        
    ├── login.yaml           
    ├── loginFallido.yaml     
    ├── navegacion.yaml       
    └── navegacionPago.yaml  
```

---

## 4. Ejecución de la suite móvil

Cada flujo se ejecuta de forma individual:
```bash
maestro test mobile-tests/login.yaml
maestro test mobile-tests/loginFallido.yaml
maestro test mobile-tests/navegacion.yaml
```

Ejecutar toda la suite:
```bash
maestro test mobile-tests/
```

Generar reporte JUnit:
```bash
maestro test mobile-tests/ --format junit --output report.xml
```

### Decisiones técnicas de la capa móvil
- **`clearState: true`** en cada flujo de login → aislamiento de datos: cada corrida arranca
  deslogueada y limpia, garantizando un test determinista.
- **Selectores estables** → se priorizó `id` (resource-id / accessibilityId) sobre texto para
  íconos, evitando XPath y selectores frágiles por contenido.
- **Aserciones de estado** → cada flujo verifica un cambio de estado real (p. ej.
  `assertVisible: "Log Out"` como firma del estado logueado; contador del carrito como
  confirmación de "producto agregado"), no solo la ejecución de acciones.
- **Sincronización** → Maestro maneja las esperas automáticamente (sin `sleep()`). Se resolvió
  un problema real donde el teclado tapaba el botón de login mediante `hideKeyboard`
  (ver Análisis Técnico en el video).

---

## 5. Capa de API — Diseño (no implementado)

Por restricción de tiempo, la capa de API se documenta y se ejcuta la prueba de 200

- **Herramienta:** Postman + **Newman** (CLI) — construcción visual, ejecución por consola y
  reporte HTML, alineado con el requisito de herramientas open-source.

- **Hallazgo documentado:** Restful-Booker devuelve `200` (no `201`) al crear un recurso y
  requiere autenticación vía header `Cookie` en lugar de `Authorization` — desviaciones de la
  convención REST que un test de contrato debería evidenciar.

---

## 6. Sistema bajo prueba (SUT)

- **App móvil:** SauceLabs My Demo App (`com.saucelabs.mydemoapp.android`). Elegida por tener
  selectores de accesibilidad estables y un usuario válido + uno bloqueado, cubriendo los casos
  positivo y negativo de login.
- **Dispositivo:** Samsung Galaxy S24 Ultra (One UI), físico, vía adb. Se optó por dispositivo
  real sobre emulador por representar el comportamiento en hardware físico.
