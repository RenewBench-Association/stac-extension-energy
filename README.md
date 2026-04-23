# Energy Extension Specification

- **Title:** Energy
- **Identifier:** <https://stac-extensions.github.io/energy/v1.0.0/schema.json>
- **Field Name Prefix:** energy
- **Scope:** Item, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @kalebphipps @emvollmer

This document explains the Energy Extension to the [SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.
It specifies energy related metadata, in particular that which is used to describe energy 
generation and storage data.

- Examples:
  - [Item example](examples/item.json): Shows the basic usage of the extension in a STAC Item
  - [Collection example](examples/collection.json): Shows the basic usage of the extension in a STAC Collection
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Fields

The fields in the table below can be used in these parts of STAC documents:

- [ ] Catalogs
- [x] Collections
- [x] Item Properties (incl. Summaries in Collections)
- [x] Assets (for both Collections and Items, incl. Item Asset Definitions in Collections)
- [ ] Links

| Field Name                   | Type      | Description                                                                                         |
|------------------------------|-----------|-----------------------------------------------------------------------------------------------------|
| `energy:entity_name`         | string    | **REQUIRED**. Name of the energy entity.                                                            |
| `energy:entity_type`         | string    | **REQUIRED**. The structural level the item represents (s. [entity_type](#energyentity_type)).      |
| `energy:group_name`          | string    | Name of the sub-facility group. Only permitted if `entity_type` is `unit`.                          |
| `energy:facility_name`       | string    | Name of the overarching facility. Only permitted if `entity_type` is `unit` or `group`.             |
| `energy:operator`            | string    | Name of the company that operates the entity.                                                       |
| `energy:entity_osm_id`       | string    | The OpenStreetMap (OSM) ID associated with the entity.                                              |
| `energy:group_osm_id`        | string    | The OSM ID of the sub-facility group. Only permitted if `entity_type` is `unit`.                    |
| `energy:facility_osm_id`     | string    | The OSM ID of the overarching facility. Only permitted if `entity_type` is `unit` or `group`.       |
| `energy:entity_ids`          | object    | Dictionary of IDs mapping ID types to values (s. [entity_ids](#energyentity_ids)).                  |
| `energy:fuel_type`           | string    | **REQUIRED**. The primary fuel used to generate energy (s. [fuel_type](#energyfuel_type)).          |
| `energy:fuel_subtype`        | string    | The subtype of fuel used (s. [fuel_subtype](#energyfuel_subtype)).                                  |
| `energy:hybrid_fuel_types`   | \[string] | **REQUIRED** if `fuel_type` is `hybrid`. The multiple types used to generate energy.                |
| `energy:hybrid_fuel_ratio`   | \[number] | Ratio or percentage of each fuel used in a hybrid system.                                           |
| `energy:generation_capacity` | number    | Installed capacity of the purely generating assets. Unit: MW.                                       |
| `energy:storage_power`       | number    | Maximum charge/discharge power of the storage system. Unit: MW.                                     |
| `energy:storage_capacity`    | number    | Total volume of energy the storage system can hold. Unit: MWh.                                      |
| `energy:status`              | string    | Operational status of the entity (s. [status](#energystatus)).                                      |
| `energy:measurement_type`    | string    | **REQUIRED**. Describes the kind of data recorded (s. [measurement_type](#energymeasurement_type)). |

### Additional Field Information

#### energy:entity_type

The `energy:entity_type` field defines the hierarchical level of the energy asset. 
Examples for suitable definitions include: 

| Value       | Description                                                             |
|-------------|-------------------------------------------------------------------------|
| `unit`      | An individual power-generating unit (e.g., a specific turbine).         |
| `group`     | A sub-facility group (e.g., a block or phase).                          |
| `facility`  | The overarching power plant / energy producing facility (e.g., a park). |
| `portfolio` | A collection of assets owned by a single company.                       |
| `region`    | A geographic aggregation of assets.                                     |
| `unknown`   | The structural level is not specified.                                  |

The `energy:entity_type` value should be set as whatever the **smallest** hierarchical 
level the data is given for. If more information is available, e.g. 
`energy:entity_type=unit` but the facility is also known, this can be provided in the 
additional `energy:facility_name` field.

#### energy:entity_ids

The `energy:entity_ids` object provides a flexible way to map multiple external 
identification systems. Through its structure as a `dict`, any additionally available 
entity ID can be stored under a descriptive `key`. Examples for such keys include: 

| Key    | Type   | Description                                                                                |
|--------|--------|--------------------------------------------------------------------------------------------|
| `gppd` | string | [Global Power Plant Database](https://resourcewatch.org/data/explore/Powerwatch) ID.       |
| `eic`  | string | [Energy Identification Code](https://www.entsoe.eu/data/energy-identification-codes-eic/). |
| `...`  | string | Any other existing ID system key.                                                          |


#### energy:fuel_type

The `energy:fuel_type` field identifies the primary source of energy. Examples for
suitable definitions include:

| Value                            | Description                                                              |
|----------------------------------|--------------------------------------------------------------------------|
| `gas`  `coal`, `oil`, `peat`     | Fossil fuel sources.                                                     |
| `nuclear`                        | Nuclear fission energy.                                                  |
| `geothermal`, `biomass`, `hydro` | Renewable thermal and water sources.                                     |
| `wind`, `solar`                  | Renewable wind and photovoltaic / solar sources.                         |
| `storage`                        | Stand-alone storage systems (e.g., batteries).                           |
| `wasteheat`                      | Energy recovered from thermal waste.                                     |
| `hybrid`                         | Systems using multiple fuel types (requires `energy:hybrid_fuel_types`). |

#### energy:fuel_subtype

The `energy:fuel_subtype` field provides additional specificity. Common pairings include:

| If `fuel_type` is... | `fuel_subtype`                        |
|----------------------|---------------------------------------|
| `gas`                | `lng`, `cng`, `biogas`                |
| `coal`               | `lignite`, `anthracite`, `bituminous` |
| `wind`               | `offshore`, `onshore`                 |
| `solar`              | `pv`, `csp`                           |

#### energy:status

The `energy:status` field describes the current lifecycle stage of the asset. Examples for
suitable definitions include:

| Value                | Description                 |
|----------------------|-----------------------------|
| `active`             | Currently operational.      |
| `inactive`           | Temporarily out of service. |
| `under_construction` | Asset is being built.       |
| `decommissioned`     | Permanently retired.        |

#### energy:measurement_type

The `energy:measurement_type` field categorizes the numerical data being reported. Examples for
suitable definitions include:

| Value                   | Description                                       |
|-------------------------|---------------------------------------------------|
| `gross_generation`      | Total production before internal plant loads.     |
| `net_generation`        | Actual energy injected into the grid.             |
| `simulated_generation`  | Data derived from physical or statistical models. |
| `estimated_generation`  | Data approximated via proxy variables.            |
| `forecast_generation`   | Predicted future energy production.               |
| `calculated_generation` | Derived via mathematical formulas.                |
| `curtailed_generation`  | Potential energy lost due to grid constraints.    |

---

## Contributing

All contributions are subject to the
[STAC Specification Code of Conduct](https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md).
For contributions, please follow the
[STAC specification contributing guide](https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md) Instructions
for running tests are copied here for convenience.

### Running tests

The same checks that run as checks on PR's are part of the repository and can be run locally to verify that changes are valid. 
To run tests locally, you'll need `npm`, which is a standard part of any [node.js installation](https://nodejs.org/en/download/).

First you'll need to install everything with npm once. Just navigate to the root of this repository and on 
your command line run:
```bash
npm install
```

Then to check markdown formatting and test the examples against the JSON schema, you can run:
```bash
npm test
```

This will spit out the same texts that you see online, and you can then go and fix your markdown or examples.

If the tests reveal formatting problems with the examples, you can fix them with:
```bash
npm run format-examples
```
