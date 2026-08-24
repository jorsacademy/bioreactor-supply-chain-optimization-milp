# Bioreactor Supply Chain Optimization MILP

This repository contains mixed-integer linear programming models for multi-period bioreactor supply-chain planning.

All entities, costs, capacities, and demand values are synthetic. No real or disguised company names, brands, or proprietary operating data are used.

The repository now contains two models:

1. A deterministic production and distribution model.
2. An extended robust model with raw-material procurement, supplier capacities, bills of materials, procurement lead times, and demand uncertainty.

## Network Structures

Deterministic model:

`Plants -> Distribution Centers -> Markets`

Robust procurement model:

`Suppliers -> Plants -> Distribution Centers -> Markets`

## Scope

This project is an Operations Research example intended for educational, research, and other non-commercial use.

Commercial use is prohibited under the repository license.

## Deterministic Model

The deterministic implementation is located in:

```text
bioreactor_supply_chain.py
```

It jointly optimizes:

- Product-specific production quantities
- Binary production setup decisions
- Plant inventory
- Distribution-center inventory
- Plant-to-DC shipments
- DC-to-market shipments

The formulation includes production limits, inventory capacities, exact demand satisfaction, plant-to-DC transportation lead times, and protection against shipments that would arrive after the planning horizon.

## Robust Procurement Model

The extended implementation is located in:

```text
robust_procurement_model.py
```

This model adds a complete upstream procurement layer:

`Suppliers -> Plants`

and explicitly represents raw materials consumed during bioreactor production.

### Additional Decision Variables

- `Procurement[t, s, p, r]`: raw material `r` ordered from supplier `s` for plant `p`
- `RawInventory[t, p, r]`: end-of-period raw-material inventory at plant `p`

The downstream production and distribution variables remain explicit.

### Raw-Material Constraints

The robust model includes:

1. Supplier capacity by period and raw material
2. Supplier-to-plant procurement lead times
3. Raw-material inventory conservation
4. Raw-material storage capacity
5. Product-specific bills of materials
6. Material consumption linked directly to production decisions
7. Prevention of procurement orders that would arrive after the planning horizon

For each plant and raw material, the inventory balance has the form:

```text
Ending Raw Inventory
= Previous Raw Inventory
+ Arriving Procurement
- Production Consumption
```

Production consumption is calculated from the bill of materials:

```text
Consumption[p, r, t]
= sum(BOM[k, r] * Production[t, p, k])
```

## Robust Demand Formulation

Demand uncertainty is modeled using an interval, or box, uncertainty set.

For each period, market, and product:

```text
Demand in [Nominal Demand, Nominal Demand + Maximum Deviation]
```

The robust demand requirement is therefore:

```text
Delivered[t, m, k] >= NominalDemand[t, m, k] + DemandDeviation[t, m, k]
```

This protects the plan against the simultaneous upper endpoint of every specified demand interval.

This is a conservative robust formulation by design. It does not use probability distributions or scenario probabilities. It is appropriate when demand deviations are interpreted as hard uncertainty bounds rather than statistical forecasts.

## Objective Function

The deterministic model minimizes:

- Manufacturing cost
- Production setup cost
- Plant inventory holding cost
- Distribution-center inventory holding cost
- Plant-to-DC transportation cost
- DC-to-market transportation cost

The robust procurement model additionally includes:

- Raw-material procurement cost
- Supplier-to-plant transportation cost
- Raw-material inventory holding cost

All reported cost components are calculated directly from the solved objective expression. No placeholder percentages are used.

## Lead-Time Logic

A procurement order dispatched from supplier `s` to plant `p` in period `tau` becomes available in:

```text
tau + procurement_lead_time[s][p]
```

A finished-product shipment dispatched from plant `p` to DC `d` in period `tau` becomes available in:

```text
tau + distribution_lead_time[p][d]
```

The model therefore separates departure periods from arrival periods at both upstream and downstream echelons.

## Installation

Python 3.10 or newer is recommended.

```bash
python -m venv .venv
pip install -r requirements.txt
```

The models use PuLP with the CBC MILP solver. Many PuLP installations include CBC. If CBC is unavailable on the host system, install the Coin-OR CBC solver separately.

## Run the Deterministic Model

```bash
python bioreactor_supply_chain.py
```

The deterministic model exports CSV result tables and a summary visualization to the `results/` directory.

## Run the Robust Procurement Model

```bash
python robust_procurement_model.py
```

A successful run prints:

- Solver status
- Total optimized cost
- Total raw-material procurement
- Total bioreactor production
- Total market deliveries
- Actual objective cost decomposition

## Tests

Run the complete regression suite with:

```bash
pytest -q
```

The tests cover both formulations.

Deterministic tests verify:

- Optimal solver status
- Positive production
- Exact demand satisfaction
- Distribution-center inventory balance with lead times
- Objective decomposition
- Preservation of zero-valued decisions
- Input validation

Robust procurement tests verify:

- Optimal solver status
- Positive procurement and production
- Satisfaction of the robust demand upper bound
- Raw-material balance with procurement lead times
- Bill-of-material consumption accounting
- Objective decomposition
- Rejection of negative demand deviations
- Rejection of insufficient first-period robust inventory

## Model Design Choices

Direct plant-to-market shipment is intentionally excluded. The downstream network therefore remains a strict two-echelon distribution structure.

Inter-DC shipment is also excluded. It can be introduced as a separate transshipment decision class if required.

The robust model currently uses box uncertainty. This means every demand component is protected up to its specified maximum deviation at the same time. The formulation is transparent and fully linear, but more conservative than budgeted or probabilistic uncertainty models.

## Future Extensions

Natural extensions include:

- Bertsimas-Sim budgeted robust optimization
- Two-stage stochastic programming with scenario probabilities
- Scenario-dependent recourse decisions
- Chance constraints
- Backorders and lost sales
- Service-level constraints
- Safety stock
- Supplier minimum-order quantities
- Supplier fixed-order costs
- Supplier disruption scenarios
- Multi-source qualification constraints
- Yield uncertainty
- Capacity expansion
- Overtime
- Carbon-emission objectives or constraints
- Multi-objective optimization

## Reproducibility

The included examples are deterministic and synthetic. No random demand, random transportation costs, external APIs, proprietary datasets, or hidden state are required.

## License

This repository is provided under a non-commercial software license. Commercial use is prohibited.

See [LICENSE](LICENSE) for the complete terms.

## Disclaimer

This project is a synthetic Operations Research example. It is not production-planning advice, regulatory guidance, procurement advice, or a validated model for any specific manufacturing operation.
