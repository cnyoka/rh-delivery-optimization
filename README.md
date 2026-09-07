# RH White Glove Delivery Optimization

Vehicle routing and delivery scheduling model for a Restoration Hardware (RH) white-glove furniture delivery case study, built during my MSBA at Hult International Business School (Spring 2026).

**[Live interactive dashboard on Tableau Public](https://public.tableau.com/app/profile/coovadia.nyoka3394/viz/RHWhiteGloveDeliveryOptimization/DailyOperations)**

The model takes 168 order lines across 19 customers, assigns them to daily truck loads under capacity and shift-time constraints, optimizes each day's stop sequence with Google OR-Tools, and computes total cost (truck, labor with pay rounding, inventory holding) and CO₂ emissions against a naive routing baseline.

## Problem

RH delivers premium furniture from a San Francisco distribution center using a 26-foot box truck (1,700 cu ft / 7,000 lbs) with a 3-person crew, Tuesday to Saturday, 8:00 AM to 5:00 PM. The task: schedule all deliveries at minimum cost while respecting truck capacity, shift length, and per-stop service times.

## Results

| Metric | Value |
|--------|-------|
| Deliveries scheduled | 19 customers over 9 delivery days (Tue to Sat) |
| Total operational cost | $12,010.45 ($632.13 average per order) |
| Total distance | 337.3 miles |
| CO₂, optimized | 661.2 kg (driving + idling) |
| CO₂, naive baseline | 715.1 kg |
| Emissions reduction | 53.9 kg (7.5%) vs unoptimized routing |

One finding worth noting: two single customer orders each exceed the truck's nominal volume capacity (104% and 111%) and are scheduled as dedicated solo trips. Strictly enforcing nominal volume would require same-day reload shuttles adding roughly 62 miles and 104 kg CO₂, more than erasing the optimization's emissions savings, so the model instead assumes blanket-wrapped, partially disassembled furniture packs below nominal assembled volume. The dashboard's capacity page documents this explicitly.

## Approach

| Step | What happens |
|------|--------------|
| 1 | Load orders, furniture dimensions, and customer addresses |
| 2 | Map every SKU to volume, weight, and load/unload times via a `(Brand, Product)` lookup dictionary |
| 3 | Compute per-customer volume, weight, and service time |
| 4 | Build a real driving-distance matrix (Google Maps Distance Matrix API, cached in-notebook) |
| 5 | Assign customers to daily loads with a first-fit-decreasing heuristic under capacity + time constraints |
| 6 | Optimize each day's stop sequence as a TSP with OR-Tools (nearest-neighbour fallback included) |
| 7 | Cost the schedule: $/mile truck operation, crew wages with 15-minute pay rounding and benefits load, holding costs for delayed items |
| 8 | Track CO₂ (driving + idling, EPA emission factors) and compare against an unoptimized baseline |
| 9 | Export schedule, manifest, daily summary, and assumptions to Excel for downstream dashboarding |

## Key modeling details

- **Objective function:** minimize truck cost (miles × $1.205/mile) + labor cost + holding cost, subject to volume, weight, and shift-window constraints.
- **Labor realism:** driver earns a 20% premium, pay rounds up to 15-minute intervals at the 7-minute mark, and a 1.30× benefits multiplier applies: small details that materially change daily labor cost.
- **CO₂ accounting:** 1.69 kg/mile driving + 3.97 kg/hr idling (EPA factors), reported alongside cost but not part of the objective.
- **Reproducibility:** all model constants live in one section; changing any value recalculates the entire model.

## Dashboards

The reporting layer exists in two implementations:

- **Power BI** (original course deliverable): 6-page dashboard on a live SharePoint pipeline with Power Query and DAX, supporting dynamic replanning scenarios.
- **Tableau Public** (individual rebuild): [live interactive version](https://public.tableau.com/app/profile/coovadia.nyoka3394/viz/RHWhiteGloveDeliveryOptimization/DailyOperations) with daily route maps, LIFO loading manifests, a full-schedule Gantt, capacity utilization checks, and cumulative cost and CO₂ tracking against the naive baseline. The Tableau data layer restructures the model's Excel export into grain-correct tables (stop, day, route-path, long-format cost and CO₂) with geocoded coordinates for the route maps.

## Tech stack

Python · pandas · NumPy · Google OR-Tools (routing solver) · Google Maps Distance Matrix API · openpyxl (Excel export) · Google Colab · Power BI · Tableau Public

## Running the notebook

1. Open `RH_Delivery_Optimization.ipynb` in Google Colab.
2. Upload the three source files when prompted (order CSV, furniture dimensions, customer addresses). **These case datasets are course materials and are not included in this repository.**
3. The distance matrix is cached in the notebook. To refresh it with live data, set your own `GOOGLE_MAPS_API_KEY` in Section 8.
4. Run all cells. Outputs export to Excel (schedule, manifest, daily summary, assumptions).

## Notes

- Data files come from the RH case study used in coursework and are excluded for copyright reasons.
- Built for a team case project (Spring 2026); the modeling, routing implementation, and cost logic in this notebook are my own work, and the Tableau rebuild is my individual work.
- Results above reflect the current committed notebook version, which is the run published in the Tableau dashboard.
