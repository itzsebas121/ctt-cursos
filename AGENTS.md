# Reglas del Proyecto CTT FISEI - UI y Arquitectura

## 1. Prohibición Estricta de Cápsulas y Eyebrow Badges Decorativos
- **NUNCA generar cápsulas / pill badges / eyebrow overlines decorativos sobre títulos de sección**, tales como:
  - `CAPACITACIÓN PROFESIONAL`
  - `SOBRE NOSOTROS`
  - `ESTAMOS PARA AYUDARTE`
  - O cualquier pastilla con `rounded-full`, chispas (`SparklesIcon`), o fondos translúcidos colocados arriba de un `<h2>` o `<h3>`.
- **Los encabezados deben ser directos y limpios**: La jerarquía visual nace del tamaño tipográfico, el contraste de color y el espaciado, sin adornos superficiales de plantillas automáticas.

## 2. Animaciones con GSAP y ScrollTrigger
- **Cero desaparición de elementos**: Nunca animar elementos hacia `opacity: 0` al salir del viewport (`toggleActions: "play reverse play reverse"` queda prohibido).
- **Siempre usar `clearProps: "all"` y `once: true`**: Garantizar que una vez ejecutada la animación de entrada, las propiedades CSS inline se limpien y el elemento permanezca 100% visible sin huecos en blanco al regresar el scroll.
- **Transiciones entre páginas**: Siempre resetear el scroll al tope (`top: 0, left: 0`) al cambiar de ruta (`ScrollToTop`).

## 3. Identidad de Marca y Contraste
- Azul institucional principal: `#00345e`.
- Modo claro y modo oscuro consistentes, sin textos blancos sobre fondo blanco ni textos oscuros sobre fondo oscuro.
- Fotografías reales y auténticas de campus universitario y laboratorios, evitando stock genérico.

