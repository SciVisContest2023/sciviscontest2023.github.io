---
title: 'Data Description'
date: 2019-02-11T19:27:37+10:00
weight: 2
---

###### Summary
This data contains:
- ~50 GB data
- 4 simulation ensemble members
- Each members consists of 50 000 neurons
- Each node collects information about 12 parameters
- The simulation is run for 1,000,000 simulations steps
- Every 100th iteration step is sampled per node

Please find more details in the `readme.md`.

#### In-Detail Description

Four simulations have been run based on neuron locations given by the same underlying human brain data (MEG 146129) provided within the human connectome database [\[ 6 \]]({{< relref "#References and Acknowledgments" >}}) and extended by additional neurons close to existing ones. In the future, we plan on using more accurate models but these are not yet open for free use.
Technical details on the simulation process can be found in [\[ 7 \]]({{< relref "#References and Acknowledgments" >}}) and [\[ 8 \]]({{< relref "#References and Acknowledgments" >}}).
Each simulation produces several text files sorted in their corresponding folder:

| Simulation | Conditions: | Folder Name |
| :--   | :-- | :-- |
| 1 | 0-calcium level at the start and equal target calcium level for all neurons, no existing connections | no-network |
| 2 | Lesion Simulation: Take the connection result of 1 and simulate with 10% neurons inactive (dead). | disable |
| 3 | Learning Simulation: Take the connection result of 1 and simulate learning (electric stimulation) of a specific area | stimulus |
| 4 | Take the connection result of 1 and use different target calcium levels for neurons | calcium |
|   |   |

Some of the simulations have additional files, e.g., `calcium_targets.txt`,`disable.txt`, and `stimulus.txt` that include simulation specific information.  
  
As the simulation can be efficiently run on a cluster of computing nodes, MPI information is included within the files and names by a leading MPI Rank ID.

Each folder has the same setup explained in the following:

###### Network Layout
The base for the simulation is a neuronal network represented by point neurons.
Each neuron has a **Neuron ID** that is unique to its **MPI Rank ID**. 
The combination of both is then used throughout the simulation and data.

The **3D-position** of each neuron can be found in `positions/rank_0_positions.txt`.  
These are fixed and do not change.
This file further contains **labels** explaining which part of the brain the neuron belongs to.
*Note: These areas are set manually and are not based on medical accurate regions within the human brain.* 

Ingoing as well as outgoing connections between nodes are sampled at different simulation steps and written in `network/<MPI_RANK_ID>_step_<STEP>_in_network.txt` and `network/<MPI_RANK_ID>_step_<STEP>_out_network.txt` respectively.

###### Per Node Information
The information about individual nodes is saved to `*.csv` files within `monitors.zip`.  
Their naming convention is as follows: `<MPI_RANK_ID>_<NEURON_ID - 1>.csv`.  
Each **row** within the file contains the per-node information at a sampled **simulation step**.  

The following parameters are listed:

| Variable | Description |
| :--   | :-- |
| Step  | Sampled simulation step|
| Fired | Boolean: Did the neuron fire within the last sample step |
| Fired Fraction  | In Percent: Number of firings since the last sampling |
| x | Electric Activity |
| Secondary Variable  | Inhibition variable used for the firing model of Izhikevich |
| Calcium | Current calcium level|
| Target Calcium | Target calcium level|
| Synaptic Input  | Input electrical activity |
| Background Activity | Background noise electric activity input |
| Grown Axons | Number of currently grown axonal boutons |
| Connected Axons | Number of current outgoing connections |
| Grown Excitatory Dendrites  | Number of currently grown dendrite spines for excitatory connections |
| Connected Excitatory Dendrites  | Number of incoming excitatory connections |
|   |   |

###### Simulation Information

Each simulation run also saves some statistics:

| File Name | Description |
| :--   | :-- |
| `neurons_overview.txt`  | Statistical summary sampled simulation steps |
| `plasticity_changes.txt`  | Summary of creation and deletions of connections |
| `stdcout.txt`  | Program output |
| `timers.txt`  | Timings |
| | |


###### Matlab example for visualizing nodes and connections

This is a small hacked example of how to read node positions, incoming connections, and calcium value and plot the network in Matlab.

The code example below should produce the following outcome:

{{< rawhtml >}}
<img src="/matlab2.png" alt="Matlab Plot" class="matlab">
{{< /rawhtml >}}

```Matlab
% Assuming the correct file locations have been set

%%% First we read in nodes and their locations
fileID = fopen(positionfilelocation,'r');

% Read number of nodes:
numOfNodes = fscanf(fileID,'# %d');
fgetl(fileID);

% Read min max values:
minMaxXYZ = fscanf(fileID,'%*s %*s %*s %f',[3,2]);
fgetl(fileID);
fgetl(fileID);

% Read positions, labels and type
data = textscan(fileID,'%*f %f %f %f %s');
fclose(fileID);

% Data containters
positions = [data{1} data{2} data{3}];
labels = [data{4}];

%%% Now let's read the incoming connections:
fileID = fopen(connectionInfilelocation,'r');
fgetl(fileID);fgetl(fileID);fgetl(fileID);fgetl(fileID);fgetl(fileID);
inconnect = textscan(fileID,'%f %f %f %f %f');
fclose(fileID);

%%% Read Calcium values:
fileID = fopen(calciumfilelocation,'r');
str = fgetl(fileID);
iterations = 0;
calcium = zeros(1,numOfNodes);

i = 1;
while(str ~= -1)
    linedata = strsplit(str,';');
    iterations(i) = sscanf(linedata{1},'#%d');
    calcium(i,:) = str2double(linedata(2:end));
    i = i+1;
    str = fgetl(fileID);
end
fclose(fileID);

%%% Plot Nodes and incoming connections
figure
hold on
axis vis3d
view(60,8)

% plot ingoing connections:
for i=1:size(inconnect{1},1)
    startnodeid = inconnect{2}(i);
    endnodeid = inconnect{4}(i);
    line = [positions(startnodeid,:);positions(endnodeid,:)];
    plot3(line(:,1),line(:,2), line(:,3),'-k')
end

% plot neuron locations
plot3(positions(:,1),positions(:,2),positions(:,3),'.')
```
***Be warned: due to the number of nodes, plotting might be very slow on some machines.***

{{< rawhtml >}}
<div style="height:  20px"></div>
{{< /rawhtml >}} 

----------   

{{< rawhtml >}}
<div style="height:  20px"></div>
{{< /rawhtml >}}