# Using IFC to Determine Where Healthy Materials Should Be a Prioritiy for Improved IEQ in a Building

## Flooring VOC Emissions and Space Concentrations

#### Use case description

The script solves the task of determining VOC concentration for each space in a building model, based on the off-gassing from the materials in the room and their surface area and the air change rate of the room. The amount af VOC gasses in a room should be limited to secure a healthy indoor environment and good indoor air quality, and it is therefore necessary to develop a tool, that can calculate the VOC concentration and determine hotspots in the building to further improve the indoor environment for the occupants.

##### Why is this valuable?

The value in this tool is that it provides any constituent in the building design process with an easy tool to perform analysis on the off-gassing of materials in a building based on the IFC file. This means that as long as there is in IFC file with spaces assigned, off-gassing analysis can be conducted from the early design stages to create improved iterations.

##### Who can this tool be used by?

- Project managers in the building industry trying to make cost effective decisions while maintaining building health

- Architects focused on designing material and health conscious spaces

- Engineers involved in running building simulations and other analyses

- Anyone working in the built environment space who is interested in improving decision making for material selection

##### What does the script do?

This script runs a loop on through all unique spaces in an IFC file to retrieve data for each space in a building including space name, floor- and ceiling areas, volumes, and floor covering in a space. The current script calculates only VOC off-gassing from the floor covering, but would (if expanded) be calculated for all walls, floors, ceilings, and furnitures in the space. The script therefore additionally runs a loop that goes through all spaces to determine the bounding wall elements and their materials, but the information for this is not further utilized, but the necessary information is available. 

Disciplinary expertise in indoor air quality (IAQ) and ventilation is required to perform an analysis and assessment of chemical concentrations in the spaces based on the information extracted from the IFC. Expertise in IAQ and ventilation was needed in order to be able to calculate the VOC concentration in the room based on the emissions and the air exhange rate. A list with off-gassing rates for particular flooring materials has been created and is utilized in calculations for VOC concentrations in the spaces. 

The emission rate for all flooring in a room can then be determined by multiplying the space area and the specific emission rate corresponding to the floor covering in the space. In this particular script, an assumed ventilation rate was used along with the extracted space volume to determine the air change rate in the space requiring expert knowledge of necessary ventilation rates for the spaces and the influence of the air exchange rate. Once the space emission rate and air change rate are determined, the VOC concentration in a space can be estimated and is printed. Plots and a 'Top 10' list are generated to easily compare VOC 'hotspots' in the building.

No other use cases needed to be done before starting the script, and no other use cases are waiting for out script to be complete. 

###### IFC Input data:

- Room identifier = space_Name

- Room type = space_LongName

- Room area = space_area

- Room volume = space_volume

- Flooring type = floor_covering

###### External Input data

- Specific Emission Rate (SER) of material

- Ventilation rate


###### IFC Concepts used
- Property- and quantity sets: Used to retrieve space volume, floor area, ceiling area and floor covering, If the property set had been defined from the quantity set of Qto_SpaceBaseQuantities, Qto_WallBaseQuantities, and Pset_SpaceCoveringRequirements. Had the property set Pset_SpaceThermalLoad been defined the with the property AirExchangeRate been defined, would it be used. 

- IfcMaterial + IfcMaterialConstituentSet: Used to determine the material of the walls of the spaces. The constituentset is further broken down in order to ascribe one material to the given wall. 

- IfcRelSpaceBoundary: An attribute type for the space which is found by utilizing the BoundedBy attribute. This makes it possible to identify the bounding walls of a space. 

##### What needs advacement for this tool to be widely and easily used?

Many iterations and improvements could be made to this tool so that more materials in a building could be evaluated. This is first going to require more extensive off-gas testing of particular materials so that there is a database of input for these calculations. 
