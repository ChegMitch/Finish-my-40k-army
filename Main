import streamlit as st
from itertools import product

st.set_page_config(layout="wide")

st.title("40k Army Builder")

# -----------------------
# INPUT
# -----------------------

target = st.number_input("Remaining Points", min_value=0, value=500)

st.markdown("## Units")

num_units = st.number_input("Number of unit types", 1, 20, 5)

units = []

# -----------------------
# UNIT INPUT LOOP
# -----------------------

for i in range(num_units):
    st.markdown(f"### Unit {i+1}")

    col1, col2, col3 = st.columns(3)
    name = col1.text_input("Name", key=f"name{i}")
    max_count = col2.number_input("Max", min_value=0, value=1, key=f"max{i}")
    multi = col3.checkbox("Multi-size?", key=f"multi{i}")

    if multi:
        sizes_text = st.text_input(
            "Sizes (e.g. 5:95,10:180)",
            key=f"sizes{i}"
        )

        enh = st.checkbox("Enhancement?", key=f"enh{i}")
        enh_cost = st.number_input("Enh Cost", key=f"enhcost{i}")

        if name and sizes_text:
            try:
                parts = sizes_text.split(",")
                for p in parts:
                    size, cost = p.split(":")
                    units.append({
                        "name": f"{name} ({size.strip()})",
                        "cost": int(cost.strip()),
                        "max": max_count,
                        "plus2": 0,
                        "plus3": 0,
                        "enh": enh,
                        "enh_cost": enh_cost
                    })
            except:
                st.warning(f"Invalid sizes format for {name}")

    else:
        col1, col2, col3 = st.columns(3)
        cost = col1.number_input("Base Cost", key=f"cost{i}")
        plus2 = col2.number_input("+2nd", key=f"p2{i}")
        plus3 = col3.number_input("+3rd", key=f"p3{i}")

        col1, col2 = st.columns(2)
        enh = col1.checkbox("Enhancement?", key=f"enh{i}")
        enh_cost = col2.number_input("Enh Cost", key=f"enhcost{i}")

        if name:
            units.append({
                "name": name,
                "cost": cost,
                "max": max_count,
                "plus2": plus2,
                "plus3": plus3,
                "enh": enh,
                "enh_cost": enh_cost
            })

# -----------------------
# COST FUNCTION
# -----------------------

def compute_cost(unit, count):
    total = 0
    for i in range(1, count + 1):
        base = unit["cost"]
        if i == 2:
            base += unit["plus2"]
        elif i >= 3:
            base += unit["plus3"]
        total += base
    return total

# -----------------------
# CALCULATION
# -----------------------

if st.button("Calculate"):

    if not units:
        st.error("No valid units entered")
    else:
        results = []

        ranges = [range(int(u["max"]) + 1) for u in units]

        for counts in product(*ranges):

            enhancement_choices = []
            for count, unit in zip(counts, units):
                if unit["enh"] and count > 0:
                    enhancement_choices.append([0, 1])
                else:
                    enhancement_choices.append([0])

            for enh_flags in product(*enhancement_choices):

                total = 0
                combo = {}

                for count, unit, enh_flag in zip(counts, units, enh_flags):
                    if count == 0:
                        continue

                    cost = compute_cost(unit, count)

                    name = unit["name"]
                    if enh_flag:
                        cost += unit["enh_cost"]
                        name += " [Enhanced]"

                    total += cost
                    combo[name] = count

                if target - 10 <= total <= target:
                    results.append((total, combo))

        results.sort(key=lambda x: x[0], reverse=True)

        if not results:
            st.warning("No combinations found")
        else:
            st.markdown("## Results")

            for total, combo in results[:30]:  # limit output
                text = ", ".join([f"{v}x {k}" for k, v in combo.items()])
                st.write(f"**{total} pts**: {text}")
