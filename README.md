# Home Assistant – Solinteg Peakshaving & Prislogik (PI + Feedforward)

Ansvarsfriskrivning

All styrning av batterier, nätimport och export sker på egen risk.
Verifiera alltid gränsvärden mot huvudsäkringar, nätägarens regler och växelriktarens specifikationer innan du kör skarpt.

Det här projektet innehåller en komplett uppsättning konfiguration för att styra ett Solinteg-batteri via Home Assistant med:

- Peakshaving mot valfri timmedeleffekt
- Prisbaserad laddning/urladdning mot Nordpool
- PI-regulator med feedforward och mjuk rampning av batteriström

Allt är paketerat som ett Home Assistant **package**, redo att läggas direkt i din konfiguration.

---

## Funktionalitet

- **Peakshaving mot timmedel**  
  Styr batteriet så att snittimporten per timme inte överstiger ett konfigurerbart värde (`max_hourly_import`).

- **Prislogik (cheap / super_cheap / expensive / ultra_expensive)**  
  Klassificering av Nordpool-priser per timme utifrån dagens prisbild och justerbara marginaler.

- **Prisbaserad laddning/urladdning**  
  - Ladda batteriet vid låga priser (`cheap`, `super_cheap`)
  - Urladda batteriet vid höga priser (`expensive`, `ultra_expensive`)
  - Valbart exportmål vid pris-urladdning.

- **Arbitragefilter**  
  Prislogiken aktiveras bara om pris-spridningen kommande timmar överstiger en viss tröskel (`battery_price_spread_threshold`).

- **PI + Feedforward**  
  - P + I-del i W/A som styr urladdningsström.
  - Feedforward på positivt effektfel.
  - Mjuk rampning (A/10 s) och maxströmbegränsning.

- **Styrläge-indikator**  
  Sensor som visar aktuellt logikläge:
  - `idle`
  - `peakshaving`
  - `price_charge`
  - `price_discharge`
  - `low_soc_protect`

---

## Förutsättningar

Projektet är byggt för Home Assistant med:

- **Solinteg-integration** (namn kan variera, men motsvarande entiteter krävs):
  - `sensor.solinteg_inverter_meter_active_power`
  - `sensor.solinteg_inverter_meter_active_power_l1`
  - `sensor.solinteg_inverter_meter_active_power_l2`
  - `sensor.solinteg_inverter_meter_active_power_l3`
  - `sensor.solinteg_inverter_battery_soc`
  - `number.solinteg_inverter_battery_discharge_current_limit`
  - `number.solinteg_inverter_battery_charge_current_limit`
  - `number.solinteg_inverter_battery_soc_min_on_grid`
  - `select.solinteg_inverter_working_mode`

- **Nordpool-sensor**
  OBS! Viktigt att köra med HACS version av Nordpool integrationen. Annars kommer inte prislogiken att fungera
  Standardnamn i detta repo:
  - `sensor.nordpool_kwh_se1_sek_2_10_025`  
  Kör du annan region/valuta behöver du justera entity-id i templaten.


Ladda hem följande
- **HACS**
  Home assistant Community store
- **ApexCharts**
  För att visa grafen men olika prislägen
- **Mushroom**
  Snyggare kort till dashboard

---
Ansvarsfriskrivning

All styrning av batterier, nätimport och export sker på egen risk.
Verifiera alltid gränsvärden mot huvudsäkringar, nätägarens regler och växelriktarens specifikationer innan du kör skarpt.


## Struktur

Repo:t är tänkt att se ut ungefär så här:

```text
.
├─ packages/
│  └─ peakshaving_core.yaml      # alla helpers + sensorer + logik
├─ automations/
│  └─ effektstyrning_pi_ff.yaml  # PI+FF-automation (valfritt att separera)
└─ README.md


