# Using IFC to Determine Where Healthy Materials Should Be a Prioritiy for Improved IEQ in a Building

## Flooring VOC Emissions and Space Concentrations

##### Who can this tool be used by?

- Project managers in the building industry trying to make cost effective decisions while maintaining building health

- Architects focused on designing material and health conscious spaces

- Engineers involved in running building simulations and other analyses

- Anyone working in the built environment space who is interested in improving decision making for material selection

##### What does the script do?

This script runs a loop on through all unique spaces in an IFC file to retrieve data for each space in a building including space name, areas, volumes, and floor covering in a space. The information is then printed for each space or 'room'. Additionally, a loop is run that goes through all spaces to determine the bounding elements. Disciplinary expertise in indoor air quality (IAQ) and ventilation is required to perform an analysis of chemical concentrations in the spaces based on the information extracted from the IFC. A list with off-gassing rates for particular flooring materials has been created and is utilized in calculations for VOC concentrations in the spaces. The emission rate for all flooring in a room can then be determined by multiplying the space area and the specific emission rate corresponding to the floor covering in the space. In this particular script, an assumed ventilation rate was used along with the extracted space volume to determine the air change rate in the space. Once the space emission rate and air change rate are determine, the VOC concentration in a space can be estimated and is printed. Plots and a 'Top 10' list are generated to easily compare VOC 'hotspots' in the building.

###### IFC Input data:

- Room identifier = space_Name

- Room type = space_LongName

- Room area = space_area

- Room volumem = space_volume

- Flooring type = floor_covering

###### External Input data

- Specific Emission Rate (SER) of material

- Ventilation rate

###### The script is as follows:

```
#Overview of the number of rooms with quantaties
if i == spaces_in_model:
    print("All spaces in the model contain quantaties.\n")
else: 
    print(f"\n {i} spaces contain quantaties out of {spaces_in_model} spaces. You may want to add quantaties to the remaining spaces.\n")
```

```
#Merges the space number together with the longname of the space
space_description = [x + ' ' + y for x, y in zip(space_Name, space_LongName)]
```

```
#Wall data  
#Empty variables
wall_space_total = []
matrix_wall_area = []
matrix_wall_material=[]
```

```
#Creates a loop that goes through all the spaces, identifies the boundering elements
#And for the ones that are wall, find the wall material and wall area
#The wall material and area is stored in a matrix, so each wall of a room can be identified with right material
for space in spaces:
    near = space.BoundedBy
    #print("\n\t####{}\n".format(space.Name))
    wall_space_area = []
    wall_material = []
    for objects in near:
        if (objects.RelatedBuildingElement != None):
            if (objects.RelatedBuildingElement.is_a('IfcWall')):
                material = ifcopenshell.util.element.get_material(objects.RelatedBuildingElement)
                if material.is_a("IfcMaterial"):
                    wall_material.append(material.Name)
                if material.is_a("IfcMaterialConstituentSet"):
                    Constituent=material.MaterialConstituents[-1]   #Assumes that the material closest to the room is defined last
                    Constituent_material = Constituent.Material
                    wall_material.append(Constituent_material.Name)
                psets_wall = ifcopenshell.util.element.get_psets(objects.RelatedBuildingElement)
                if "Qto_WallBaseQuantities" in psets_wall:
                    wall_space_area.append(psets_wall["Qto_WallBaseQuantities"]["NetSideArea"])
    matrix_wall_material.append(wall_material)
    matrix_wall_area.append(wall_space_area)
    wall_space_total.append(np.sum(wall_space_area))
```

```
############################### Calculation of VOC concentration 
```

```
#Define SER values (off-gassing values) for known materials
materials = {"Epoxy":25,"Beton":5,"Vinyl":30,"Fliser":0,"Gummi":15}     #ug/h/m2
```

```
#Functions that search for partial matches between dictionary key and searchword
def partial_match(search_word, keys):
    for key in keys:
        if key in search_word:
            return key
    return None
```

```
#Makes a list of SER values corresponding to each floor covering in each space
floor_SER = [materials[partial_match(word, materials.keys())] if partial_match(word, materials.keys()) is not None else 0 for word in floor_covering]
```

```
#Calculates the emission rate
emission_rate_floor = np.multiply(floor_SER,space_area)       #ug/h
```

```
#Defining the ventilation rate to be 0,7 l/s*m2(since unknown in model) and calculates air change rate of the space
ventilation_rate = [0.7 for _ in range(len(space_area))]    #l/s*m2
air_change = np.divide((np.multiply(ventilation_rate,space_area)*(10**(-3))*3600),space_volume)     #h^-1
```

```
#Calculation of VOC concentration and relates VOC concentration to room in a dictionary
VOC_concentration = np.around(np.divide(emission_rate_floor,np.multiply(air_change,space_volume)),3)             #ug/m3
zip_dict = dict(zip(space_description,VOC_concentration))
```

```
########################### Data visualisation #################################
```

```
#Create a dataframe for easier management of all of the data
data = {"Space number":space_Name,"Space type":space_LongName,"Floor area":space_area,
        "Room volume":space_volume,"Floor covering":floor_covering,"VOC concentration":VOC_concentration}
```

```
df = pd.DataFrame(data)
```

```
print(f"The VOC for each room is now calculated. The data for each room can be found in the dataframe df. \n \n {df}")
```

```
#Plot of the concentrations in the rooms
indices = np.where(~np.isnan(VOC_concentration))[0]     #Gets all the indices where there are calculated VOC concentration
```

```
plt.bar(np.array(space_description)[indices], np.array(VOC_concentration)[indices], color ='maroon', width = 0.4)    
```

```
plt.xticks(rotation=90)
plt.title("VOC concentration pr. room")
plt.xlabel("Room name")
plt.ylabel("VOC concentration [ug/m3]")
```

```
print("The VOC concentration is plotted as bar plots in the 'plots' tab")
```

```
#Finds the 'Top 10 worst rooms' in the building
indices10 = np.argsort(np.array(VOC_concentration)[indices])[-10:]
top10_VOC = [np.array(VOC_concentration)[indices][i] for i in indices10]
top10_spaces = [np.array(space_description)[indices][i] for i in indices10]
```

```
print(f"\n The 10 worst spaces in the model are:{top10_spaces}, with the given values:\n")
```

```
for top10_spaces, top10_VOC in zip(top10_spaces, top10_VOC):
    print(f"{top10_spaces:<15}{top10_VOC}")
```

##### Why is this valuable?

The value in this tool is that it provides any constituent in the building design process with an easy tool to perform analysis on the off-gassing of materials in a building based on the IFC file. This means that as long as there is in IFC file with spaces assigned, off-gassing analysis can be conducted from the early design stages to create improved iterations.

##### What needs advacement for this tool to be widely and easily used?

Many iterations and improvements could be made to this tool so that more materials in a building could be evaluated. This is first going to require more extensive off-gas testing of particular materials so that there is a database of input for these calculations. 
