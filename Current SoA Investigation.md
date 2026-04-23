## NeuroML:



## GeNN:
GeNN is a GPU-enhanced Neuronal Network simulation environment based on code generation for Nvidia CUDA and AMD HIP.
https://github.com/genn-team/genn/tree/master#

Results from running the cortical microcircuit on my P1 Gen 5 laptop with GPU acceleration:
**Total neurons=77169, total synapses=298624465**

```bash
genn@12901c735a94:/local_home/Research/neuralArchitecture/work/genn/sim_results$ python ../userproject/potjans_microcircuit.py --connectivity-scale 1.0 --save-data --kernel-profiling --procedural-connectivity --duration 10000.0
Max excitatory delay:4.289262ms , max inhibitory delay:2.144631ms
Max dendritic delay slots:43
Creating neuron populations:
	Population 23E: num neurons:20683, external input rate:12800.000000, external weight:0.087800, external DC offset:0.000000
	Population 23I: num neurons:5834, external input rate:12000.000000, external weight:0.087800, external DC offset:0.000000
	Population 4E: num neurons:21915, external input rate:16800.000000, external weight:0.087800, external DC offset:0.000000
	Population 4I: num neurons:5479, external input rate:15200.000000, external weight:0.087800, external DC offset:0.000000
	Population 5E: num neurons:4850, external input rate:16000.000000, external weight:0.087800, external DC offset:0.000000
	Population 5I: num neurons:1065, external input rate:15200.000000, external weight:0.087800, external DC offset:0.000000
	Population 6E: num neurons:14395, external input rate:23200.000000, external weight:0.087800, external DC offset:0.000000
	Population 6I: num neurons:2948, external input rate:16800.000000, external weight:0.087800, external DC offset:0.000000
Creating synapse populations:
	Connection between '23E' and '23E': numConnections=45481917, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '23I' and '23E': numConnections=22316957, meanWeight=-0.351200, weightSD=0.035120, meanDelay=0.750000, delaySD=0.375000
	Connection between '4E' and '23E': numConnections=20248657, meanWeight=0.175600, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '4I' and '23E': numConnections=9658961, meanWeight=-0.351200, weightSD=0.035120, meanDelay=0.750000, delaySD=0.375000
	Connection between '5E' and '23E': numConnections=3288597, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '6E' and '23E': numConnections=2254447, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '23E' and '23I': numConnections=17443660, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '23I' and '23I': numConnections=5017240, meanWeight=-0.351200, weightSD=0.035120, meanDelay=0.750000, delaySD=0.375000
	Connection between '4E' and '23I': numConnections=4101302, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '4I' and '23I': numConnections=1686026, meanWeight=-0.351200, weightSD=0.035120, meanDelay=0.750000, delaySD=0.375000
	Connection between '5E' and '23I': numConnections=2216920, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '6E' and '23I': numConnections=350040, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '23E' and '4E': numConnections=3484485, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '23I' and '4E': numConnections=745110, meanWeight=-0.351200, weightSD=0.035120, meanDelay=0.750000, delaySD=0.375000
	Connection between '4E' and '4E': numConnections=24479055, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '4I' and '4E': numConnections=17400510, meanWeight=-0.351200, weightSD=0.035120, meanDelay=0.750000, delaySD=0.375000
	Connection between '5E' and '4E': numConnections=701280, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '6E' and '4E': numConnections=14617305, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '23E' and '4I': numConnections=8108920, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '23I' and '4I': numConnections=87664, meanWeight=-0.351200, weightSD=0.035120, meanDelay=0.750000, delaySD=0.375000
	Connection between '4E' and '4I': numConnections=9933427, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '4I' and '4I': numConnections=5221487, meanWeight=-0.351200, weightSD=0.035120, meanDelay=0.750000, delaySD=0.375000
	Connection between '5E' and '4I': numConnections=87664, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '6E' and '4I': numConnections=8810232, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '23E' and '5E': numConnections=10611800, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '23I' and '5E': numConnections=1813900, meanWeight=-0.351200, weightSD=0.035120, meanDelay=0.750000, delaySD=0.375000
	Connection between '4E' and '5E': numConnections=5504750, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '4I' and '5E': numConnections=150350, meanWeight=-0.351200, weightSD=0.035120, meanDelay=0.750000, delaySD=0.375000
	Connection between '5E' and '5E': numConnections=2037000, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '5I' and '5E': numConnections=2405600, meanWeight=-0.351200, weightSD=0.035120, meanDelay=0.750000, delaySD=0.375000
	Connection between '6E' and '5E': numConnections=1435600, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '23E' and '5I': numConnections=1240725, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '23I' and '5I': numConnections=169335, meanWeight=-0.351200, weightSD=0.035120, meanDelay=0.750000, delaySD=0.375000
	Connection between '4E' and '5I': numConnections=607050, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '4I' and '5I': numConnections=12780, meanWeight=-0.351200, weightSD=0.035120, meanDelay=0.750000, delaySD=0.375000
	Connection between '5E' and '5I': numConnections=319500, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '5I' and '5I': numConnections=430260, meanWeight=-0.351200, weightSD=0.035120, meanDelay=0.750000, delaySD=0.375000
	Connection between '6E' and '5I': numConnections=132060, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '23E' and '6E': numConnections=4678375, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '23I' and '6E': numConnections=547010, meanWeight=-0.351200, weightSD=0.035120, meanDelay=0.750000, delaySD=0.375000
	Connection between '4E' and '6E': numConnections=6722465, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '4I' and '6E': numConnections=1309945, meanWeight=-0.351200, weightSD=0.035120, meanDelay=0.750000, delaySD=0.375000
	Connection between '5E' and '6E': numConnections=4102575, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '5I' and '6E': numConnections=302295, meanWeight=-0.351200, weightSD=0.035120, meanDelay=0.750000, delaySD=0.375000
	Connection between '6E' and '6E': numConnections=8363495, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '6I' and '6E': numConnections=10825040, meanWeight=-0.351200, weightSD=0.035120, meanDelay=0.750000, delaySD=0.375000
	Connection between '23E' and '6I': numConnections=2258168, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '23I' and '6I': numConnections=14740, meanWeight=-0.351200, weightSD=0.035120, meanDelay=0.750000, delaySD=0.375000
	Connection between '4E' and '6I': numConnections=218152, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '4I' and '6I': numConnections=5896, meanWeight=-0.351200, weightSD=0.035120, meanDelay=0.750000, delaySD=0.375000
	Connection between '5E' and '6I': numConnections=400928, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '5I' and '6I': numConnections=23584, meanWeight=-0.351200, weightSD=0.035120, meanDelay=0.750000, delaySD=0.375000
	Connection between '6E' and '6I': numConnections=2886092, meanWeight=0.087800, weightSD=0.008780, meanDelay=1.500000, delaySD=0.750000
	Connection between '6I' and '6I': numConnections=1353132, meanWeight=-0.351200, weightSD=0.035120, meanDelay=0.750000, delaySD=0.375000
Total neurons=77169, total synapses=298624465
Building Model
make: Entering directory '/local_home/Research/neuralArchitecture/work/genn/sim_results/potjans_microcircuit_CODE'
make: Leaving directory '/local_home/Research/neuralArchitecture/work/genn/sim_results/potjans_microcircuit_CODE'
Loading Model
Simulating
100%
200%
300%
400%
500%
600%
700%
800%
900%
1000%
Timing:
	Simulation:111464.877075
	Init:3.502080
	Sparse init:0.000000
	Neuron simulation:7153.258573
	Synapse simulation:103046.639075
```
## neuroAIx:
According to neuroAIx's paper
![[Pasted image 20260416153541.png]]
on their rack of 35 [NetFPGA-SUMEs](https://digilent.com/reference/programmable-logic/netfpga-sume/start) which have roughly 10x the resources in all categories as the arty-a7-100t.
![[https___www.frontiersin.org_files_Articles_1144143_xml-images_fncom-17-1144143-g0002.webp]]
According to their paper "*models with up to 89,600 neurons can be handled by the cluster of 35 FPGAs*". In their architecture, each FPGA handles 10 workers which each manage 255 LIF neurons. Their primary limitation is "the available BRAM which is used for ... ring buffers".
