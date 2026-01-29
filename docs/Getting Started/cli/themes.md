---
title: "Themes"
excerpt: "Available color themes for the SkySpy CLI"
---

# Themes

The SkySpy CLI includes 10 carefully designed color themes to customize your radar display experience.

## Available Themes

| Theme | Name | Description |
|-------|------|-------------|
| `classic` | Classic Green | Traditional green phosphor display |
| `amber` | Amber | Vintage amber monochrome display |
| `ice` | Blue Ice | Cold blue tactical display |
| `cyberpunk` | Cyberpunk | Neon futuristic display |
| `military` | Military | Tactical military display |
| `high_contrast` | High Contrast | Maximum visibility white display |
| `phosphor` | Phosphor | Realistic CRT phosphor glow |
| `sunset` | Sunset | Warm orange sunset tones |
| `matrix` | Matrix | Matrix digital rain inspired |
| `ocean` | Ocean | Deep blue oceanic display |

## Theme Previews

### Classic Green

The default theme featuring traditional green phosphor colors reminiscent of vintage radar displays.

- **Primary**: Green (#28)
- **Bright**: Bright green (#46)
- **Status colors**: Yellow warnings, red emergencies, magenta military

### Amber

A warm amber monochrome theme inspired by vintage computer terminals.

- **Primary**: Amber/Yellow (#178)
- **Bright**: Bright yellow (#226)
- **Great for**: Night use, reducing eye strain

### Blue Ice

A cold blue tactical theme for a modern military aesthetic.

- **Primary**: Blue (#21)
- **Bright**: Cyan (#51)
- **Great for**: Professional appearance, tactical operations

### Cyberpunk

A vibrant neon theme with magenta and cyan highlights.

- **Primary**: Magenta (#165)
- **Secondary**: Cyan (#51)
- **Great for**: Futuristic aesthetic, high visibility

### Military

A tactical green theme with yellow target highlights.

- **Primary**: Green (#28)
- **Targets**: Yellow (#226)
- **Great for**: Military simulation, tactical displays

### High Contrast

Maximum visibility white-on-black for accessibility.

- **Primary**: White (#231)
- **Dim**: Grey (#249)
- **Great for**: Accessibility, bright environments

### Phosphor

Realistic CRT phosphor glow effect with smooth gradients.

- **Primary**: Phosphor green (#33ff33)
- **Effect**: Authentic CRT look
- **Great for**: Retro enthusiasts, immersive experience

### Sunset

Warm orange and red tones for a sunset-inspired look.

- **Primary**: Orange (#208)
- **Bright**: Red (#196)
- **Great for**: Evening use, warm atmosphere

### Matrix

Inspired by the digital rain effect from The Matrix.

- **Primary**: Matrix green (#00ff00)
- **Background**: Pure black
- **Great for**: Movie fans, hacker aesthetic

### Ocean

Deep blue oceanic colors for a calm, professional look.

- **Primary**: Ocean blue (#0066cc)
- **Secondary**: Teal (#00cccc)
- **Great for**: Maritime operations, calm atmosphere

## Changing Themes

### During Runtime

Press `t` to open the theme selector, then use arrow keys to navigate and Enter to select.

### Command Line

```bash
skyspy --theme cyberpunk
skyspy --theme military
```

### Configuration File

Edit `~/.config/skyspy/settings.json`:

```json
{
  "display": {
    "theme": "cyberpunk"
  }
}
```

### List Available Themes

```bash
skyspy --list-themes
```

Output:

```
Available Themes:
  classic         Classic Green   - Traditional green phosphor display
  amber           Amber           - Vintage amber monochrome display
  ice             Blue Ice        - Cold blue tactical display
  cyberpunk       Cyberpunk       - Neon futuristic display
  military        Military        - Tactical military display
  high_contrast   High Contrast   - Maximum visibility white display
  phosphor        Phosphor        - Realistic CRT phosphor glow
  sunset          Sunset          - Warm orange sunset tones
  matrix          Matrix          - Matrix digital rain inspired
  ocean           Ocean           - Deep blue oceanic display
```

## Theme Color Elements

Each theme defines colors for the following UI elements:

| Element | Description |
|---------|-------------|
| Primary | Main interface color |
| Primary Bright | Highlighted/active elements |
| Primary Dim | Subtle/background elements |
| Secondary | Accent color |
| Success | Positive status indicators |
| Warning | Warning indicators |
| Error | Error/emergency indicators |
| Info | Information indicators |
| Military | Military aircraft highlight |
| Emergency | Emergency squawk highlight |
| Selected | Currently selected item |
| Border | Panel borders |
| Text | Normal text |
| Text Dim | Secondary/dimmed text |
| Radar Sweep | Rotating sweep line |
| Radar Ring | Range ring lines |
| Radar Target | Aircraft blips |
| Radar Trail | Aircraft trails |

## Tips

1. **Night flying**: Use `amber` or `phosphor` for reduced eye strain in dark environments.

2. **Accessibility**: Use `high_contrast` for maximum readability.

3. **Presentations**: Use `cyberpunk` or `ocean` for visually appealing demonstrations.

4. **Military simulation**: Use `military` or `ice` for tactical aesthetics.

5. **Terminal compatibility**: All themes work with 256-color terminals. For best results with `phosphor`, `matrix`, and `ocean` themes, use a terminal that supports true color (24-bit).
