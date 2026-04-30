Matlab simulation using just numerical integration:
![[Pasted image 20260430091731.png]]

No constraints default ode15s (just using start and end time):
![[Pasted image 20260430091955.png]]
Resulting difference:
![[Pasted image 20260430091930.png]]

Forced to use the **same timesteps as the numerical integration**, and used:
`options = odeset('RelTol',1e-6, 'AbsTol', 1e-8);`
![[Pasted image 20260430092023.png]]
![[Pasted image 20260430092507.png]]

Allowed variable timestep to manage timesteps, but still enforced:
`options = odeset('RelTol',1e-6, 'AbsTol', 1e-8);`
![[Pasted image 20260430092721.png]]
![[Pasted image 20260430092732.png]]