Lets explain this function in parts
```python
def trace_field_line(r0, get_B, r_min=0.5, r_max=10.0):
    """Trace a magnetic field line starting at position r0 in the field
     provided by the get_B function.

     This function integrates the field line in the positive direction
     until it get either closer than `r_min` to the origin, or outside of `r_max`.
     
     Returns a pandas DataFrame with columns 'time', 'x', 'y', 'z'.
     """
    def rhs(t, x):
        B = get_B(t, x)
        B /= np.linalg.norm(B)
        return B
    
    def outside(t, x):
        return np.linalg.norm(x) - r_max  # stop if r > r_max
    outside.terminal = True
    outside.direction = 1  # only trigger when approaching from inside

    def inside(t, x):
        return np.linalg.norm(x) - r_min  # stop if r < r_min
    inside.terminal = True
    inside.direction = -1  # only trigger when approaching from outside
        
    t = (0., 20.) # integrate up to time 20 -- That's kinda arbitrary, but usually is long enough
    sol = scipy.integrate.solve_ivp(rhs, t, r0, events=[outside, inside], max_step = .2)
    df = pd.DataFrame(np.column_stack((sol.t, sol.y.T)), columns=['time', 'x', 'y', 'z'])
 ```

This function calculates the normalized B field at a point over time to be used to trace the field lines

 First the inputs and outputs
 ```python
 def trace_field_line(r0, get_B, r_min=0.5, r_max=10.0):
 ```

r0 - is the starting point to calculate the field

get_B is your magnetic field

r_min and r_max tell the function when to stop calculating to avoid singularities 
and to save computational time (choose just a window to show field lines)
```python
 def rhs(t, x):
        B = get_B(t, x)
        B /= np.linalg.norm(B)
        return B
```

This function is the right hand side of the ODE and 
normalizes our B vector field since field lines only consider direction

```python
t = (0., 20.) # integrate up to time 20 -- That's kinda arbitrary, but usually is long enough
    sol = scipy.integrate.solve_ivp(rhs, t, r0, events=[outside, inside], max_step = .2)
    df = pd.DataFrame(np.column_stack((sol.t, sol.y.T)), columns=['time', 'x', 'y', 'z'])
```
This chunk integrates the equation $$\frac{d\vec{x}}{dt} = \frac{\vec{B}}{|B|}$$ over our specified time value
in this exmaple right now our fields are independent of time

Finally the function returns a pandas dataframe with time, x, y, z coordinates

```python
def outside(t, x):
        return np.linalg.norm(x) - r_max  # stop if r > r_max
    outside.terminal = True
    outside.direction = 1  # only trigger when approaching from inside

    def inside(t, x):
        return np.linalg.norm(x) - r_min  # stop if r < r_min
    inside.terminal = True
    inside.direction = -1  # only trigger when approaching from outside
```
One last thing is these encode the stopping conditions on the solve_ip routine.
One for r is larger than specified max and the other for r is less than specified max
