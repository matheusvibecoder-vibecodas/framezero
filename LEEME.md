<div align="center">

# framezero

### Tu mejor fotograma es el fotograma cero.

**Crea videos promocionales verticales que detienen el scroll para tu app con HTML, un poco de JavaScript y música generada por código.**
**Sin editor de video, sin After Effects, sin música de stock, sin suscripciones. Salida 4K60 desde una laptop o un sandbox en la nube de 1 GB.**

Escrito para **personas** *y* para **agentes de codificación con IA** (Claude Code, Codex, Cursor, Copilot, Gemini CLI…).

[![smoke](https://github.com/matheusvibecoder-vibecodas/framezero/actions/workflows/smoke.yml/badge.svg)](https://github.com/matheusvibecoder-vibecodas/framezero/actions/workflows/smoke.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-FFC233)](LICENSE)
[![Agent Skill](https://img.shields.io/badge/Agent%20Skill-ready-6A4BFF)](skills/framezero/SKILL.md)
[![AGENTS.md](https://img.shields.io/badge/AGENTS.md-included-13D18C)](AGENTS.md)
[![PRs welcome](https://img.shields.io/badge/PRs-welcome-FF7A52)](CONTRIBUTING.md)

**[Sitio web](https://matheusvibecoder-vibecodas.github.io/framezero/)** · [English](README.md) · [Português](LEIAME.md) · **Español**

<img src="media/estuda-tour-hook.gif" width="230" alt="video del tour de estuda+ — gancho">&nbsp;&nbsp;<img src="media/starter.gif" width="230" alt="plantilla inicial">&nbsp;&nbsp;<img src="media/estuda-tour-challenge.gif" width="230" alt="video del tour de estuda+ — reto diario">

<sub>Izquierda/derecha: videos reales de TikTok/Reels para <b>estuda+</b>, una app de estudios gratuita (el texto en pantalla está en portugués). Centro: la plantilla inicial en inglés de este repositorio.</sub>

</div>

---

## Por qué existe

Soy un desarrollador independiente de Brasil. Creé una app de estudios gratuita y necesitaba videos para TikTok e Instagram. Los editores eran lentos y el resultado se veía genérico. Entonces me pregunté: *¿y si un video fuera simplemente una página web que sabe qué dibujar en cada instante `t`?*

Eso se convirtió en una pequeña pipeline. Con ella hice cinco videos promocionales en dos semanas, cada uno mejor que el anterior. Este repositorio reúne la **pipeline**, las **reglas de storytelling** que aprendí (algunas por las malas) e **instrucciones que un agente de IA puede seguir** para crear un video para *tu* app desde cero.

## Cómo funciona (la versión de 30 segundos)

```
scene.html  ──seek(t)──▶  Playwright screenshots every frame  ──▶  ffmpeg  ──▶  video.mp4 (1080p or 4K60)
soundtrack.py  ──numpy──▶  lo-fi music + whooshes/pops synced to the same timeline  ──▶  muxed in
```

1. **Una escena es un único archivo HTML** (1080×1920) con `const DURATION` y una función pura `seek(t)` que posiciona todo para el instante `t`. No hay temporizadores ni animaciones CSS. Como cada fotograma es determinista, cada uno puede renderizarse y revisarse por separado.
2. **`kit/render.py`** lo abre en Chromium headless, llama a `seek(i/fps)` para cada fotograma, toma capturas y se lo pasa a ffmpeg.
3. **`kit/sound.py`** sintetiza la música y los efectos de sonido a partir de matemáticas, así que el riesgo de copyright es **cero**. Colocas cada sonido en el mismo segundo en el que sucede la animación.
4. **`--stills`** renderiza fotogramas individuales en segundos, para que tú (o tu agente) podáis *mirar* el video antes de gastar minutos en un render completo.

## Inicio rápido

```bash
git clone https://github.com/matheusvibecoder-vibecodas/framezero && cd framezero
pip install -r requirements.txt && python3 -m playwright install chromium   # add --with-deps on fresh Linux

# 1) look at a few frames (2 seconds)
python3 kit/render.py templates/starter.html out/shot --stills 0,3.5,6.6,10.9

# 2) make the soundtrack and the full video (1080p30, ~2 min)
python3 templates/soundtrack.py out/soundtrack.wav
python3 kit/render.py templates/starter.html out/starter.mp4 --audio out/soundtrack.wav

# 3) final quality: 4K60, low-RAM chunked mode
python3 kit/render.py templates/starter.html out/starter-4K.mp4 --scale 2 --fps 60 --crf 16 \
        --audio out/soundtrack.wav --frames-dir /tmp/frames
```

<img src="media/starter-strip.jpg" alt="plantilla inicial — hoja de contactos">

Después copia `templates/` y hazla tuya: cambia el bloque CONFIG, redibuja `appUI()` (o inserta capturas de pantalla) y mantén los tiempos de la banda sonora sincronizados.

## Usarlo con un agente de codificación con IA

Apunta tu agente a este repositorio y dile algo como:

> *"Lee AGENTS.md. Haz un video promocional vertical de 15–25 s para mi app (las capturas están en `./shots`). El gancho debería ser un POV. Música lo-fi. Muéstrame los fotogramas antes de renderizar."*

- [`AGENTS.md`](AGENTS.md) es el manual de operaciones: el flujo de trabajo, las reglas estrictas, el ciclo de QA y los errores ya conocidos. Claude Code, Codex, Cursor, Gemini CLI y Copilot lo reconocen.
- [`skills/framezero/SKILL.md`](skills/framezero/SKILL.md) es el mismo conocimiento como una **Agent Skill** instalable. Copia la carpeta a `~/.claude/skills/` o al directorio de skills de tu agente.
- [`llms.txt`](llms.txt) es un mapa del repositorio para los LLM.

El flujo de trabajo está diseñado en torno a lo que los agentes hacen bien (escribir código determinista) y a lo que hacen mal (juzgar un movimiento que no pueden ver). La respuesta son las **hojas de contactos de fotogramas**: los agentes pueden mirarlas y juzgarlas. La regla más importante: *"la visión apunta, el DOM decide"*. Usa la captura para detectar un problema y luego confírmalo midiendo el elemento.

## El playbook (qué hace que la gente deje de hacer scroll)

| Regla | Por qué |
|---|---|
| **El fotograma 0 es la miniatura y el gancho.** Todo el texto del gancho está completamente visible en `t=0`. Nada de fundidos de entrada ni intros en negro. | La gente decide en menos de un segundo, y el primer fotograma es lo que se muestra en la cuadrícula. |
| **Contexto para el segundo 3** (*"5 cosas que hace"*), y después una **interrupción de patrón cada 2–4 s**. | Usa un color nuevo, un zoom, una etiqueta o un sonido para reiniciar la atención. |
| **Muestra el producto real.** Usa pantallas reales dentro de un teléfono, con movimientos de cámara y etiquetas. | La gente confía más en la interfaz que en los adjetivos. |
| **Guarda lo mejor para el final, y dilo** (*"la última es la mejor"*). | Un bucle abierto hace que sigan mirando. |
| **Solo texto, grande, dentro de la zona segura** (x 80–940, y 220–1480 en 1080×1920). | La mayoría mira en silencio, y la interfaz de TikTok/Reels cubre los bordes. |
| **Cada evento visual tiene un sonido.** Whoosh antes de los cortes, impacto en los golpes, pop en las etiquetas, moneda en las recompensas. | El sonido hace que el movimiento se sienta físico, incluso a volumen bajo. |
| **Bucle perfecto.** Los últimos 0.4 s barren de vuelta al fotograma 0. | Las repeticiones cuentan. |
| **Termina con una CTA acorde a tu etapa.** Antes del lanzamiento, *"muy pronto · síguenos para enterarte del lanzamiento"*. | Un "link en la bio" que no lleva a nada destruye la confianza. |

Guía completa: [`docs/`](docs/).

## Muestra: la serie estuda+

Todos se hicieron con este kit, y todos tienen sonido y una versión maestra en 4K60.

**#4 Tour de la app** (39 s), *"POV: encontré la app que estudia conmigo"*. Muestra 5 funciones sobre pantallas reales, con una barra de progreso estilo Stories y una pista lo-fi. → [`examples/estuda-tour`](examples/estuda-tour)

<img src="media/estuda-tour-strip.jpg" alt="tour de estuda+ — hoja de contactos">

**#3 Quiz del reto diario** (33.5 s), *"Apuesto a que no aciertas las 3"*. Está construido para generar comentarios.

<img src="media/estuda-challenge-strip.jpg" alt="reto de estuda+ — hoja de contactos">

**#2 Tutor con IA** (25 s), *"¿Atascado con la tarea a las 11:47 pm?"*

Lo que aprendimos en el camino, incluidos los fracasos, está en [`docs/08-case-study-estuda.md`](docs/08-case-study-estuda.md).

## ¿Hiciste algo con framezero?

Abre un issue con la plantilla **[Show your video](https://github.com/matheusvibecoder-vibecodas/framezero/issues/new?template=showcase.yml)**. Los mejores se destacan aquí. ¿Nuevo en el código? Prueba con un [good first issue](https://github.com/matheusvibecoder-vibecodas/framezero/labels/good%20first%20issue) o lee [CONTRIBUTING.md](CONTRIBUTING.md).

¿Quieres ver un video *sobre* framezero? Está en [`examples/framezero-promo`](examples/framezero-promo/), y el MP4 está adjunto en la [versión v0.1.0](https://github.com/matheusvibecoder-vibecodas/framezero/releases/tag/v0.1.0).

## Mapa del repositorio

```
kit/            the engine — render.py, assemble.sh (4K in 1 GB RAM), motion.js (helpers), sound.py (synth), base.css (Inter), icons.svg
templates/      starter.html + soundtrack.py — a complete 12.6 s English promo to copy
examples/       estuda-tour/ — the real 39 s tour video (scene + screenshots + soundtrack)
docs/           01 method · 02 story & hooks · 03 animation · 04 phone & camera · 05 sound · 06 rendering · 07 publishing · 08 case study · 09 QA checklist
skills/         framezero/SKILL.md — Agent Skill
AGENTS.md       operating manual for AI agents
```

## Preguntas frecuentes

**¿Por qué no Remotion / After Effects / CapCut?** Remotion está genial, pero necesita React y un bundler. After Effects y CapCut son manuales y difíciles de conducir para un agente. framezero es HTML + JS + Python sin más, sin nada que configurar, y tu agente ya conoce los tres.

**¿Necesito una GPU?** No. El renderizado es solo con CPU. Un video 4K60 de 39 s tarda unos 23 minutos en un sandbox diminuto de 1 vCPU, y mucho menos en una laptop.

**¿Puedo usar en su lugar los sonidos en tendencia de TikTok?** Sí. Renderiza con la pista generada por código como opción segura por defecto, y al publicar elige un sonido en tendencia dentro de la app y baja el audio original.

**¿La música es realmente libre de derechos?** Sí. La genera `kit/sound.py` a partir de ondas sinusoidales y ruido, y no se samplea nada.

## Licencia

Código y documentación: [MIT](LICENSE). La fuente Inter incluida está bajo la SIL Open Font License. El nombre, el logotipo y las capturas de la app **estuda+** en `examples/` y `media/` son © Matheus Patricio Pessoa Santana. Se incluyen como referencia y **no** están cubiertos por la licencia MIT, así que no los reutilices en tus propios videos. Consulta [NOTICE.md](NOTICE.md).

<div align="center"><sub>Hecho en Brasil 🇧🇷 por <a href="https://github.com/matheusvibecoder-vibecodas">@matheusvibecoder</a>. Si esto te ayudó a publicar un video, una ⭐ ayuda mucho.</sub></div>
