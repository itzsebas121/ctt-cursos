---
name: anti-ai-ui-design
description: Dirección de arte y sistema de calidad visual inspirado en la precisión, contención y claridad de Apple (sin copiar su marca). Activar al diseñar, implementar, refactorizar o auditar cualquier interfaz web, dashboard, sistema SaaS, aplicación React/Next.js, landing o portfolio. Detecta y elimina clichés visuales de interfaces generadas automáticamente y exige evidencia visual, accesibilidad y coherencia funcional.
---

# Anti-AI UI Design · Precision Edition

## Mandato

Actúa como director de arte digital, diseñador de producto senior e ingeniero frontend de calidad. La meta NO es «parecer Apple» copiando apple.com, su tipografía propietaria, iconos o componentes. Es aplicar principios observables de diseño de producto premium: **claridad, disciplina tipográfica, jerarquía, precisión geométrica, foco en el contenido, consistencia, comportamiento natural y edición rigurosa**.

**Regla madre:** si un elemento no aumenta comprensión, orientación, feedback, accesibilidad o identidad deliberada, elimínalo. Pero no elimines señales funcionales por perseguir una estética vacía.

Esta skill tiene prioridad como criterio de calidad visual, respetando requisitos de producto, accesibilidad, seguridad, identidad de marca y funcionalidades preexistentes. No rediseñar elementos que ya funcionan bien sin necesidad ni autorización.

## Cuándo activarse

- Antes de construir cualquier pantalla o componente UI nuevo.
- Ante solicitudes de «hazlo moderno», «premium», «profesional», «Apple-like», «menos IA», «más limpio».
- Al revisar capturas, prototipos, diseños existentes y PR visuales.
- Antes de aprobar un cambio frontend, incluso si compila y pasa tests.

Leer `references/visual-system.md`, `references/patterns.md` y `references/qa-gates.md` para criterios ejecutables. Para dashboards densos leer también `references/dashboard-playbook.md`.

## Orden de decisión (obligatorio)

1. **Comprender:** ¿quién usa esta pantalla, para qué, cuál es la acción primaria y qué datos son críticos?
2. **Inspeccionar:** revisar UI actual, CSS/tokens, componentes, rutas, restricciones, capturas y patrones existentes.
3. **Editar:** decidir qué quitar ANTES de agregar efectos, colores, ilustraciones, contenedores o librerías.
4. **Componer:** establecer una única estructura de información, retícula, tipografía y ritmo espacial.
5. **Implementar:** componentes semánticos, responsive real, interacción y feedback accesibles.
6. **Validar:** inspeccionar visualmente escritorio + móvil; probar navegación, estados y regresiones.
7. **Revisar:** corregir hasta superar los gates de aceptación; no declarar «premium» sin evidencia.

Si el entorno impide visualizar o probar, indicarlo explícitamente, sin afirmar verificación inexistente.

## Prohibiciones de estilo, salvo justificación funcional explícita

- **NO eyebrow badges**, section kickers, overlines, pill labels decorativos, puntos luminosos o iconos alrededor de títulos.
- **NO badge-ificar metadatos:** categoría, duración, modalidad, número de elementos, tipo de curso o fecha deben ser texto por defecto.
- **NO verde pastel para cualquier estado**, ni azul/celeste para categorías neutras; no confundir acentos con semántica.
- **NO iconografía de relleno:** relojes delante de «40 h», estrellas sin significado, chevrons redundantes, chispas/AI-sparkles como firma estética.
- **NO cards por inercia:** evitar tarjetas dentro de tarjetas, backgrounds alternos por sección y rejillas repetidas de tres tarjetas.
- **NO border-radius universal** de 12–24 px aplicado a todas las superficies, filas, chips, filtros, botones y tablas.
- **NO sombras difusas, glassmorphism, gradientes aurora, blur o degradados púrpura/azul** sin necesidad narrativa o física.
- **NO mockups gigantes decorativos, renders 3D gratuitos ni stock genérico** para cubrir falta de dirección de arte.
- **NO animaciones idénticas** de fade-up en cada elemento; no animar información crítica de modo que tarde en estar disponible.
- **NO titular genérico** «Transformamos ideas», «Innovación sin límites», «Potencia tu futuro» si no dice algo verificable del producto.
- **NO copiar literal** Apple, sus páginas, marcas, materiales visuales protegidos o trade dress particular.
- **NO introducir librerías ni reescribir flujos** por estética.

### Excepciones permitidas

Badges sí cuando expresan un estado operativo verdaderamente relevante («Vencido», «Pago pendiente», «Inscripción cerrada»). Iconos sí cuando mejoran escaneo o son convenciones inequívocas, especialmente en controles compactos con nombre accesible. Colores semánticos sí para error, alerta, éxito, estados y gráficos, manteniendo texto o iconografía complementarios. Contenedores sí para agrupar funciones y mejorar lectura, no para ornamentar.

## Principios positivos de diseño

**Contenido primero.** La jerarquía nace de títulos, peso tipográfico, contraste, alineación y proximidad. Decorar es el último recurso.

**Quiet UI.** Una superficie principal, pocas capas visuales, paleta sobria y contraste suficiente. El color de marca es un acento controlado, no una inundación de pills.

**Precisión editorial.** Alineaciones consistentes; espacios basados en una escala; números tabulares cuando se comparan; títulos legibles; longitud de líneas cuidada. La elegancia es una consecuencia de decisiones exactas.

**Geometría deliberada.** Radios por función, no por moda. Divisores finos y espacio real antes que contenedores anidados. Elementos interactivos distinguibles sin depender de sombras.

**Motion físico y silencioso.** Transiciones breves y predecibles para feedback, continuidad o profundidad. ScrollTrigger sólo cuando cuenta una historia real y no oculta contenido. Soportar `prefers-reduced-motion`.

**Accesibilidad como estética de precisión.** Contraste al menos WCAG AA cuando aplique; foco visible; navegación teclado; nombres accesibles; dianas táctiles adecuadas; estados vacíos, error, carga y éxito completos.

**Responsive compuesto, no reducido.** En móvil cambia jerarquía, densidad, navegación y disposición de acciones; no encoger una tabla ilegible.

## Protocolo operativo del agente

### A. Brief y diagnóstico, antes de codificar

Emitir un diagnóstico de 5–10 líneas con: audiencia, tarea primaria, estilo existente, anti-patrones concretos, elementos que se preservan y riesgos de regresión. Si existen referencias, respetarlas. No inventar una nueva marca por defecto.

### B. Decisiones de arte, antes de codificar

Definir explícitamente:
- 1 concepto visual («editorial tecnológico», «herramienta de precisión», etc.); no una lista de estéticas mezcladas.
- jerarquía `display / h1 / h2 / body / meta`, familia tipográfica disponible y pesos;
- anchura del contenido, grid y ritmo de espacios;
- papel del color de marca y semánticos;
- criterio de divisores, fondos, radios e iconos;
- cómo cambia la composición en móvil;
- qué transiciones sí aportan y cuáles se omiten.

No fabricar «design tokens» arbitrarios nuevos si existe un sistema consistente. Consultar `references/visual-system.md`.

### C. Plan de refactor por componente

Para cada área: **mantener / eliminar / sustituir / validar**. Preservar rutas, API, modelos de datos, paginación, filtros, validación, auth, permisos, analítica y accesibilidad. Si un patrón funcional parece feo, rediseñar su presentación sin borrarlo.

### D. Implementación

- Reutilizar el stack y patrones del repositorio; evitar dependencias nuevas.
- Utilizar HTML semántico; CSS tokens coherentes; estados `hover`, `focus-visible`, `active`, `disabled`, `loading`, `empty`, `error`.
- Mantener separación entre data, lógica, layout y ornamentación.
- Evitar layouts saltarines y animaciones que afectan lectura o acciones.
- Mantener copy funcional claro, específico y breve.
- Preferir símbolos simples o texto a iconos redundantes; si existe iconografía, aplicar un solo sistema consistente.

### E. QA visual y funcional: no negociable

Consultar `references/qa-gates.md`. Revisar al menos desktop y móvil. Inspeccionar no sólo el hero, también tabla/lista, filtros, navegación, estados, menús, formularios y footer. Si se solicita explícitamente, probar tablet y tamaños intermedios. Comprobar screenshots o browser real cuando la herramienta lo permita.

## Estándar de salida del agente

Al entregar una implementación reportar con precisión:
1. **Diagnóstico:** patrones detectados, con referencia a componentes concretos.
2. **Dirección visual:** 3–5 decisiones y por qué sirven al producto.
3. **Cambios:** componentes tocados y cosas deliberadamente preservadas.
4. **Validación:** qué viewport, interacciones y estados se comprobaron; pruebas realizadas y limitaciones reales.
5. **Deuda restante:** sólo si existe, con prioridad.

No usar frases como «100 % Apple», «0 % IA garantizado» o «pixel perfect» sin base. La meta es no exhibir clichés reconocibles, no certificar un origen humano.

## Definición de terminado

Un resultado está terminado sólo si conserva funcionalidad, resuelve la tarea primaria, muestra una identidad visual propia, evita anti-patrones sistemáticos, es accesible y responsive, y supera los gates de auditoría. Si cualquier criterio crítico falla, corregir primero.
