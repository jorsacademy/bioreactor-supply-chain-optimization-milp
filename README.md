# Bioreactor Supply Chain Optimization MILP

A deterministic mixed-integer linear programming model for multi-period bioreactor production and distribution planning.

The model represents a two-echelon supply chain:

`Plants -> Distribution Centers -> Markets`

It jointly optimizes production quantities, setup decisions, plant inventory, distribution-center inventory, and transportation flows while respecting production limits, inventory capacities, exact market demand, and plant-to-distribution-center lead times.

## Scope

This repository is an Operations Research example intended for educational, research, and other non-commercial use.

All entities and data are synthetic. The repository does not use real company names, brands, or proprietary business data.

## Key Features

- Multi-period mixed-integer linear programming formulation
- Multiple plants, distribution centers, markets, and bioreactor types
- Binary production setup decisions
- Plant and distribution-center inventory balance equations
- Explicit plant-to-DC transportation lead times
- Exact demand satisfaction
- Planning-horizon protection against late shipments
- Input validation before model construction
- Solver-status validation before result extraction
- Real objective-function cost decomposition
- CSV result export
- Matplotlib visualization
- Automated regression tests
- Reproducible synthetic sample instance

## Model Structure

### Decision Variables

For each applicable period and network entity:

- `Production[t, p, k]`: units of product `k` produced at plant `p`
- `Setup[t, p, k]`: binary setup decision for product `k` at plant `p`
- `PlantInventory[t, p, k]`: end-of-period inventory at plant `p`
- `DCInventory[t, d, k]`: end-of-period inventory at distribution center `d`
- `PlantToDC[t, p, d, k]`: units dispatched from plant `p` to DC `d`
- `DCToMarket[t, d, m, k]`: units shipped from DC `d` to market `m`

### Objective

The objective minimizes total supply-chain cost:

- Manufacturing cost
- Production setup cost
- Plant inventory holding cost
- Distribution-center inventory holding cost
- Plant-to-DC transportation cost
- DC-to-market transportation cost

The reported cost breakdown is calculated directly from the solved objective components. No placeholder percentages are used.

### Main Constraints

1. Total production capacity at each plant and period
2. Product-specific production capacity linked to binary setup decisions
3. Plant inventory conservation
4. Plant inventory capacity
5. Distribution-center inventory conservation
6. Distribution-center inventory capacity
7. Exact market-demand satisfaction
8. Transportation lead-time synchronization
9. Prevention of shipments that would arrive after the planning horizon

## Lead-Time Logic

A shipment dispatched from plant `p` to distribution center `d` in period `tau` is available at the DC only in period:

`tau + lead_time[p][d]`

The model therefore distinguishes shipment departure from shipment arrival. This avoids the invalid same-period flow logic that often causes incorrect or infeasible multi-period supply-chain models.

The sample instance includes initial DC inventory so first-period demand can be served before newly produced units arrive.

## Installation

Python 3.10 or newer is recommended.

```bash
python -m venv .venv
```

Activate the environment and install dependencies:

```bash
pip install -r requirements.txt
```

The model uses PuLP with the CBC solver. Many PuLP installations include a CBC binary. If CBC is not available on your system, install the Coin-OR CBC solver separately.

## Run

```bash
python bioreactor_supply_chain.py
```

A successful run prints the optimization status, total cost, production volume, shipment totals, and objective decomposition.

It also creates a `results/` directory containing:

```text
results/
├── cost_breakdown.csv
├── dc_to_market.csv
├── inventory.csv
├── plant_to_dc.csv
├── production.csv
└── summary.png
```

## Tests

Run the regression suite with:

```bash
pytest -q
```

The tests verify:

- Optimal solver status
- Positive production in the sample instance
- Exact demand satisfaction
- Distribution-center balance equations with lead times
- Equality between objective value and reported cost components
- Preservation of zero-valued decisions in result tables
- Rejection of invalid first-period inventory data

## Reproducibility

The included sample data are deterministic. Running the same model with the same solver configuration should produce the same optimal objective value and an equivalent optimal plan.

No random demand, random transportation cost, or hidden state is used.

## Repository Design Choices

Direct plant-to-market shipments are intentionally excluded. This keeps the model consistent with a strict two-echelon distribution structure and prevents distribution centers from becoming irrelevant because of artificially cheap direct arcs.

Inter-DC shipments are also excluded. They can be added in a future extension as a separate transshipment decision class with its own lead times and costs.

All decision values, including zeros, are retained in exported result tables. This avoids misleading plots or summaries caused by filtering out inactive periods.

The solver output is read only after PuLP reports an `Optimal` solution. If the model is infeasible or otherwise unsolved, the program raises an error instead of reporting stale or invalid variable values.

## Possible Extensions

The formulation can be extended with:

- Backorders and lost sales
- Service-level constraints
- Safety stock
- Supplier procurement decisions
- Raw-material availability
- Production resource consumption by product type
- Capacity expansion
- Overtime
- Carbon-emission objectives or constraints
- Scenario-based stochastic demand
- Robust optimization
- Multi-objective optimization
- Inter-DC transshipment
- Customer delivery lead times

## License

This repository is provided under a non-commercial software license. Commercial use is prohibited.

See [LICENSE](LICENSE) for the complete terms.

## Disclaimer

This project is a synthetic Operations Research example. It is not production-planning advice, regulatory guidance, or a validated model for any specific manufacturing operation.
