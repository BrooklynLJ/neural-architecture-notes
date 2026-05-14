You can do fixed point operations with these.
![[Pasted image 20260513200736.png]]
This covers the case below
$$\begin{equation}
\frac{1}{1+e^{-x}}
\end{equation}$$
However, it doesn't (immediately) cover the functions of the form
$$\begin{equation}
\frac{1}{1-e^{-x}}
\end{equation}$$
But by doing some algebra:
$$\frac{1}{1-e^{-x}}=\frac{e^x}{e^x-1}=\frac12+\frac12\coth\left(\frac{x}{2}\right)$$
coth(x) we can accelerate with a different matlab CORDIC function [**cordictanh(x)**](https://www.mathworks.com/help/fixedpoint/ref/cordictanh.html) , which has an HDL optimized [example](https://www.mathworks.com/help/fixedpoint/ref/hyperbolictangenthdloptimized.html). Their code could possibly be modified into cot, or just inverted at runtime. This would almost certainly be slower to run than the LUT, but could very easily be drafted and iterated on in matlab. 
	*an issue might arise with the above approach because coth has a singularity at zero, however, that would exist in the normal model and I think won't be a region we would actually reach*

There is also a [cordicexp()](https://www.mathworks.com/help/fixedpoint/ref/cordiccexp.html) for the remaining functions.

[Relevant paper](https://doi.org/10.1016/j.jocs.2025.102567) if we wanted to try to reimplement what the matlab gods have done for us 