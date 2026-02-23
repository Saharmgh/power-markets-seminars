# Answer Key - Power Markets Seminars

## Economic Dispatch (Enhanced) - Exercise Answers

### Exercise 1: Demand exceeds total capacity

```python
# Total available capacity
total_cap = gen_df['existing_cap_mw'].sum()
print(f"Total installed capacity: {total_cap:.1f} MW")
print(f"Setting demand to 5,000 MW (exceeds capacity by {5000 - total_cap:.1f} MW)")
print("="*60)

# Create infeasible demand
loads_infeasible = pd.DataFrame({'demand': [5000]})
result_infeasible = economic_dispatch_single(gen_df, loads_infeasible, var_cf_single)

print(f"\nSolver status: {result_infeasible['status']}")
if result_infeasible['cost'] is None:
    print("\nThe model is INFEASIBLE. There is no way to meet 5,000 MW of demand")
    print("with only {:.0f} MW of capacity. In the real world, this would mean".format(total_cap))
    print("rolling blackouts or emergency imports from neighboring regions.")
    print("\nIn our OPF seminar, we will see how transmission interconnections")
    print("with neighboring regions help prevent this situation.")
```

### Exercise 2: Remove all ramp limits

```python
print("=" * 60)
print("COMPARISON: With vs. Without Ramp Constraints")
print("=" * 60)
print(f"\nCost WITHOUT ramp limits: ${solution_multi['cost']:>15,.2f}")
print(f"Cost WITH ramp limits:    ${solution_ramp['cost']:>15,.2f}")
print(f"Additional cost of ramps: ${solution_ramp['cost'] - solution_multi['cost']:>15,.2f}")
print(f"\nThis {(solution_ramp['cost'] - solution_multi['cost'])/solution_multi['cost']*100:.2f}% cost increase")
print("represents the 'price of inflexibility' -- the cost of engineering")
print("limitations on how fast generators can change output.")
print("\nIn practice, removing ramp limits is unrealistic.")
print("However, more flexible generators (like batteries) can reduce this cost.")
```

### Exercise 3: Pmin > 0 (base solar)

```python
# Test with Pmin = 40% for thermal plants (base solar)
print("Solving with Pmin = 40% of Pmax for all thermal generators...")
solution_pmin = economic_dispatch_with_pmin(gen_df, loads_multi, gen_variable_multi, pmin_fraction=0.4)
print(f"Status: {solution_pmin['status']}")
if solution_pmin['cost']:
    print(f"Cost: ${solution_pmin['cost']:,.2f}")
```

### Exercise 3: Pmin > 0 (high solar)

```python
# Now try with high solar AND Pmin -- this can become infeasible!
print("Solving with Pmin = 40% AND 3500 MW solar...")
print("(This may be infeasible if thermal minimum output + solar > demand at midday)")
print("=" * 60)

solution_pmin_duck = economic_dispatch_with_pmin(gen_df_duck, loads_multi, gen_variable_multi, pmin_fraction=0.4)
print(f"\nStatus: {solution_pmin_duck['status']}")

if solution_pmin_duck['status'] != 'optimal':
    print("\nThe model is INFEASIBLE! This is because:")
    min_thermal = gen_df[gen_df['is_variable'] == False]['existing_cap_mw'].sum() * 0.4
    print(f"  - Minimum thermal output (40%): {min_thermal:.0f} MW")
    print(f"  - Minimum demand: {loads_multi['demand'].min():.0f} MW")
    print(f"  - Plus solar can add up to ~2,000 MW at midday")
    print(f"  - Total minimum supply can exceed demand!")
    print("\nIn reality, this is the 'over-generation' problem that CAISO faces.")
    print("Solutions include: curtailing renewables, exporting power, or shutting")
    print("down thermal units -- which requires unit commitment (next seminar!).")
else:
    print(f"Cost: ${solution_pmin_duck['cost']:,.2f}")
    sol_gen_btm_pmin = process_and_plot(solution_pmin_duck, gen_df, gen_variable_multi, T_period,
                                        'Generation with Pmin=40% and 3500 MW Solar')
```

### Scenario Comparison Table

```python
# Build comparison table
scenarios = {}

# 1. Base case (no ramps)
scenarios['Base (no ramps)'] = solution_multi

# 2. Base case (with ramps)
scenarios['Base (with ramps)'] = solution_ramp

# 3. Duck curve (3500 MW solar, with ramps)
scenarios['3500 MW Solar'] = solution_duck

# 4. Pmin = 40% (base solar)
if solution_pmin['cost'] is not None:
    scenarios['Pmin=40% (base solar)'] = solution_pmin

# Build table rows
rows = []
for name, sol in scenarios.items():
    lmps = sol['lmps']
    row = {
        'Scenario': name,
        'Total Cost ($)': f"${sol['cost']:,.0f}",
        'Avg LMP ($/MWh)': f"${lmps['lmp'].mean():.2f}",
        'Min LMP ($/MWh)': f"${lmps['lmp'].min():.2f}",
        'Max LMP ($/MWh)': f"${lmps['lmp'].max():.2f}",
        'LMP Spread': f"${lmps['lmp'].max() - lmps['lmp'].min():.2f}",
        'Status': sol['status'],
    }
    rows.append(row)

# Add infeasible scenarios
if result_infeasible['cost'] is None:
    rows.append({
        'Scenario': 'Demand > Capacity',
        'Total Cost ($)': 'N/A',
        'Avg LMP ($/MWh)': 'N/A',
        'Min LMP ($/MWh)': 'N/A',
        'Max LMP ($/MWh)': 'N/A',
        'LMP Spread': 'N/A',
        'Status': result_infeasible['status'],
    })

if solution_pmin_duck['cost'] is None:
    rows.append({
        'Scenario': 'Pmin=40% + 3500MW Solar',
        'Total Cost ($)': 'N/A',
        'Avg LMP ($/MWh)': 'N/A',
        'Min LMP ($/MWh)': 'N/A',
        'Max LMP ($/MWh)': 'N/A',
        'LMP Spread': 'N/A',
        'Status': solution_pmin_duck['status'],
    })

comparison_df = pd.DataFrame(rows)
comparison_df
```

## Optimal Power Flow - Exercise Answers

### IEEE 14-bus DC-OPF Solution

```python
def dcopf_ieee(gens, lines, loads):
    """
    Solve DC OPF problem using IEEE test cases.

    Inputs:
        gens  -- DataFrame with generator info
        lines -- DataFrame with transmission line info
        loads -- DataFrame with load info
    """
    model = pyo.ConcreteModel()

    # Enable dual extraction for LMPs
    model.dual = pyo.Suffix(direction=pyo.Suffix.IMPORT)

    # Define sets based on data
    # Set of generator buses
    G = list(gens['connnode'])

    # Set of all nodes
    N = sorted(set(lines['fromnode']).union(set(lines['tonode'])))

    # sets J_i and G_i will be described using dataframe indexing below

    # Define per unit base units for the system
    # used to convert from per unit values to standard unit
    # values (e.g. p.u. power flows to MW/MVA)
    baseMVA = 100  # base MVA is 100 MVA for this system

    # Flow pairs from the lines dataframe (includes both directions)
    flow_pairs = list(zip(lines['fromnode'], lines['tonode']))

    # Decision variables
    model.GEN = pyo.Var(N, within=pyo.NonNegativeReals)  # generation at each node
    # Note: we assume Pmin = 0 for all resources for simplicity here
    model.THETA = pyo.Var(N, within=pyo.Reals)            # voltage phase angle of bus
    model.FLOW = pyo.Var(flow_pairs, within=pyo.Reals)    # flows between connected pairs of nodes

    # Create slack bus with reference angle = 0; use bus 1 with generator
    model.THETA[1].fix(0)

    # Objective function
    model.obj = pyo.Objective(
        expr=sum(
            gens.loc[gens['connnode'] == g, 'c1'].iloc[0] * model.GEN[g]
            for g in G
        ),
        sense=pyo.minimize
    )

    # Supply demand balance constraints
    model.cBalance = pyo.Constraint(N)
    for i in N:
        # Generators at node i
        gens_at_i = list(gens.loc[gens['connnode'] == i, 'connnode'])
        # Loads at node i (note: demand values are NEGATIVE in this dataset)
        loads_at_i = list(loads.loc[loads['connnode'] == i, 'demand'])
        # Lines leaving node i
        lines_from_i = lines.loc[lines['fromnode'] == i]
        tonode_list = list(lines_from_i['tonode'])

        model.cBalance[i] = (
            sum(model.GEN[g] for g in gens_at_i)
            + sum(d for d in loads_at_i)
            == sum(model.FLOW[i, j] for j in tonode_list)
        )

    # Max generation constraint
    model.cMaxGen = pyo.ConstraintList()
    for _, row in gens.iterrows():
        g = row['connnode']
        model.cMaxGen.add(model.GEN[g] <= row['pgmax'])

    # Power flow equations
    # In DCOPF, line flow is a function of voltage angles
    model.cLineFlows = pyo.ConstraintList()
    for l in range(len(lines)):
        fnode = lines.iloc[l]['fromnode']
        tnode = lines.iloc[l]['tonode']
        b = lines.iloc[l]['b']
        model.cLineFlows.add(
            model.FLOW[fnode, tnode]
            == baseMVA * b * (model.THETA[fnode] - model.THETA[tnode])
        )

    # Max line flow constraints
    model.cLineLimits = pyo.ConstraintList()
    for l in range(len(lines)):
        fnode = lines.iloc[l]['fromnode']
        tnode = lines.iloc[l]['tonode']
        model.cLineLimits.add(
            model.FLOW[fnode, tnode] <= lines.iloc[l]['capacity']
        )

    # Solve
    solver = pyo.SolverFactory('appsi_highs')
    result = solver.solve(model, tee=True)

    # Output variables
    generation = pd.DataFrame({
        'node': gens['connnode'].values,
        'gen': [pyo.value(model.GEN[g]) for g in G]
    })

    angles = {i: pyo.value(model.THETA[i]) for i in N}

    flows = pd.DataFrame({
        'fbus': lines['fromnode'].values,
        'tbus': lines['tonode'].values,
        'flow': [
            baseMVA * lines.iloc[l]['b'] * (angles[lines.iloc[l]['fromnode']] - angles[lines.iloc[l]['tonode']])
            for l in range(len(lines))
        ]
    })

    # We output the marginal values of the demand constraints,
    # which will in fact be the prices to deliver power at a given bus.
    prices = pd.DataFrame({
        'node': N,
        'value': [model.dual[model.cBalance[i]] for i in N]
    })

    # Return the solution and objective as a dictionary
    return {
        'generation': generation,
        'angles': angles,
        'flows': flows,
        'prices': prices,
        'cost': pyo.value(model.obj),
        'status': str(result.solver.termination_condition)
    }
```
