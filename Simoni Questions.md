* Hodkin Huxley Formalism, actually simulates the biology
* LIF is just a comparator and a capacitor (easily simulatable on a GPU)
* test


**1 off question to ponder:**
* if you include the propagation delay that would exist in cells, and can guarantee transmission time between nodes in your simulation, and that transmission time is significantly less than the propagation delay, then you could do event based managing, and variable timesteps across different nodes, and still be time synchronized.