---
title: 'Data Description'
date: 2019-02-11T19:27:37+10:00
weight: 2
---

The data contains:
- Number of ensemble members
- Number of neurons
- number of connections
- calcium information
- simulation statistics
- ...

The simulation produces a number of text files.
The data consists of different simulation runs each saved to their own folder.
  
As the simulation can be efficiently run on a cluster of computing nodes, MPI information is included within the files and names by a leading MPI thread ID, which can be ignored.

Each folder has the same setup explained in the following:

###### Network Layout
The base for the simulation is a neural network represented by point neurons.
Each neuron has a unique **ID** that is used throughout the simulation.

The **3D-position** of each neuron can be found in `positions/rank_0_positions.txt`  
This file further contains **labels** explaining which part of the brain the neuron belongs to as well as the **neuron type**. (Is this type always ex?)

Ingoing as well as outgoing connections between nodes at the start of the simulation are written in `network/rank_0_in_network.txt` and `network/rank_0_out_network.txt` respectively.

###### Per Node Information

The information about individual nodes is saved to `*.csv` files.  
Their naming convention is as follow: `0_(ID - 1).csv`.  
Each **row** within the file contains the per node information at a sampled **simulation step**.  

The following parameters are listed:

| Variable | Description |
| :--   | :-- |
| Step  | Sampled simulation step|
| Fired | Boolean: Did the neuron fire within the last sample step |
| Fired Fraction  | ? |
| x | ?|
| Secondary Variable  | ?|
| Calcium | Current calcium level|
| Target Calcium | Target calcium level|
| Synaptic Input  | Input electrical activity |
| Background Activity | ?|
| Grown Axons | ?|
| Connected Axons | ?|
| Grown Excitatory Dendrites  | ?|
| Connected Excitatory Dendrites  | ?|
| Grown Inhibitory Dendrites  | ?|
| Connected Inhibitory Dendrites  | ?|

### Matlab example for visualizing nodes and connections

This is a small hacked example of how to read node positions, incoming connections and calcium value and plot the network in Matlab.

The code example below should produce the following outcome:

{{< rawhtml >}}
<img src="/matlab.png" alt="Matlab Plot" class="matlab">
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
data = textscan(fileID,'%*f %f %f %f %s %s');
fclose(fileID);

% Data containters
positions = [data{1} data{2} data{3}];
labels = [data{4}];
type = [data{5}];

%%% Now let's read the incoming connections:
fileID = fopen(connectionInfilelocation,'r');
fgetl(fileID);fgetl(fileID);fgetl(fileID);fgetl(fileID);
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