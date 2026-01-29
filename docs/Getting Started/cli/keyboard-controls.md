---
title: "Keyboard Controls"
excerpt: "Interactive keyboard shortcuts for the SkySpy CLI"
---

# Keyboard Controls

The SkySpy CLI provides comprehensive keyboard controls for navigating and interacting with the radar display.

## Quick Reference

Press `?` or `h` while running to view the built-in help screen.

## Navigation Controls

| Key | Action |
|-----|--------|
| `Up` / `k` | Select previous aircraft |
| `Down` / `j` | Select next aircraft |
| `+` / `=` | Zoom out (increase range) |
| `-` / `_` | Zoom in (decrease range) |

## Display Toggles

| Key | Action |
|-----|--------|
| `l` / `L` | Toggle aircraft labels |
| `b` / `B` | Toggle aircraft trails |
| `a` / `A` | Toggle ACARS panel |
| `v` / `V` | Toggle VU meters |
| `s` / `S` | Toggle spectrum display |
| `m` / `M` | Toggle military-only filter |
| `g` / `G` | Toggle hide ground aircraft |

## View/Panel Shortcuts

| Key | Action |
|-----|--------|
| `?` / `h` / `H` | Show help screen |
| `t` / `T` | Open theme selector |
| `o` / `O` | Open overlay manager |
| `r` / `R` | Open alert rules editor |
| `/` | Enter search mode |
| `Esc` | Exit current view/mode |
| `q` / `Q` | Quit application |
| `Ctrl+C` | Force quit |

## Export Shortcuts

| Key | Action |
|-----|--------|
| `p` / `P` | Export screenshot (HTML) |
| `e` / `E` | Export aircraft to CSV |
| `Ctrl+E` | Export aircraft to JSON |

## Filter Presets

Quick filter presets using function keys:

| Key | Filter |
|-----|--------|
| `F1` | All aircraft (clear filters) |
| `F2` | Military only |
| `F3` | Emergencies only (7500, 7600, 7700) |
| `F4` | Low altitude (< 5000 ft) |

## Search Mode

Press `/` to enter search mode for advanced filtering.

### Search Mode Controls

| Key | Action |
|-----|--------|
| Type | Enter search query |
| `Backspace` | Delete character |
| `Up` / `Down` | Navigate search results |
| `Enter` | Apply filter |
| `Esc` | Cancel search |

### Search Query Syntax

```
callsign:UAL*        # Callsign pattern
hex:A12345           # ICAO hex code
squawk:7700          # Squawk code
type:B737            # Aircraft type
military:true        # Military aircraft
altitude:>30000      # Above altitude
altitude:<5000       # Below altitude
distance:<50         # Within distance (nm)
```

Multiple terms can be combined:

```
military:true altitude:>30000
callsign:UAL* distance:<100
```

## Theme Selector

Press `t` to open the theme selector.

| Key | Action |
|-----|--------|
| `Up` / `k` | Previous theme |
| `Down` / `j` | Next theme |
| `Enter` / `Space` | Select theme |
| `t` / `Esc` | Close selector |

## Overlay Manager

Press `o` to open the overlay manager.

| Key | Action |
|-----|--------|
| `Up` / `k` | Previous overlay |
| `Down` / `j` | Next overlay |
| `Enter` / `Space` | Toggle overlay visibility |
| `d` / `D` | Delete overlay |
| `o` / `Esc` | Close manager |

## Alert Rules Editor

Press `r` to open the alert rules editor.

| Key | Action |
|-----|--------|
| `Up` / `k` | Previous rule |
| `Down` / `j` | Next rule |
| `Enter` / `Space` | Toggle rule enabled |
| `Esc` | Close editor |

## Configuration Wizard

When running `skyspy configure`:

| Key | Action |
|-----|--------|
| `Tab` / `Down` | Next field |
| `Shift+Tab` / `Up` | Previous field |
| `Space` | Toggle boolean values |
| `Left` / `Right` | Cycle select options |
| `Enter` | Continue to next section |
| `Esc` | Go back to previous section |
| `q` | Quit wizard (welcome/summary only) |

## Help Screen

The built-in help screen (press `?`) displays:

```
SKYSPY RADAR PRO - KEYBOARD SHORTCUTS

NAVIGATION
  Up/k        Select previous aircraft
  Down/j      Select next aircraft
  +/=         Zoom out (increase range)
  -/_         Zoom in (decrease range)

DISPLAY
  l           Toggle labels
  b           Toggle trails
  a           Toggle ACARS panel
  v           Toggle VU meters
  s           Toggle spectrum
  m           Toggle military filter
  g           Toggle hide ground

VIEWS
  ?/h         This help screen
  t           Theme selector
  o           Overlay manager
  r           Alert rules
  /           Search mode

EXPORT
  p           Screenshot (HTML)
  e           Export CSV
  Ctrl+E      Export JSON

FILTERS
  F1          All aircraft
  F2          Military only
  F3          Emergencies
  F4          Low altitude

  q           Quit
  Esc         Close current view
```

## Tips

1. **Vim-style navigation**: Use `j`/`k` for up/down navigation if you prefer vim-style keys.

2. **Quick filtering**: Use F1-F4 for instant filter presets without entering search mode.

3. **Persistent settings**: All toggle states are automatically saved when you quit.

4. **Range options**: Zoom cycles through preset ranges: 25, 50, 100, 200, 400 nm.

5. **Trail history**: Trails show the last few minutes of aircraft movement.
