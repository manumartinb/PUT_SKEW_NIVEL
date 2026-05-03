# PUT SKEW NIVEL Dashboard

Pagina web con grafico interactivo + evidencia estadistica del **PUT SKEW NIVEL**
(percentil expanding del spread IV puts 25-delta vs ATM, DTE 60, snapshot 10:30 ET).

URL publica: https://manumartinb.github.io/PUT_SKEW_NIVEL_BATMAN_LT/

## Que muestra

- Linea principal: **skew_25d_vs50_pct_expanding** (percentil 0-100)
- Bandas coloreadas: FAVORABLE (>=80, Allantis/Batman), NEUTRAL (20-80), ADVERSO (<=20)
  - **OJO BWB**: convencion invertida. BWB FAV = pct <=20.
- Selector de rango: 30D / 90D / 1A / 3A / All
- Seccion de evidencia (8 cards) bajo el grafico:
  1. Concepto + bandas + inversion BWB
  2. Metodologia (Allantis MT, filtro |SPX|<=3%)
  3. Spearman r vs PnL Allantis por horizonte d001-d049
  4. Deciles D1-D10 + spread D10-D1 por horizonte
  5. Year stability 2019-2025
  6. Regime split (FAV/NEU/ADV) en d030 y d050
  7. Window-forward conditioning (HIGH/LOW PUT SKEW durante el trade)
  8. Cross-strategy: convencion estandar (Allantis/Batman) vs invertida (BWB)

## Pipeline

Actualizacion automatica diaria via `V0.[PERMA] MASTER_DAILY_PIPELINE.py` (Step 4):

```
V18 -> V8.0 (genera SKEW_PUT_ENRICHED.csv) -> Step 3 LIBERATION -> Step 4 PUT SKEW
```

`update_dashboard.py` lee SKEW_PUT_ENRICHED.csv filtrado a DTE=60/snapshot=10:30/side=PUT,
regenera `data.json` y hace push a este repo. GitHub Pages sirve el HTML estatico.

## Fuente de datos

`Skew/SKEW_PUT_ENRICHED.csv` (output de V8.0 SKEW PIPELINE), columna
`skew_25d_vs50_pct_expanding`.

## Seccion de evidencia estadistica

La evidencia es **estatica** (no se regenera con V0 diario). Para regen manual:

```
python "C:\Users\Administrator\Desktop\PUT_SKEW_NIVEL_DASHBOARD\generate_evidence.py" --push
```

Lo que hace `generate_evidence.py`:

1. Lee Allantis MT dataset (`[MAIN RANKEO MT]_combined_ALLANTIS_ALLDAYS.csv`).
2. Joinea PUT SKEW NIVEL desde `SKEW_PUT_ENRICHED.csv` (DTE=60) por `trade_date`.
3. Aplica filtro |SPX_chg_pct_d030|<=3% (cleanest signal per documentacion).
4. Calcula Spearman + bootstrap CI95, deciles, year stability, regime split.
5. Genera 5 PNGs propios (matplotlib dark theme matching dashboard).
6. Re-renderiza window-forward chart desde `Skew/ANALISIS/02_PUT_SKEW_NIVEL/PUT_SKEW_NIVEL_window_forward_results.csv`.
7. Lee `Skew/ANALISIS/02_PUT_SKEW_NIVEL/PF_DXXX_P20_P80/put_pct_ge80_pf_d010_step10.csv` para tabla cross-strategy.
8. Volca `evidence/evidence.json` con metricas + tablas HTML inline.
9. Si `--push`: hace `git pull --rebase`, commit y push usando `GH_PUT_SKEW_TOKEN`.

**No correr entre 12:25 y 12:45 Madrid** — coincide con la ventana del push diario de V0
Steps 3 y 4 y podria provocar conflictos de rebase.

Sin `--push`: solo genera locales (util para iterar diseno antes de publicar).