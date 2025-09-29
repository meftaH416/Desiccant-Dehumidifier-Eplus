<!-- <p style="text-align: justify; font-size:12px; color:black; font-weight:normal> -->
<p>

## Creating Dehumidification in Openstudio
The OSM file name "CDD-watercoil.osm" is created.
In order to control the Humidity we need to add Coil:Cooling:Water into Air Loop HVAC.
The following systems are added in HVAC Tab.
1. An empty air loop with Fan, Coil:Cooling:Water, Gast Heating Coil along with SetPointManager:SingleZone:Reheat.
Add Air Terminal with Reheat Electricity. Add SetPointManager:SingleZone:Humidity:Maximum to outlet node of
cooling water coil.
2. A plant Loop with Pump, Chiller Electric, Set Point Manager Schedule, and the same Coil:Cooling:Water.
Add one bypass adiabatic pipe to each of Chiller and Coil. 
3. Run the model and check the values.
4. Take the IDF file created from OSM. Change control variable of Controller:WaterCoil to TemperatureAndHumidity
from Temperature. Change Timestep to 1 minute. 

## Creating Desiccant Dehumidifier NoFan in Openstudio within AirLoopHVAC system

Editing the IDF to add Desiccant Dehumidifiers
Make sure that control variable of Controller:WaterCoil is Temperature. 

### 1. Edit Branch of AirLoopHVAC.
- Add "Dehumidifier:Desiccant:NoFans as components 11.
- Add "Component 2 Outlet Node" to Desiccant Process Air Inlet Node
- Add "Process Air Outlet Node" Desiccant object outlet 
- Add "Process Air Outlet Node" to Component 3 (Coil:Cooling:Water) Inlet Node
<pre>
Branch, 
    Air Loop HVAC Main Branch,  !- Name  
    ,                        !- Pressure Drop Curve Name 
    AirLoopHVAC:OutdoorAirSystem,  !- Component 1 Object Type 
    OA System,               !- Component 1 Name
    Sup Inlet Node,          !- Component 1 Inlet Node Name
    Mixwd Air Node,          !- Component 1 Outlet Node Name
    Fan:VariableVolume,      !- Component 2 Object Type
    Var Spd Fan,             !- Component 2 Name 
    Mixwd Air Node,          !- Component 2 Inlet Node Name
    Cool Inlet Node,         !- Component 2 Outlet Node Name
    Dehumidifier:Desiccant:NoFans,  !- ***Component 11 Object Type
    Desiccant 1,             !- ***Component 11 Name
    Cool Inlet Node,          !- ***Component 11 Inlet Node Name 
    Process Air Outlet Node,         !- ***Component 11 Outlet Node Name 
    Coil:Cooling:Water,      !- Component 3 Object Type 
    CHW Clg Coil,            !- Component 3 Name 
    Process Air Outlet Node,         !- ***Component 3 Inlet Node Name 
    Cool Outlet Node,        !- Component 3 Outlet Node Name 
    Coil:Heating:Fuel,       !- Component 4 Object Type 
    Gas Htg Coil,            !- Component 4 Name 
    Cool Outlet Node,        !- Component 4 Inlet Node Name 
    Sup Outlet Node;         !- Component 4 Outlet Node Name 
</pre>
### 2. Desiccant dehumidifier object added
- Add the "Component 2 Outlet Node" from Branch to "Process Air Inlet Node"   
<pre>
Dehumidifier:Desiccant:NoFans, 
    Desiccant 1,             !- Name 
    Always On,               !- Availability Schedule Name 
    Cool Inlet Node,           !- Process Air Inlet Node Name 
    Process Air Outlet Node, !- Process Air Outlet Node Name 
    Regen Coil Out Node,     !- Regeneration Air Inlet Node Name 
    Regeneration Fan Inlet Node,  !- Regeneration Fan Inlet Node Name 
    LeavingMaximumHumidityRatioSetpoint,  !- Control Type 
    0.0074,                    !- Leaving Maximum Humidity Ratio Setpoint {kgWater/kgDryAir} 
    0.35,                     !- Nominal Process Air Flow Rate {m3/s} 
    2.5,                     !- Nominal Process Air Velocity {m/s} 
    10,                      !- Rotor Power {W}
    Coil:Heating:Fuel,       !- Regeneration Coil Object Type 
    Desiccant Regen Coil,    !- Regeneration Coil Name
    Fan:VariableVolume,      !- Regeneration Fan Object Type 
    Desiccant Regen Fan,     !- Regeneration Fan Name 
    DEFAULT;                 !- Performance Model Type
</pre>
### 3. Remove the old OutdoorAir:NodeList and add the following components
- Create new OutdoorAir:NodeList with name OutsideAirInletNodes (user defined name)
- Add "Process Air Inlet Node Name" and "Regeneration Fan Inlet Node Name" to this NodeList  
<pre>
OutdoorAir:NodeList, 
    OutsideAirInletNodes;    !- Node or NodeList Name 1  

NodeList,
   OutsideAirInletNodes,               !- Name 
   OA Inlet Node,                      !- Node 1 Name
   Regeneration Fan Inlet Node;        !- Node 2 Name
</pre> 
### 4. Add Heating coil
- Add "Air Outlet Node" of the "Desiccant Regen Fan" (Fan:VariableVolume) "Air Inlet Node" to this heating coil.  
- Add "Process Air Outlet Node" of Dehumidifier:Desiccant as "Air Outlet Node" to heating coil  
<pre>
Coil:Heating:Fuel, 
    Desiccant Regen Coil,    !- Name 
    Always On,               !- Availability Schedule Name 
    NaturalGas,              !- Fuel Type 
    0.80,                    !- Burner Efficiency 
    40000,                  !- Nominal Capacity {W}
    Regen Fan Outlet Node,   !- Air Inlet Node Name 
    Regen Coil Out Node;     !- Air Outlet Node Name
</pre>
### 5. Add Fan object 
- Add "Regeneration Fan Inlet Node" from Dehumidifier:Desiccant:NoFans to Air Inlet Node
- Add "Air Inlet Node" from "Desiccant Regen Coil" object to "Air Outlet Node" of "Desiccant Regen Fan"  
<pre>
Fan:VariableVolume,
    Desiccant Regen Fan,     !- Name 
    Always On,               !- Availability Schedule Name 
    0.7,                     !- Fan Total Efficiency  
    700.0,                   !- Pressure Rise {Pa}  
    0.35,                     !- Maximum Flow Rate {m3/s}  
    FixedFlowRate,           !- Fan Power Minimum Flow Rate Input Method  
    ,                        !- Fan Power Minimum Flow Fraction  
    0.0,                     !- Fan Power Minimum Air Flow Rate {m3/s}  
    0.9,                     !- Motor Efficiency  
    1.0,                     !- Motor In Airstream Fraction  
    0,                       !- Fan Power Coefficient 1  
    1,                       !- Fan Power Coefficient 2  
    0,                       !- Fan Power Coefficient 3  
    0,                       !- Fan Power Coefficient 4  
    0,                       !- Fan Power Coefficient 5  
    Regeneration Fan Inlet Node,  !- Air Inlet Node Name  
    Regen Fan Outlet Node;   !- Air Outlet Node Name  
</pre>
### 6. Add Avaiabability schedule name 
- The schedule object "Always On" already exists in this IDF.

### 7. Replace "Air Inlet Node" with "Process Air Outlet Node" of Desiccant 
<pre>
Coil:Cooling:Water,  
    CHW Clg Coil,            !- Name  
    Always On Discrete hvac_library,  !- Availability Schedule Name  
    Autosize,                !- Design Water Flow Rate {m3/s}  
    Autosize,                !- Design Air Flow Rate {m3/s}  
    Autosize,                !- Design Inlet Water Temperature {C}  
    Autosize,                !- Design Inlet Air Temperature {C}  
    Autosize,                !- Design Outlet Air Temperature {C}  
    Autosize,                !- Design Inlet Air Humidity Ratio {kgWater/kgDryAir}  
    Autosize,                !- Design Outlet Air Humidity Ratio {kgWater/kgDryAir}  
    Cooling Water Inlet Node,!- Water Inlet Node Name  
    Cooling Water Outlet Node,  !- Water Outlet Node Name  
    Process Air Outlet Node,         !- Air Inlet Node Name  
    Cool Outlet Node,        !- Air Outlet Node Name  
    SimpleAnalysis,          !- Type of Analysis  
    CrossFlow;               !- Heat Exchanger Configuration  
</pre>
### 8. Replace "Setpoint Node or NodeList Name" with "Process Air Outlet Node"
- Also edited the name to "Process Air Outlet Node OS Default SPM" from "Coil Inlet Node OS Default SPM"
<pre>
SetpointManager:SingleZone:Reheat,  
    Process Air Outlet Node OS Default SPM,  !- Name  
    Temperature,             !- Control Variable  
    6,                       !- Minimum Supply Air Temperature {C}  
    82,                      !- Maximum Supply Air Temperature {C}  
    wearhouse,               !- Control Zone Name  
    wearhouse_1_a8047464 Zone Air Node,  !- Zone Node Name  
    Air Terminal Outlet Node,!- Zone Inlet Node Name  
    Process Air Outlet Node;         !- Setpoint Node or NodeList Name  
</pre>

## Creating Desiccant Dehumidifier NoFan in Openstudio to OutdoorAirSystem
Editing the IDF to add Desiccant Dehumidifiers

1. Edit "AirLoopHVAC:OutdoorAirSystem:EquipmentList".
- Add "Dehumidifier:Desiccant:NoFans" and "HeatExchanger:AirToAir:FlatPlate" as components
<pre>
AirLoopHVAC:OutdoorAirSystem:EquipmentList,  
  Air Loop HVAC Outdoor Air System 1 Equipment List, !- Name  
  Dehumidifier:Desiccant:NoFans,          !- Component 1 Object Type  
  Desiccant 1,                            !- Component 1 Name  
  HeatExchanger:AirToAir:FlatPlate,       !- Component 2 Object Type  
  OA Heat Recovery 1,                     !- Component 2 Name  
  OutdoorAir:Mixer,                       !- Component Object Type 3  
  Air Loop HVAC Outdoor Air System 1 Outdoor Air Mixer; !- Component Name 3  
</pre>pre>
2. Desiccant dehumidifier object added
- Add the OutdoorAirSystem Inlet Node as Process Air Inlet Node.
<pre>
Dehumidifier:Desiccant:NoFans,  
  Desiccant 1,                                 !- Name  
  Always On Discrete hvac_library,             !- Availability Schedule Name  
  OA Inlet Node,                               !- Process Air Inlet Node Name  
  Process Air Outlet Node,                     !- Process Air Outlet Node Name  
  Regen Coil Out Node,                         !- Regeneration Air Inlet Node Name  
  Regeneration Fan Inlet Node,                 !- Regeneration Fan Inlet Node Name  
  LeavingMaximumHumidityRatioSetpoint,         !- Control Type  
  0.01,                                       !- Leaving Maximum Humidity Ratio Setpoint {kgWater/kgDryAir}  
  1.5,                                         !- Nominal Process Air Flow Rate {m3/s}  
  2.5,                                         !- Nominal Process Air Velocity {m/s}  
  10,                                          !- Rotor Power {W}  
  Coil:Heating:Fuel,                           !- Regeneration Coil Object Type  
  Desiccant Regen Coil,                        !- Regeneration Coil Name  
  Fan:VariableVolume,                          !- Regeneration Fan Object Type  
  Desiccant Regen Fan,                         !- Regeneration Fan Name  
  DEFAULT;                                     !- Performance Model Type  
</pre>
3. Remove the old OutdoorAir:NodeList and add the following components
- Create new OutdoorAir:NodeList with name OutsideAirInletNodes (user defined name)
- Add "Process Air Inlet Node Name" and "Regeneration Fan Inlet Node Name" to this NodeList
<pre>
OutdoorAir:NodeList,  
    OutsideAirInletNodes;    !- Node or NodeList Name 1 
NodeList,  
   OutsideAirInletNodes,               !- Name  
   OA Inlet Node,                      !- Node 1 Name  
   Regeneration Fan Inlet Node;        !- Node 2 Name  
</pre>
4. Add Heating coil
- Add "Air Outlet Node" of the "Desiccant Regen Fan" (Fan:VariableVolume) "Air Inlet Node" to this heating coil.  
- Add "Process Air Outlet Node" of Dehumidifier:Desiccant as "Air Outlet Node" to heating coil
<pre>
Coil:Heating:Fuel,  
    Desiccant Regen Coil,    !- Name  
    Always On Discrete hvac_library,    !- Availability Schedule Name  
    NaturalGas,              !- Fuel Type  
    0.80,                    !- Burner Efficiency  
    135000,                  !- Nominal Capacity {W}  
    Regen Fan Outlet Node,   !- Air Inlet Node Name  
    Regen Coil Out Node;     !- Air Outlet Node Name  
</pre>
5. Add Fan object 
- Add "Regeneration Fan Inlet Node" from Dehumidifier:Desiccant:NoFans to Air Inlet Node
- Add "Air Inlet Node" from "Desiccant Regen Coil" object to "Air Outlet Node" of "Desiccant Regen Fan"
<pre>
Fan:VariableVolume,  
    Desiccant Regen Fan,     !- Name  
    Always On Discrete hvac_library,    !- Availability Schedule Name  
    0.7,                     !- Fan Total Efficiency  
    700.0,                   !- Pressure Rise {Pa}  
    1.5,                     !- Maximum Flow Rate {m3/s}  
    FixedFlowRate,           !- Fan Power Minimum Flow Rate Input Method  
    ,                        !- Fan Power Minimum Flow Fraction  
    0.0,                     !- Fan Power Minimum Air Flow Rate {m3/s}  
    0.9,                     !- Motor Efficiency  
    1.0,                     !- Motor In Airstream Fraction  
    0,                       !- Fan Power Coefficient 1  
    1,                       !- Fan Power Coefficient 2  
    0,                       !- Fan Power Coefficient 3  
    0,                       !- Fan Power Coefficient 4  
    0,                       !- Fan Power Coefficient 5  
    Regeneration Fan Inlet Node,         !- Air Inlet Node Name  
    Regen Fan Outlet Node;   !- Air Outlet Node Name  
</pre>

6. Add Heat Exchanger 
- Add "Process Air Outlet Node" of Dehumidifier:Desiccant:NoFans to "Supply Air Inlet Node" of this object
- Add "Relief Air Outlet Node" of Controller:OutdoorAir object to "Secondary Air Inlet Node" of this object 
- Other Node names are user defined. 
<pre>
HeatExchanger:AirToAir:FlatPlate,  
    OA Heat Recovery 1,      !- Name  
    Always On Discrete hvac_library,    !- Availability Schedule Name  
    CounterFlow,             !- Flow Arrangement Type  
    Yes,                     !- Economizer Lockout  
    1.0,                     !- Ratio of Supply to Secondary hA Values  
    1.5,                       !- Nominal Supply Air Flow Rate {m3/s}  
    5.0,                     !- Nominal Supply Air Inlet Temperature {C}  
    15.0,                    !- Nominal Supply Air Outlet Temperature {C}  
    1.5,                       !- Nominal Secondary Air Flow Rate {m3/s}  
    20.0,                    !- Nominal Secondary Air Inlet Temperature {C}  
    0.0,                     !- Nominal Electric Power {W}  
    Process Air Outlet Node, !- Supply Air Inlet Node Name  
    Heat Recovery Outlet Node,           !- Supply Air Outlet Node Name  
    OA Relief Node,          !- Secondary Air Inlet Node Name  
    Heat Recovery Secondary Outlet Node;  !- Secondary Air Outlet Node Name  	
</pre>

7. Add Avaiabability schedule name 
- The schedule object "Always On Discrete hvac_library" already exists in this IDF.

8. Edit OutdoorAir:Mixer object
- Replace "Outdoor Air Stream Node" with "Supply Air Outlet Node" of HeatExchanger:AirToAir:FlatPlate object
<pre>
OutdoorAir:Mixer,  
  Air Loop HVAC Outdoor Air System 1 Outdoor Air Mixer, !- Name  
  Mixed Air Node,                         !- Mixed Air Node Name  
  Heat Recovery Outlet Node,              !- Outdoor Air Stream Node Name  
  OA Relief Node,                         !- Relief Air Stream Node Name  
  Sup Inlet Node;                         !- Return Air Stream Node Name  
</pre>

</p>
