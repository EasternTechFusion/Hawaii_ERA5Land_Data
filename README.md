# Hawaiian_VPD_Data

Hourly meteorological data for Hawaiʻi from 1950 to 2026, based on the ERA5-Land reanalysis, together with derived **Vapor Pressure Deficit (VPD)** and related variables.

## Files

| File | Rows | Description |
|------|------|-------------|
| `1950_2026_Data.csv` | 673,259 | Raw hourly ERA5-Land variables, 1950-01-01 01:00 to 2026-10-22 16:00 |
| `Processed_1950_2026_Data.csv` | 673,259 | Same period with unit conversions and derived variables (VPD, relative humidity) |
| `era5_hawaii_1950.csv` | 8,759 | Raw data for the year 1950 only (a subset of `1950_2026_Data.csv`) |

```
Hawaliian_VPD_Data/
├── Raw_Data/          # 1950_2026_Data.csv, era5_hawaii_1950.csv
├── Processed_Data/    # Processed_1950_2026_Data.csv
└── README.md
```

<!-- TODO: confirm which file sits in which folder; add the study location (name, lat/lon) and the timestamp time zone (e.g. UTC) -->

## Data Source

| Property | Value |
|----------|-------|
| Dataset | ERA5-Land Hourly (ECMWF) |
| Access | Google Earth Engine |
| Native pixel size | 11,132 m (about 0.1°) |
| Temporal resolution | Hourly |

Values are grid-cell averages from a reanalysis model, not direct station observations.

## Raw Data Columns (`1950_2026_Data.csv`, `era5_hawaii_1950.csv`)

| Column | Unit | Description |
|--------|------|-------------|
| `timestamp` | datetime | Hourly timestamp (`YYYY-MM-DD HH:MM:SS`) |
| `temperature_2m` | K | Temperature of the air at 2 m above the surface of land, sea or inland waters, interpolated between the lowest model level and the Earth's surface |
| `dewpoint_temperature_2m` | K | Temperature to which the air at 2 m would have to be cooled for saturation to occur; a measure of air humidity. Together with temperature and pressure it can be used to calculate relative humidity |
| `surface_solar_radiation_downwards_hourly` | J/m² | Solar (shortwave) radiation reaching the Earth's surface, both direct and diffuse; roughly what a pyranometer at the surface would measure. Hourly values disaggregated from the original accumulated field. Divide by 3,600 s to obtain W/m² |
| `surface_pressure` | Pa | Pressure of the atmosphere on the land, sea or inland-water surface, i.e. the weight of the air column above that point (1 hPa = 100 Pa) |
| `total_precipitation_hourly` | m | Liquid and frozen water (rain and snow) falling to the surface, the sum of large-scale and convective precipitation; excludes fog, dew and precipitation that evaporates before landing. Expressed as the depth the water would have if spread evenly over the grid cell; hourly values disaggregated from the accumulated field |
| `u_component_of_wind_10m` | m/s | Eastward component of the 10 m wind: horizontal speed of air moving toward the east (positive = toward the east) |
| `v_component_of_wind_10m` | m/s | Northward component of the 10 m wind: horizontal speed of air moving toward the north (positive = toward the north). Combine with the U component to get horizontal wind speed and direction |
| `volumetric_soil_water_layer_1` | m³/m³ | Volume of water in soil layer 1 (0–7 cm) of the ECMWF Integrated Forecasting System, a volume fraction that depends on soil texture, soil depth and the underlying groundwater level |

Horizontal wind speed can be obtained as `sqrt(u² + v²)`. Values are grid-cell averages, so they may differ from local point observations affected by terrain, vegetation and buildings.

## Processed Data Columns (`Processed_1950_2026_Data.csv`)

| Column | Unit | Description |
|--------|------|-------------|
| `timestamp` | datetime | Hourly timestamp |
| `temperature_2m_c` | °C | `temperature_2m` − 273.15 |
| `dewpoint_temperature_2m_c` | °C | `dewpoint_temperature_2m` − 273.15 |
| `relative_humidity` | % | Relative humidity from T and Td (see below), capped at 100 |
| `vpd_kpa` | kPa | Vapor pressure deficit (see below) |
| `light_intensity_wm2` | W/m² | `surface_solar_radiation_downwards_hourly` ÷ 3600; negative values set to 0 |
| `surface_pressure` | Pa | Pressure of the atmosphere on the land, sea or inland-water surface, i.e. the weight of the air column above that point (1 hPa = 100 Pa). Same values as the raw data |
| `total_precipitation_hourly` | m | Liquid and frozen water (rain and snow) falling to the surface, the sum of large-scale and convective precipitation; excludes fog, dew and precipitation that evaporates before landing. Depth of water if spread evenly over the grid cell, hourly values. Same values as the raw data |
| `u_component_of_wind_10m` | m/s | Eastward component of the 10 m wind: horizontal speed of air moving toward the east (positive = toward the east). Same values as the raw data |
| `v_component_of_wind_10m` | m/s | Northward component of the 10 m wind: horizontal speed of air moving toward the north (positive = toward the north). Same values as the raw data |
| `volumetric_soil_water_layer_1` | m³/m³ | Volume of water in soil layer 1 (0–7 cm), a volume fraction that depends on soil texture, soil depth and the underlying groundwater level. Same values as the raw data |

### How the derived variables are computed

Saturation vapor pressure (kPa), Tetens equation, with T in °C:

```
es(T) = 0.6108 × exp(17.27 × T / (T + 237.3))
```

Relative humidity and VPD:

```
RH  = 100 × es(Td) / es(T)      [%]
VPD = es(T) − es(Td)            [kPa]
```

where `T` is `temperature_2m_c` and `Td` is `dewpoint_temperature_2m_c`. The `vpd_kpa` column reproduces this formula to floating-point precision.
