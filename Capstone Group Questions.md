* Hodkin Huxley Formalism, actually simulates the biology
* LIF is just a comparator and a capacitor (easily simulatable on a GPU)
* test

![[Pasted image 20260501130743.png]]

- (T) - What exactly is being computed in the synaptic module and how?
- (T) - Are the ionic currents being computed in the RISC core? 
- (T) How is the LUT going to look like when it eventually gets fabricated? Is it going to be in SRAM?
- (T) - Why do we need a real time OS? Is it just for a file system?
- (T) -Is networking the main future bottleneck? What are some other bottlenecks?
- (B) Why is the github repo so big (partially my fault)
- (B) are we trying to accelerate simulation of biological models, or accelerate computation of machine learning models
- (B) why are we focused on tapeout compatibility before seeing viability of our general setup (riscv core & DMA synaptic module) on FPGA?
- (B) Have previous groups proved that this approach is viable on FPGA?
- (B) Do modern FPGAs offer enough speedup that they would be viable as an end solution instead of fabbing an ASIC?
- (B) Would this be a high enough volume to justify tapeout/ASIC?
- (B) How much usage is devoted specifically to the accelerators & risc core? (do could we just demo a connection of 4 Neurons on 1 FPGA?)
- (B) If every ASIC needs a network connection, wont that just become the new (very significant) bottleneck for simulation speed?
- (B) Why do we care about writing to an SD card instead of something more simple like outputting over uart to a host PC? (or using the micropython debugger)
- (B) what is the micropython MCU used for?
- (B) Have you tried programming for a laptop with a modern NPU? (accelerated sigmoid)
	- Which NPUs accelerate sigmoid computation? 
	- Has apple silicon, google coral, or intel loihi already produced a product that accomplishes this at reasonable cost?
- (B) Have we looked into just doing 32bit integer arithmetic instead of floats? the roundoff would be pretty similar if we limited the min and max voltage range to like 150mV
	- What precision are we trying to achieve in the simulation/computation, (is this an open question in neuroscience?)
- (B) Are we trying to implement learning/neuroplasticity?
- (B) Do we have a particular benchmark we are atrying to achieve?
	- Cortical microcircuit?
	- Model learns to accomplish a task?
	- Emulates leech circulatory system neurons
- (B) Where is flexibility desired? 
	- Network connections level
	- Simulation model type
		- neuroml compatibility? :/
	- Size of network
- (B) Do we have any speed comparisons of different numerical DE solvers in FPGA? 
	- Are we using fixed time?
	- why?
- (B) Why are we designing an ASIC before evaluating which algorithms offer reasonable tradeoff between compute time/accuracy when hardware accelerated?
	- Has this been looked at?
- (B) With Efabless having shut down(?), is there a path forward with this asic design
	- Is caravel vendor/node specific?


**1 off question to ponder:**
* if you include the propagation delay that would exist in cells, and can guarantee transmission time between nodes in your simulation, and that transmission time is significantly less than the propagation delay, then you could do event based managing, and variable timesteps across different nodes, and still be time synchronized.