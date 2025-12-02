# ⚖️ NovaBalance Worker: LCA Calculator

> **The Environmental Physics Engine.**
> Automated, real-time Life Cycle Assessment (LCA) based on dynamic Material Bills of Materials (BOM) and energy grids.

[](https://www.google.com/search?q=https://github.com/novaeco-tech/novabalance-worker-lca-calculator/actions)
[](https://opensource.org/licenses/MIT)
[](https://www.google.com/search?q=https://lca.balance.novaeco.tech)

**LCACalc** is a headless, event-driven worker belonging to the **[NovaBalance](https://www.google.com/search?q=https://balance.novaeco.tech)** enabler.

Traditional LCAs are static, expensive PDF reports produced once a year. **LCACalc** makes environmental auditing dynamic. It listens for changes in the ecosystem (e.g., a new design in `NovaMake` or a completed harvest in `NovaAgro`) and instantly calculates the environmental footprint using the latest emission factors and grid intensities.

-----

## 🎯 Value Proposition

To close the loop, we must measure the loop. This worker provides the "Scoreboard" for the Circular Economy:

1.  **Dynamic Granularity:** Instead of using a global average for "Steel," it calculates the specific impact of *this* batch of steel, produced with *that* specific energy mix.
2.  **Grid Awareness:** It integrates with **NovaEnergy** to check if a factory was running on Solar or Coal at the exact moment of production.
3.  **Passport Injection:** It automatically writes the verified Carbon Footprint into the **Digital Product Passport (DPP)** managed by `NovaMaterial`.

-----

## 🏗️ Architecture (The Calculation Pipeline)

This worker consumes the `queue.balance.calculation-requests` queue managed by NovaBalance.

```mermaid
graph LR
    subgraph "Triggers"
        Make["NovaMake (Job Done)"] -->|1. Event: Material + Energy Used| MQ[(RabbitMQ)]
        Mat["NovaMaterial (New Design)"] -->|1. Event: BOM| MQ
    end

    subgraph "The Worker (NovaBalance)"
        MQ -->|2. Ingest Data| Worker[LCA Worker]
        Worker -->|3. Fetch Grid Mix| Energy[NovaEnergy API]
        Worker -->|4. Lookup Emission Factors| DB[(LCA Database)]
        Worker -->|5. Compute Impact| Engine[SciPy Engine]
    end

    subgraph "Outputs"
        Worker -->|6. Update Passport| Mat
        Worker -->|7. Audit Log| Balance[NovaBalance Ledger]
    end
```

### The Logic Flow

1.  **Input:** A JSON payload describing inputs.
      * *Example:* `{"material": "PET_Recycled", "mass_kg": 5.0, "energy_kwh": 2.5, "location": "DE-BY"}`.
2.  **Factor Lookup:** Queries the internal database (seeded with Idemat/Ecoinvent proxies) for the emission factor of "PET\_Recycled".
3.  **Grid Correction:** Queries `NovaEnergy` for the carbon intensity of the German/Bavarian grid (`DE-BY`) at the timestamp of production.
4.  **Calculation:** `(Mass * Material_EF) + (Energy * Grid_EF) = Total_CO2e`.
5.  **Certification:** Signs the result and emits an `ImpactCalculated` event.

-----

## ✨ Key Features

### 1\. "Cradle-to-Gate" Automation

It automatically scopes the assessment based on the input data depth.

  * **Scope 1:** Direct emissions (if sensor data from `NovaInfra` is present).
  * **Scope 2:** Energy usage (via `NovaEnergy`).
  * **Scope 3:** Upstream material impacts (via `NovaMaterial` linkage).

### 2\. The "Green Grid" Bonus

This is a critical incentive feature.

  * If a manufacturer in `NovaMake` waits to run their machines until `NovaEnergy` signals "High Wind Output," this worker records a significantly lower CO₂ score for that specific product batch.

### 3\. Multi-Dimensional Metrics

It doesn't just count Carbon.

  * **GWP:** Global Warming Potential (kg CO₂e).
  * **H₂O:** Blue Water Consumption (Liters).
  * **Toxicity:** Human Toxicity Potentials (HTP).

-----

## 🚀 Getting Started

### Prerequisites

  * Docker Desktop
  * Python 3.11+ (We use `pandas` and `scipy` for calculations)

### Installation

1.  **Clone the repo:**
    ```bash
    git clone https://github.com/novaeco-tech/novabalance-worker-lca-calculator.git
    cd novabalance-worker-lca-calculator
    ```
2.  **Start the Dev Environment:**
    ```bash
    make dev
    ```
      * Starts the worker and a mock LCA database (Postgres).
      * **Health Check:** `http://localhost:8080/health`

### Configuration (`.env`)

```ini
# Queue
RABBITMQ_URI=amqp://guest:guest@rabbitmq:5672/
QUEUE_NAME=queue.balance.calculation-requests

# External Services (gRPC/REST)
NOVAENERGY_URL=http://novaenergy-api:8000
NOVAMATERIAL_URL=http://novamaterial-api:8000

# Calculation Constants
DEFAULT_GRID_INTENSITY=0.475 # kgCO2/kWh (fallback)
```

-----

## 📂 Repository Structure

```text
novabalance-worker-lca-calculator/
├── src/
│   ├── main.py             # Consumer Loop
│   ├── engine/             # The core math logic (ISO 14040 compliant)
│   ├── factors/            # Database adapters for Emission Factors
│   └── models/             # Pydantic schemas for BOMs and Impact
├── data/                   # Seed data for common materials (plastic, steel)
├── tests/                  # Pytest suite
└── Dockerfile              # Python data-science runtime
```

-----

## 🧪 Testing

We use **Scientific Verification** testing.

  * **Formula Tests:** `make test-math`
      * Verifies that the calculations match standard ISO reference datasets.
  * **Integration Tests:** `make test-flow`
      * Simulates a full message flow: Ingest BOM -\> Mock Energy API -\> Verify Output JSON.

-----

## 🤝 Contributing

We need contributors with backgrounds in **Environmental Science**, **Data Engineering**, and **Python**.
See [CONTRIBUTING.md](https://www.google.com/search?q=../.github/CONTRIBUTING.md) for details.

**Maintainers:** `@novaeco-tech/maintainers-enabler-novabalance`