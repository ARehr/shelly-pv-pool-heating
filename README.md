# PV-optimized solar pool heating with differential temperature control

This project controls a solar pool heating pump based on PV surplus power, differential temperature measurement and Home Assistant automation.

Instead of running the pool pump by a fixed timer, the pump only starts when there is enough surplus solar power available and the solar collector is actually warmer than the pool water.

The system uses Shelly hardware for switching, power measurement and temperature measurement.

## Project idea

A solar pool collector only provides useful heat when its temperature is sufficiently above the pool water temperature.

A simple timer-based control may run the pump even when there is no useful thermal gain or when electricity has to be imported from the grid.

This automation combines three decision criteria:

1. Available PV surplus power
2. Pool water temperature
3. Temperature difference between solar collector and pool water

The result is a simple but effective control strategy: the pump only runs when the collector can actually transfer heat into the pool and when enough solar electricity is available.

## Hardware

- Shelly Plus 2PM
- Shelly sensor add-on
- Two temperature sensors
- Solar pool collector
- Pool circulation pump
- Home Assistant
- PV surplus power sensor

## Measurement concept

Two temperatures are measured:

- **Collector temperature**: measured at the warm pipe / pressure side near the solar collector
- **Pool temperature**: measured directly in the pool water / suction side

The temperature difference is calculated as:

```text
Delta T = collector temperature - pool temperature
```

When the pump starts, cooler pool water flows through the solar collector. This causes the collector temperature sensor to drop, which is expected and shows that heat is being transferred from the collector circuit into the pool water.

## Control logic

### Turn-on conditions

The pump is switched on when all of the following conditions are met:

- PV surplus power is greater than 600 W
- Pool temperature is below 32 °C
- Collector temperature is at least 4 K above pool temperature
- Pump is currently off

### Turn-off conditions

The pump is switched off when at least one of the following conditions is met:

- Temperature difference falls below 2 K
- PV surplus power falls below 600 W
- Pool temperature reaches the configured target temperature

## Hysteresis

The automation uses a simple hysteresis:

- Turn-on threshold: 4 K temperature difference
- Turn-off threshold: 2 K temperature difference

This avoids unnecessary switching and prevents the pump from rapidly cycling on and off when the temperature difference is close to the threshold.

## Estimated thermal output

In addition to the switching logic, the system estimates the thermal power transferred from the solar collector to the pool.

The calculation is based on the temperature difference and an assumed hydraulic flow rate.

For water, the thermal power can be estimated as:

```text
Thermal power [kW] = flow rate [l/min] × 0.0698 × Delta T [K]
```

In this setup, the assumed flow rate is approximately 37.5 l/min, which corresponds to about 2.25 m³/h.

This results in the simplified factor:

```text
Thermal power [kW] = Delta T [K] × 2.62
```

The value is an estimate because the flow rate is not measured continuously. It is used to visualize the approximate useful solar heat transferred into the pool.

## Effective performance ratio

The system also calculates an effective performance ratio:

```text
Performance ratio = estimated thermal output / electrical pump input
```

This is not a heat pump COP.  
It describes how much estimated solar heat is transferred into the pool per unit of electrical energy used by the circulation pump.

## Benefits

- Higher PV self-consumption
- Reduced grid import
- Reduced unnecessary pump runtime
- Better use of solar thermal gain
- Simple differential temperature control
- Robust hysteresis logic
- Estimated thermal output calculation
- Estimated performance ratio
- Fully implemented in Home Assistant YAML
- Uses Shelly hardware for switching, power measurement and temperature measurement

## Default parameters

| Parameter | Value |
|---|---:|
| PV surplus turn-on threshold | 600 W |
| PV surplus turn-off threshold | 600 W |
| Pool target temperature | 32 °C |
| Differential temperature turn-on threshold | 4 K |
| Differential temperature turn-off threshold | 2 K |
| Assumed flow rate | approx. 37.5 l/min |
| Heating power factor | 2.62 kW/K |

## Files

This repository contains:

```text
pool_solar_heating_automation.yaml
```

The YAML file contains:

- Template sensor for temperature difference
- Template sensor for estimated heating power
- Template sensor for estimated performance ratio
- Home Assistant automation for turning the pump on
- Home Assistant automation for turning the pump off

## Screenshots

Recommended screenshots for documentation:

- Home Assistant pool dashboard
- Temperature curve of pool and collector
- PV surplus and pump power curve
- Shelly temperature sensor overview
- Optional photo of the collector temperature sensor mounted on the pipe

## Notes

The published YAML file uses anonymized and generic entity names.

Before using it in another Home Assistant installation, the entity IDs must be adapted to the local setup.
