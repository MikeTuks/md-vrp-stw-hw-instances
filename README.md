[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.23298839.svg)](https://doi.org/10.5281/zenodo.23298839)

# MD-VRP-STW-HW instances

Benchmark instances for the multi-depot vehicle routing problem with soft time windows and
heterogeneous workers (MD-VRP-STW-HW), used in the honours thesis

> M. R. Tukker (2027). *PPO-based operator selection in ALNS for vehicle routing.*
> Department of Logistics, Stellenbosch University.

The set holds 1,464 instances, made up of 366 base instances in four matched variants each.
Every instance is derived from a 100-customer Solomon VRPTW file (Solomon, 1987) and keeps its
customer coordinates, demands, service times and time windows. On top of that, each instance
has synthetic depots, a heterogeneous worker fleet built from the vehicle types of Liu and Shen
(1999), and per-customer penalty coefficients for early and late arrival.

## Contents

    manifest.json            train, val and test instance IDs (the authoritative split)
    instances/
      full/            366   C101_n100_d3_seed001.json
      single_centre/   366   C101_n100_d1_seed001.json
      homogeneous/     366   C101_n100_d3_seed001_hom.json
      no_tw/           366   C101_n100_d3_seed001_notw.json
    worker_costs/
      liushen.json             raw Liu and Shen fixed costs, as stored in the instances
      liushen_scaled10.json    fixed costs divided by 10, as used in the thesis
    LICENSE                  CC BY 4.0

## Splits

Splits are made on whole Solomon base files, so no base geometry appears in more than one split.

| Split | Base files | Seeds per file | Per variant | All variants |
|---|---|---|---|---|
| train | R101 to R103, C101 to C103, RC101 to RC103, R201 to R203, C201 to C203, RC201 to RC203 | 17 | 306 | 1,224 |
| val | R104, C104, RC104 | 10 | 30 | 120 |
| test | R204, C204, RC204 | 10 | 30 | 120 |

`manifest.json` lists the instance IDs in each split. An ID is the file name without `.json`.

## Variants

The four variants of a given base file and seed share the same customers.

| Variant | Depots | Worker types | Time window penalties | File name |
|---|---|---|---|---|
| `full` | 3 | 3 | yes | no suffix |
| `single_centre` | 1, at the customer centroid | 3 | yes | no suffix, `d1` instead of `d3` |
| `homogeneous` | 3 | 1 | yes | `_hom` |
| `no_tw` | 3 | 3 | no (`alpha` and `beta` are 0) | `_notw` |

## File format

Each instance is a JSON object with these fields.

- `centres` holds the depots, each with `id`, `x` and `y`.
- `customers` holds `id`, `x`, `y`, `demand`, `ready_time`, `due_date` and `service_time`, plus
  `alpha` (lateness penalty per unit of time) and `beta` (earliness penalty per unit of time).
- `worker_types` holds `type_id`, `capacity`, `speed` and `fixed_cost`.
- `worker_counts` gives the number of workers of each type at each depot, for example
  `{"centre_0": {"A": 2, "B": 2, "C": 3}, ...}`.
- `meta` records `base_file`, `group` (the Solomon family, such as `C1`), `variant`, `seed`,
  `n_customers`, `n_centres`, `total_demand` and a few generation details.

## Conventions

Each of these affects the objective value, so it is worth checking them before comparing
results with the thesis.

- `fixed_cost` in the instance files is the raw Liu and Shen (1999) value. The thesis divides
  it by 10 at solve time. `worker_costs/liushen_scaled10.json` lists the values actually used,
  keyed by Solomon family (`meta.group`) and `type_id`.
- Distances are Euclidean and unrounded. Travel time is distance divided by the worker's
  `speed` and is not stored in the files.
- Time windows are soft. Lateness costs `alpha * max(0, arrival - due_date)` and earliness costs
  `beta * max(0, ready_time - arrival)`, both measured at the arrival time. A worker who arrives
  early waits until `ready_time` before starting service.
- Every worker that is used leaves its depot at time zero and returns to the same depot. A
  customer may be served by a worker from any depot.
- A worker carries at most the `capacity` of its type, summed over the `demand` of the
  customers it serves.
- The objective is total travel distance plus the time window penalties plus the fixed cost of
  each worker used.

## Licence

The instances, manifest and cost files are released under the Creative Commons Attribution 4.0
International licence (CC BY 4.0). See `LICENSE`.

## References

- Solomon, M. M. (1987). Algorithms for the vehicle routing and scheduling problems with time
  window constraints. *Operations Research*, 35(2), 254 to 265.
- Liu, F.-H. and Shen, S.-Y. (1999). A method for vehicle routing problem with multiple vehicle
  types and time windows. *Proceedings of the National Science Council, Republic of China (A)*,
  23(4), 526 to 536.
