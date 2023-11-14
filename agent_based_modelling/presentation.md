---
marp: true
theme: uncover
paginate: true
description: agent based modelling @ centre for net zero
transition: fade
header: agent based modelling @ cnz
footer: :earth_africa: slides by [ryan](https://www.ryan.eco) || 𝕏 @ryancjenkinson
style: |
  .columns {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 1rem;
  }
---

<!-- _header: '' -->
![bg left:40% 80%](https://www.bankofengland.co.uk/-/media/boe/images/publications/qb/agent-based-models.png?h=478&w=500&hash=13068B3B89DAE3A6159BE0FA2CA04647)

# **agent based modelling @ cnz**

lessons from modelling the GB target of 600,000 heat pumps by 2028 using an ABM

---

![bg right:40% 80%](https://github.com/ryanjenkinson.png)

# 👋 **hey, i'm ryan**

#### 👇 about me 👇

* 📈 data science @ krakenflex
* 🧠 works on domestic asset optimisation - heat pumps [hp], electric vehicles [ev], solar & storage (batteries)
* 🤝 worked @ cnz before

---

![bg right:40% 80%](./figures/cnz_report.png)

# CNZ Report: Hitting the target

📌 **Goal:** _what combination of policy interventions are required to meet the UK target of 600,000 HPs by 2028?_

<!-- 
* This was the main question we were answering
* As you can see, it defines a clear metric we were able to use our ABM to answer, which was important
 -->

---

# Agenda

1. [What is an agent based model?](#5)
2. [How did we build the model?](#13)
3. [What did we learn?](#19)
4. [Any questions?](#27)

---

# Agenda

1. [**What is an agent based model?**](#5)
2. [How did we build the model?](#13)
3. [What did we learn?](#19)
4. [Any questions?](#27)

---

# What is an agent based model (ABM)?

![width:700px](https://organizationalphysics.com/wp-content/uploads/2016/09/Top-down-hierarchy-vs-bottom-up-design-1024x530.png)

<!-- 
* Most models are top down (they model whole regions / areas as one "object")
* ABMs are bottom up - the "unit" is a household, and we define the rules of the game and "aggregate up" to see how the entire system behaves
 -->

---

# An agent based model needs

1. `Agent`s, which we considered to be a `Household`

2. `Environment`, which dictates _costs, incentives_ etc

3. `Rules`, for how the agents _interact with each other and the environment_ at each timestamp

At each timestep, the environment evolves according to the rules, and the agents respond to the environment.

---

### Our approach centred on households as the key decision maker

![width:900px](./figures/abm_diagram.png)

<!-- 
* In our research households and household-environment interactions were most common
* But in the future, you _could_ have agent-agent interactions (e.g you are more likely to buy a gas boiler because your neighbour has one). Research suggested this wasn't the case, so isnt in our model, but could be added.
* Environment contains things like prices, installer availability, policy interventions like subsidies
* Agents contain information about the household, what heating systems they know about etc
 -->

---

# like code? same :)

<div class="columns">
<div>

This is what an agent looks like in the code.

You can make them with

```python
agents = [Household(**household) for household in households]
```

and then add them to the environment

</div>
<div>

```python
class Household(Agent):
    def __init__(
        self,
        id: int,
        location: str,
        property_value_gbp: int,
        total_floor_area_m2: int,
        is_off_gas_grid: bool,
        construction_year_band: Optional[ConstructionYearBand],
        property_type: PropertyType,
        built_form: BuiltForm,
        heating_system: HeatingSystem,
        heating_system_install_date: datetime.date,
        epc_rating: EPCRating,
        potential_epc_rating: EPCRating,
        occupant_type: OccupantType,
        is_solid_wall: bool,
        walls_energy_efficiency: int,
        windows_energy_efficiency: int,
        roof_energy_efficiency: int,
        is_heat_pump_suitable_archetype: bool,
        is_heat_pump_aware: bool,
    ):
```

</div>
</div>

---

# the `Environment` was similar

<div class="columns">
<div>

```python
class DomesticHeatingABM(AgentBasedModel):
    def __init__(
        self,
        start_datetime: datetime.datetime,
        step_interval: relativedelta,
        # ... some other stuff ...
        interventions: Optional[List[InterventionType]],
        # ... some other stuff ...
        price_gbp_per_kwh_gas: float,
        price_gbp_per_kwh_electricity: float,
        price_gbp_per_kwh_oil: float,
        air_source_heat_pump_price_discount_schedule: Optional[
            List[Tuple[datetime.datetime, float]]
        ],
        heat_pump_installer_count: int,
        heat_pump_installer_annual_growth_rate: float,
        annual_new_builds: Optional[Dict[int, int]],
    ):
```

</div>

We made assumptions globally about how the "world model" evolved. These were parameters, so we could change them and run lots of scenarios.

</div>
</div>

<!-- 
* This was quite parallelisable, we could set several simulations running in parallel and change these parameters as needed via a CLI.
-->

---

![bg 40%](./figures/code_vs_high_level_description.png)

---

# Agent decision making loop

![width:700px](./figures/agent_decision_loop.png)

---

# Agenda

1. [What is an agent based model?](#5)
2. [**How did we build the model?**](#13)
3. [What did we learn?](#19)
4. [Any questions?](#27)

---

# ABM Architecture Diagram

![width:1000px](./figures/abm_architecture_diagram.png)

---

# Development pipeline

* ~~We~~Dani did a literature review of the "key" behaviours we want to model. As you'll see later, only worry about the big stuff.
* **Test-driven development**: write the _test of expected behaviour_ then add the feature as code.
* Check how it impacted our _key metric_ (HPs installed)
* Sensitivity analysis (see later)
* Run a notebook of graphs to check results against scenarios

<!-- 
* Adding lots of behaviours can get complicated. If you want behaviour A then add a feature for behaviour B. How do you know you haven't broken behaviour A? We had tests in python to test we didnt break anything, and tested desired outcomes rather than explicit unit tests (e.g if the prices were extremely high, no one would buy that heating system) rather than asserting on probabilities)
 -->

---

![bg right:55% 95%](figures/sensitivity_tornado.png)

**Technical: The model is sensitive to awareness and upfront costs**

![width:290px](figures/sensitivity_table.png)

---

**Technical: How we calibrated and validated the ABM**

Show that you can recreate a historical dataset with default parameters. We used RHI uptake from 2010 - 2021.
![bg right:45% 90%](figures/model_calibration.png)

<!-- 
This shows that your rule based bottom-up approach can emulate a top-level view
* Then - <big assumption> - you assume purchasing behaiour doesnt change and simulate the future
* Was popular for covid modelling
* We felt this was OK since we were simulating "near term" e.g out to ~2035.
* But Russia-Ukraine, inflation etc might lead to a fundamentally different purchasing behaviour now compared to when we built our model.
 -->

---

#### Scenarios were agreed with our policy team

![width:800px](./figures/scenarios.png)

<!-- 
* Based on things that government was thinking about at the time and we felt likely to happen
* Up to scenarios that might be more ambitious
 -->

---

# Agenda

1. [What is an agent based model?](#5)
2. [How did we build the model?](#13)
3. [**What did we learn?**](#19)
4. [Any questions?](#27)

---
<style scoped>
table {
  font-size: 16px;
}
</style>

# Our Findings

| Area | Description of Finding |
| --- | --- |
| **Upfront costs** | Present the biggest barrier to adoption by our household agents. Even in our most ambitious projections, the upfront cost of a HP is still 1.5- 2 times more expensive than an equivalent boiler in the short-medium term.  |
| **Awareness** | Our model is most sensitive to awareness and rate of renovation; changing these values leads to a large increase in the cumulative HP installations over the time period simulated. |
| **Resolving Friction** | Renovation is most important trigger for agents to purchase new heating systems. Resolving frictions a household experiences with changing heating system type in a ‘near-breakdown’ scenario, especially if property type and insulation are sufficient is important to meeting targets is important for avoiding lock-in to fossil fuel systems. |
| **Boiler Ban** | Of scenarios that enable us to reach the Government’s target, the early announcement of a fossil fuel boiler ban with a commencement date of 2035 is crucial to driving uptake. |
| **Policy costs** | Redistribution of policy costs does not materially change the number of HP installations since upfront costs outweigh the future running costs of a heating system in a household’s decision-making. |
| **Training** | Training more HP installers is necessary to meet demand in the short-medium term. Our simulations suggest 30,000 HP installers are required by 2028, a tenfold increase on today’s numbers and approx a quarter of existing Gas Safe installers. Failure to do so will lead to medium term capacity constraints in 2025/27. |

---

![bg 90%](./figures/finding_1.png)

<!-- 
* During BUS costs of heating system falls (because of subsidy)
* Post-BUS, due to learning rates, system costs are slightly higher than when there is a subsidy, but lower than pre-BUS
* We shows that during the BUS the average property price of households buying that heating system (proportional to welfare of house) was lower on average, meaning less affluent households could buy this technology
 -->

---

![bg 90%](./figures/finding_1_cont.png)

<!-- 
* This is shown here in this locational map too
 -->

---

![bg 90%](./figures/finding_2.png)

<!-- 
* Our model prioritised new build installs over existing build installs (under hypothesis it would be easier to install in a new build), but this shows there might be constraints on installers if we cannot grow the base fast enough
* An existing-build heat pump was only installed if there was an installer available
* Limitation: national availability not regional availability!
 -->

---

![bg 90%](./figures/finding_3.png)

<!-- 
* Some scenarios get us there!
* These scenarios tend to involve a ban, which we assumed increase awareness dramatically, which we know our model is sensitive to.
* Still, customers would only choose to get it if they could afford it (we weren't "ripping out" existing boilers) and was still proportional to the cost of the system (so more likely to get a boiler over a HP)
 -->

---

# Recommendations to Policymakers

1. Take a longer-term view of upfront costs
2. Launch a public awareness campaign on decarbonising home heating
3. Set an unambiguous time frame for the phase out of fossil fuel heating systems as early as possible
4. Prioritise efforts to grow heat pump installer numbers

---


# Stuff we learnt

<style scoped>
section {
  font-size: 30px;
}
</style>

* Everything you add to the model is an assumption. **Add as little as possible**, because you'll have to justify everything and you want a model you can explain. You _can_ have too many things going on.
* Writing ABM code is easy. Knowing what to add and running it at scale is hard. **Stand on the shoulders of giants, using data and techniques that already exist**.
* Orient yourself around a key question: what do you want to answer? What do you want to test?
* **Sensitivity analysis** is key towards seeing what would swing the output. Only focus on the big stuff (both inputs & outputs of a model).
* <Probably more stuff here!>

---

# Agenda

1. [What is an agent based model?](#5)
2. [How did we build the model?](#13)
3. [What did we learn?](#19)
4. [**Any questions?**](#27)
