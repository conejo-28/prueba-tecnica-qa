# Bitácora de Co-Pilotaje e IA (AI_USAGE.md)

## Herramientas utilizadas

- **Claude** — asistente principal durante toda la sesión
               Rol: guía técnica,
              resolución de problemas de entorno
              explicación de conceptos de automatización (selectores, aserciones, sincronización).

## Casos de uso específicos

La IA se utilizó como **acelerador de aprendizaje y resolución de problemas**, no como
generador ciego de código. Dado que esta era mi primera experiencia con automatización móvil,
el asistente cumplió el rol de un ingeniero senior guiando la construcción paso a paso:

- **Configuración autónoma del entorno:** diagnóstico de la conexión adb con el dispositivo
  físico (el S24 Ultra no aparecía en `adb devices`; se identificó que la depuración USB y el
  Auto Blocker de Samsung eran las causas), selección de la versión correcta de Java (17 LTS,
  por incompatibilidad de Maestro con versiones nuevas), e instalación de Maestro Studio.
- **Diseño de selectores estables:** entender la diferencia entre un selector frágil (por
  texto/contenido) y uno estable (por `id`/accesibilidad), y aplicarlo con el inspector de
  Maestro Studio.


## Ejemplos de prompts clave

**Prompt 1 — Diseño de aserción de estado:**
> "Después del login exitoso, ¿qué elemento debo afirmar que demuestre que *de verdad* inicié
> sesión, y no solo que algún elemento existe?"

Valor: este prompt destrabó el concepto central de la prueba. Llevó a razonar que la firma del
estado logueado no estaba en la pantalla de destino (el catálogo se ve igual con o sin sesión),
sino en el cambio del menú de "Log In" a "Log Out" — una aserción que sí discrimina el estado.

**Prompt 2 — Diagnóstico de fallo de sincronización:**
> "Tengo un error después del login: `Failed to tap element: Element not found: Text matching
> regex: Login`, pero el botón Login existe. ¿Por qué Maestro no lo encuentra?"

Valor: llevó a diagnosticar que el teclado virtual tapaba el botón tras escribir la contraseña.
La solución (`hideKeyboard` antes del tap) es un problema de sincronización real, no un error de
selector — distinción importante para el análisis técnico.

## Reflexión técnica

**Impacto en velocidad y calidad:** la IA aceleró significativamente la curva de aprendizaje en
un dominio nuevo (automatización móvil), reduciendo horas de documentación a minutos de
diálogo dirigido. Sin embargo, el valor real no estuvo en generar código, sino en **forzar el
razonamiento**: el asistente insistió repetidamente en verificar en lugar de asumir, y en
entender el *porqué* de cada decisión antes de ejecutarla.

**Corrección de errores y alucinaciones:** hubo casos donde el asistente describió elementos de
UI que no existían en la versión instalada (p. ej. un botón "Inspect" que en realidad no
estaba). Estos se detectaron ejecutando y verificando contra la herramienta real, no confiando
en la descripción. La lección práctica: **la fuente de verdad es la ejecución, no el asistente
ni la memoria.** Cada selector se validó con el inspector real; cada test se confirmó con una
corrida, no con una suposición de que "debería funcionar".

**Juicio de ingeniería aplicado:** las decisiones de alcance (Maestro sobre Appium por el
tiempo y la experiencia; dispositivo físico sobre emulador; recorte de la capa de API a diseño
documentado) fueron decisiones propias, informadas por el diálogo pero sostenibles de forma
independiente.
