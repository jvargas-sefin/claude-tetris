---
name: clima-local
description: Obtiene el clima actual y el pronóstico de los próximos días para la ubicación local del usuario (o una ciudad indicada). Úsala cuando el usuario pregunte por el clima, la temperatura, la lluvia o el pronóstico.
---

# Clima local

Obtén el clima usando servicios públicos gratuitos que no requieren API key. Responde siempre en español.

## Pasos

1. **Determina la ubicación**
   - Si el usuario indica una ciudad (ej. `/clima-local Tegucigalpa`), úsala.
   - Si no, deja que el servicio detecte la ubicación por IP (no envíes datos personales del usuario).

2. **Consulta el clima** con `curl` (disponible en Windows 11 y Bash):

   ```bash
   # Ubicación detectada por IP
   curl -s "https://wttr.in/?format=j1&lang=es"

   # Ciudad específica (reemplaza espacios por +)
   curl -s "https://wttr.in/Tegucigalpa?format=j1&lang=es"
   ```

   Campos útiles del JSON:
   - `current_condition[0]`: `temp_C`, `FeelsLikeC`, `humidity`, `windspeedKmph`, `precipMM`, `lang_es[0].value` (descripción)
   - `nearest_area[0]`: `areaName[0].value`, `country[0].value` (ubicación resuelta)
   - `weather[0..2]`: `date`, `maxtempC`, `mintempC`, y `hourly[*].chanceofrain`

3. **Alternativa** si `wttr.in` falla o hay timeout: Open-Meteo (requiere latitud/longitud).
   - Geocodificar: `curl -s "https://geocoding-api.open-meteo.com/v1/search?name=CIUDAD&count=1&language=es"`
   - Clima: `curl -s "https://api.open-meteo.com/v1/forecast?latitude=LAT&longitude=LON&current=temperature_2m,relative_humidity_2m,apparent_temperature,precipitation,wind_speed_10m&daily=temperature_2m_max,temperature_2m_min,precipitation_probability_max&timezone=auto"`

4. **Presenta el resultado** de forma breve, en español y en unidades métricas (°C, km/h, mm):

   ```
   📍 <Ciudad, País>
   Ahora: <temp>°C (sensación <feels>°C) — <descripción>
   Humedad: <h>% · Viento: <v> km/h · Lluvia: <mm> mm

   Próximos días:
   - <fecha>: mín <x>°C / máx <y>°C, prob. lluvia <z>%
   ```

## Notas

- Si ambas fuentes fallan (sin red), informa el error y no inventes datos.
- Aclara la ubicación usada; si fue detectada por IP, puede ser aproximada y el usuario puede indicar otra ciudad.
- Esta skill es solo de consulta: no modifica archivos del proyecto.
