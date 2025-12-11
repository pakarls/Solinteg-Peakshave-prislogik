# Installation – Solinteg Peakshave & Prislogik (Home Assistant)

Detta projekt innehåller logik för:

- Peakshaving mot valbar timmedeleffekt  
- Prisbaserad laddning/urladdning mot Nordpool  
- PI-regulator med feedforward för batteriurladdning  
- Styrning av Solinteg-växelriktare via Home Assistant  
<img width="1059" height="1020" alt="image" src="https://github.com/user-attachments/assets/d72731b3-9951-41c7-8635-3a695adc7222" />

---

## Ansvarsfriskrivning

All styrning av batterier, nätimport och export sker på **egen risk**.

Verifiera alltid gränsvärden mot:

- Huvudsäkringar  
- Nätägarens regler och villkor  
- Växelriktarens och batteriets specifikationer  

Detta ska ses som **exempelkonfiguration**, inte som en certifierad färdig produkt.

---

## 1. Förutsättningar

Du behöver:

- Home Assistant (YAML-konfiguration aktiv)  
- Solinteg integrerad i Home Assistant  
- Nordpool-integrationen installerad  

I exemplen används följande entiteter (byt i YAML om dina heter något annat):

### Solinteg (exempel)

- `sensor.solinteg_inverter_meter_active_power`
- `sensor.solinteg_inverter_meter_active_power_l1`
- `sensor.solinteg_inverter_meter_active_power_l2`
- `sensor.solinteg_inverter_meter_active_power_l3`
- `sensor.solinteg_inverter_battery_soc`
- `sensor.solinteg_inverter_battery_power`
- `number.solinteg_inverter_battery_discharge_current_limit`
- `number.solinteg_inverter_battery_charge_current_limit`
- `number.solinteg_inverter_battery_soc_min_on_grid`
- `select.solinteg_inverter_working_mode`

### Nordpool (exempel)

- `sensor.nordpool_kwh_se1_sek_2_10_025`

Om dina entiteter heter något annat behöver du justera entity-id i YAML-filerna.

---

## 2. Aktivera packages i Home Assistant

Öppna `configuration.yaml` och säkerställ att du har:

    homeassistant:
      packages: !include_dir_named packages

Skapa sedan katalogen (om den inte redan finns):

    config/packages/

Starta om Home Assistant om detta är första gången du aktiverar `packages`.

---

## 3. Lägg in package-filen

Kopiera filen från repo:t:

    packages/peakshaving_core.yaml

till din Home Assistant-konfiguration:

    config/packages/peakshaving_core.yaml

> Viktigt: Om du redan har `input_number`, `sensor`, `template`, `utility_meter` osv med samma namn i andra YAML-filer (t.ex. `sensor.yaml`, `template.yaml`, `input_number.yaml`) måste du ta bort eller kommentera dessa för att undvika dubbletter.

---

## 4. Lägg in automationerna

Det finns två huvudautomationer:

- `Effektstyrning – PI + Feedforward (Premium)`  
- `Batteri – nollställ ström vid idle`  

### Alternativ A – `automations.yaml`

Om du använder standardläget där alla automationer ligger i `automations.yaml`:

Öppna:

    config/automations.yaml

Klistra in automationerna från repo:t längst ned i filen. Filen ska vara en lista av automationer, t.ex.:

    - id: effektstyrning_pi_ff_premium
      alias: Effektstyrning – PI + Feedforward (Premium)
      ...
    - id: batteri_nollstall_vid_idle
      alias: Batteri – nollställ ström vid idle
      ...

### Alternativ B – automations-katalog

Om du använder:

    automation: !include_dir_merge_list automations

kan du istället:

1. Skapa filen:

       config/automations/effektstyrning_pi_ff.yaml

2. Klistra in båda automationerna där.

---

## 5. Starta om Home Assistant

När package + automationer är på plats:

- Gå till **Inställningar → System → Starta om**, eller  
- Använd **Utvecklarverktyg → YAML → Kontrollera konfiguration → Starta om**.

---

## 6. Konfigurera parametrar (Helpers)

Gå till:

> **Inställningar → Enheter & tjänster → Hjälpare**

Justera framför allt följande `input_number` och relaterade värden:

### Effekt / begränsningar

- `input_number.max_hourly_import`  
  Max tillåtet timmedel (W) för nätimport, t.ex. `4000–5000`.

- `input_number.battery_w_per_amp`  
  Ungefärlig effekt per ampere för ditt batteri, t.ex. `50` (W/A).

- `input_number.max_discharge_current`  
  Max urladdningsström batteriet får gå upp till (säkerhetslimit).

### SoC-gränser

- `input_number.battery_soc_min_peakshave`  
  Min SoC där peakshaving får urladda (t.ex. `15–30 %`).

- `number.solinteg_inverter_battery_soc_min_on_grid`  
  Absolut min SoC på nät (pris-urladdning), styr Solinteg direkt.

- `input_number.battery_soc_max_charge`  
  Max SoC för prisladdning (t.ex. `85–95 %`).

- `input_number.battery_capacity_kwh`  
  Batteriets nominella kapacitet i kWh.

### Pris- och arbitrageparametrar

- `input_number.nordpool_cheap_margin_percent`
- `input_number.nordpool_supercheap_offset_ore`
- `input_number.nordpool_expensive_margin_percent`
- `input_number.nordpool_ultraexpensive_offset_ore`
- `input_number.battery_price_spread_threshold`  
  Min spread (SEK/kWh) kommande timmar för att prislogiken ska anses “värd att köra”.

### Ladd- och urladdningsströmmar

- `input_number.battery_charge_current_cheap`
- `input_number.battery_charge_current_supercheap`
- `input_number.battery_discharge_current_expensive`
- `input_number.battery_discharge_current_ultraexpensive`
- `input_number.export_allow_watt`  
  Exportmål vid pris-urladdning.

### PI-parametrar

- `input_number.pid_kp`
- `input_number.pid_ki`
- `input_number.pid_kff`
- `input_number.pid_ramp_step`

Startvärdena i YAML:en är rimliga som baseline, men bör fintrimmas per anläggning.

---

## 7. Verifiera att allt fungerar

Gå till **Utvecklarverktyg → Status** och kontrollera att följande sensorer lever och ser rimliga ut:

- `sensor.energy_from_power_total`  
  Ökar i kWh när du har last.

- `sensor.energy_current_hour`  
  Tickar upp under innevarande timme, nollställs vid hel timme.

- `sensor.medeleffekt_innevarande_timme`  
  Visar snitt-effekt (W) för aktuell timme.

- `sensor.allowed_import_power_this_moment`  
  Visar hur mycket du “får” importera just nu för att hålla timmedel under gränsen.

- `sensor.import_error_w`  
  Positivt när du ligger över allowed import, nära 0 eller negativt när du ligger under.

- `sensor.safe_extra_import_power`  
  Positiv marginal när du har buffert upp till timgränsen.

- `sensor.nordpool_prislage`  
  Ett av: `super_cheap`, `cheap`, `neutral`, `expensive`, `ultra_expensive`.

- `sensor.nordpool_arbitrage_spread_24h_v3`  
  Spread (SEK/kWh) mellan framtida min/max-priser.

- `binary_sensor.battery_price_arbitrage_ok`  
  `on` när spreaden är över din satta tröskel.

- `sensor.batteri_styrlage`  
  Visar aktuellt läge: `idle`, `peakshaving`, `price_charge`, `price_discharge`, `low_soc_protect`.

---

## 8. Rekommenderad dashboard

För att följa beteendet i realtid är dessa entiteter vettiga att lägga på en Lovelace-vy:

- `sensor.medeleffekt_innevarande_timme`
- `sensor.allowed_import_power_this_moment`
- `sensor.import_error_w`
- `sensor.safe_extra_import_power`
- `sensor.nordpool_prislage`
- `sensor.nordpool_arbitrage_spread_24h_v3`
- `binary_sensor.battery_price_arbitrage_ok`
- `sensor.batteri_styrlage`
- Batteriets SoC, batteri-effekt samt charge/discharge current limits  

När allt detta ser rimligt ut kan du börja fintrimma parametrar och optimera beteendet efter din egen anläggning.
