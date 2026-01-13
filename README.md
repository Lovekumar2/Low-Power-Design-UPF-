# Low-Power-Design-UPF
# UPF Power State Table for Multi-Domain Design

This README documents the **Unified Power Format (UPF)** setup for a design with three power domains:

*   **PD\_CPU** (CPU block) → `VDD_C = 1.0V`
*   **PD\_VIDEO** (Video block) → `VDD_V = 1.2V`
*   **PD\_DISPLAY** (Display block) → `VDD_D = 0.8V`

All domains share a common ground (`VSS`).

***

## Power Domains and States

| Domain      | Supply | Voltage |
| ----------- | ------ | ------- |
| PD\_CPU     | VDD\_C | 1.0 V   |
| PD\_VIDEO   | VDD\_V | 1.2 V   |
| PD\_DISPLAY | VDD\_D | 0.8 V   |

### Valid Power States

| State   | CPU | Video | Display |
| ------- | --- | ----- | ------- |
| ALLOFF  | OFF | OFF   | OFF     |
| LOWP\_1 | ON  | OFF   | OFF     |
| LOWP\_2 | ON  | ON    | OFF     |
| ALLON   | ON  | ON    | ON      |

***

## UPF Commands

```tcl
# =========================================================
# 1. Create Power Domains
# =========================================================
create_power_domain PD_CPU    -elements {CPU}
create_power_domain PD_VIDEO  -elements {Video}
create_power_domain PD_DISPLAY -elements {Display}

# =========================================================
# 2. Create Supply Ports
# =========================================================
create_supply_port VDD_C -domain PD_CPU
create_supply_port VSS_C -domain PD_CPU

create_supply_port VDD_V -domain PD_VIDEO
create_supply_port VSS_V -domain PD_VIDEO

create_supply_port VDD_D -domain PD_DISPLAY
create_supply_port VSS_D -domain PD_DISPLAY

# =========================================================
# 3. Create Supply Nets
# =========================================================
create_supply_net VDD_C_NET
create_supply_net VDD_V_NET
create_supply_net VDD_D_NET
create_supply_net VSS_NET

# =========================================================
# 4. Connect Supply Nets to Ports
# =========================================================
connect_supply_net VDD_C_NET -ports VDD_C
connect_supply_net VDD_V_NET -ports VDD_V
connect_supply_net VDD_D_NET -ports VDD_D
connect_supply_net VSS_NET   -ports {VSS_C VSS_V VSS_D}

# =========================================================
# 5. Define Supply Sets
# =========================================================
create_supply_set SS_CPU    -supplies {VDD_C_NET VSS_NET}
create_supply_set SS_VIDEO  -supplies {VDD_V_NET VSS_NET}
create_supply_set SS_DISPLAY -supplies {VDD_D_NET VSS_NET}

# =========================================================
# 6. Add Port States (ON/OFF)
# =========================================================
add_port_state VDD_C -state {C_OFF 0.0} -state {C_ON 1.0}
add_port_state VDD_V -state {V_OFF 0.0} -state {V_ON 1.2}
add_port_state VDD_D -state {D_OFF 0.0} -state {D_ON 0.8}

# =========================================================
# 7. Create Power State Table
# =========================================================
create_power_state_table PST_TOP \
    -domain {PD_CPU PD_VIDEO PD_DISPLAY} \
    -state {
        ALLOFF  {PD_CPU C_OFF PD_VIDEO V_OFF PD_DISPLAY D_OFF}
        LOWP_1  {PD_CPU C_ON  PD_VIDEO V_OFF PD_DISPLAY D_OFF}
        LOWP_2  {PD_CPU C_ON  PD_VIDEO V_ON  PD_DISPLAY D_OFF}
        ALLON   {PD_CPU C_ON  PD_VIDEO V_ON  PD_DISPLAY D_ON}
    }
```

***

## Diagram

    +----------------+      +----------------+      +----------------+
    |    PD_CPU      |      |   PD_VIDEO     |      |  PD_DISPLAY    |
    |  VDD_C (1.0V)  |      | VDD_V (1.2V)  |      | VDD_D (0.8V)   |
    |      VSS       |      |      VSS      |      |      VSS       |
    +----------------+      +----------------+      +----------------+
           |                       |                       |
           +--------- VSS_NET -----+-----------------------+

***

