# Skills instalables

Carpetas con formato `SKILL.md` listas para copiar a tu directorio de skills.

## directing-stickman-videos

Convierte un texto, artículo, notas o simplemente un tema en un paquete de
producción para un video de "stickman" (monigote de líneas) de ~1 minuto:
guion en inglés (130–150 palabras), storyboard de 6 escenas de ~10s y 6 prompts
autocontenidos para Gemini Omni Flash.

- Origen: https://github.com/kaomei/stickman-video-director (MIT)
- Publicada originalmente como skill de Codex; el formato `SKILL.md` es
  compatible con Claude Code sin cambios.

### Instalación

Claude Code (global, para todos los proyectos):

```bash
cp -R skills/directing-stickman-videos ~/.claude/skills/
```

Claude Code (solo este proyecto):

```bash
cp -R skills/directing-stickman-videos .claude/skills/
```

Codex:

```bash
cp -R skills/directing-stickman-videos "${CODEX_HOME:-$HOME/.codex}/skills/"
```

Reiniciá el CLI para que la detecte.

### Uso

Invocala con `/directing-stickman-videos` y pegá el material. Pide siempre
tres cosas antes de planificar: material fuente, relación de aspecto
(`16:9`, `9:16` o `1:1`) y tema (claro = fondo blanco/figura negra,
oscuro = fondo negro/figura blanca). Después presenta una propuesta de
dirección de 6 escenas y **se detiene** hasta que la apruebes; recién ahí
genera los 6 prompts finales.
