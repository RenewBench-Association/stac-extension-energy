# Energy Extension Specification

- **Title:** Energy
- **Identifier:** <https://stac-extensions.github.io/energy/v0.1.0/schema.json>
- **Field Name Prefix:** energy
- **Scope:** Item, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @kalebphipps @emvollmer

This document explains the Energy Extension to the [SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.
It specifies energy related metadata, in particular that which is used to describe energy 
generation and storage data.

- Examples:
  - Items:
    - [Wind turbine](examples/item-unit-wind.json) (`entity_type=unit`, `fuel_type=wind`): 
      Shows a single-fuel unit-level entity with group and facility references.
    - [Battery block](examples/item-group-storage.json) (`entity_type=group`, 
      `fuel_type=storage`): 
      Shows a storage block entity with associated power and capacity.
    - [Hybrid facility](examples/item-facility-hybrid.json) (`entity_type=facility`, 
      `fuel_type=hybrid`): 
      Shows a hybrid generation facility entity with two fuel types and ratios.
  - Collections:
    - [Wind park](examples/collection-windpark.json): Shows an example of a **highly homogeneous** Collection where all 
      Items belong to one wind park. Shared metadata (such as operator and fuel) are defined
      as top-level `energy` fields and only operational characteristics (like statuses or 
      entity types) are summarized.
    - [Country](examples/collection-country.json): Shows an example of a **heterogeneous** Collection of all Items within
      one geographic region. Due to the data diversity, extensive summary fields are 
      used in lieu of top-level fields.
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Fields

The fields in the table below can be used in these parts of STAC documents:

- [ ] Catalogs
- [x] Collections
- [x] Item Properties (incl. Summaries in Collections)
- [ ] Assets (for both Collections and Items, incl. Item Asset Definitions in Collections)
- [ ] Links

| Field Name                     | Type      | Description                                                                                                                        |
|--------------------------------|-----------|------------------------------------------------------------------------------------------------------------------------------------|
| `energy:entity_name`           | string    | **REQUIRED**. Name of the energy entity.                                                                                           |
| `energy:entity_type`           | string    | **REQUIRED**. The structural level the item represents (s. [entity_type](#energyentity_type)).                                     |
| `energy:group_name`            | string    | Name of the sub-facility group. *Only permitted if `entity_type` is `unit`.*                                                       |
| `energy:facility_name`         | string    | Name of the overarching facility. *Only permitted if `entity_type` is `unit` or `group`.*                                          |
| `energy:operator`              | string    | Name of the company that operates the entity.                                                                                      |
| `energy:entity_osm_id`         | string    | The [OpenStreetMap (OSM)](https://www.openstreetmap.org) ID associated with the entity (s. [entity_osm_id](#energyentity_osm_id)). |
| `energy:group_osm_id`          | string    | The OSM ID of the sub-facility group. *Only permitted if `entity_type` is `unit`.*                                                 |
| `energy:facility_osm_id`       | string    | The OSM ID of the overarching facility. *Only permitted if `entity_type` is `unit` or `group`.*                                    |
| `energy:entity_ids`            | object    | Dictionary of IDs mapping ID types to values (s. [entity_ids](#energyentity_ids)).                                                 |
| `energy:fuel_type`             | string    | **REQUIRED**. The primary fuel used to generate energy (s. [fuel_type](#energyfuel_type)).                                         |
| `energy:fuel_subtype`          | string    | The subtype of fuel used (s. [fuel_subtype](#energyfuel_subtype)).                                                                 |
| `energy:hybrid_fuel_types`     | \[string] | **REQUIRED if `fuel_type` is `hybrid`**. The multiple types used to generate energy.                                               |
| `energy:hybrid_fuel_fractions` | \[number] | Fraction of each fuel used in a hybrid system. *Only permitted if `fuel_type` is `hybrid`; length must match `hybrid_fuel_types`.* |
| `energy:generation_capacity`   | number    | Installed capacity of the purely generating assets. Unit: MW.                                                                      |
| `energy:storage_power`         | number    | Maximum charge/discharge power of the storage system. Unit: MW.                                                                    |
| `energy:storage_capacity`      | number    | Total volume of energy the storage system can hold. Unit: MWh.                                                                     |
| `energy:status`                | string    | Operational status of the entity (s. [status](#energystatus)).                                                                     |
| `energy:measurement_type`      | string    | **REQUIRED**. Describes the kind of data recorded (s. [measurement_type](#energymeasurement_type)).                                |

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

> **IMPORTANT**
>
> The `energy:entity_type` value should be set as whatever the **smallest** hierarchical 
> level the data is given for. If more information is available, e.g. 
> `energy:entity_type=unit` but the facility is also known, this can be provided in the 
> additional `energy:facility_name` field.

#### `energy:entity_osm_id`

The `energy:entity_osm_id` field defines the ID that is associated with the energy 
entity on OSM. Following OSM conventions, it should be formatted as `<type>/<id>` where 
`<type>` is one of `node`, `way`, or `relation`. The specific type definition depends on how the 
corresponding entity is represented in OSM. For reference, some examples include:

- a wind turbine listed as [a node](https://www.openstreetmap.org/node/3347682357#map=14/53.45696/-3.30075),
- wind farms defined as [a node](https://www.openstreetmap.org/node/310852906),
  [a way](https://www.openstreetmap.org/way/327949356#map=12/53.4824/-3.2598), or
  [a relation](https://www.openstreetmap.org/relation/6949277#map=12/55.4521/-3.5763).

> **TIP**
>
> Depending on what the `entity_type` itself is, `energy:group_osm_id`, and 
> `energy:facility_osm_id` can similarly be populated with OSM IDs. These must follow the 
> same convention.

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
| `wasteheat`                      | Energy recovered from thermal waste.                                     |
| `storage`                        | Stand-alone storage systems (e.g., batteries).                           |
| `hybrid`                         | Systems using multiple fuel types (requires `energy:hybrid_fuel_types`). |

> **IMPORTANT**
>
> If `energy:fuel_type` is defined as `hybrid`, the field `energy:hybrid_fuel_types` 
> must be populated with a list of which exact `fuel_type` values make up said hybrid. If 
> the fractions of each is known, this information can be provided via 
> `energy:hybrid_fuel_fractions`. Both field lists must have the same length.
> An example would be:
>
> - `energy:hybrid_fuel_types`: `["gas", "geothermal"]`
> - `energy:hybrid_fuel_fractions`: `[0.3, 0.7]`

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

| Value               | Description                                       |
|---------------------|---------------------------------------------------|
| `gross_output`      | Total production before internal plant loads.     |
| `net_output`        | Actual energy injected into the grid.             |
| `simulated_output`  | Data derived from physical or statistical models. |
| `estimated_output`  | Data approximated via proxy variables.            |
| `forecast_output`   | Predicted future energy production.               |
| `calculated_output` | Derived via mathematical formulas.                |
| `curtailed_output`  | Potential energy lost due to grid constraints.    |

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
