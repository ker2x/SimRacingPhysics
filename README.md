# A bunch of stuff to work on car simulation

That's quite infortunate but we'll need Math. A lot of it.
And I'm not a mathematician.

So, first thing first: [Math& for Sim Racers](docs/Math.md).
- You'll have to read it (and I'll have to write it)
- The guide assumes you have the math knowledge of a 16yo (all 4 operations, fraction, basic equation stuff, exponent, trigonometry)
- You don't need to know how to solve anything, you just need to be able to read it and understand some of it. (or at least have some intuition about the meaning)

Next, we'll code stuff: [Coding]()

## Physic note

- Tyre physic is HARD. This will probably involve the most work.
- Suspension might be a slight problem as well, but not as bad as tyre.
- Car geometry ? mostly tyre related

## Units & conversion

All SI, thank you. seconds, meters, and so on.

- Meter: $m$
- Second: $s$
- Time: $T$ or $t$
  - delta time: the time difference between two time (eg: how much time elapsed between two frames)
    - $\Delta T$ 
    - $\frac{dx}{dt}$ (Leibniz's notation)
    - $\dot{x}$ (Newton Notation. just don't)
- Acceleration: $m/s^2$
- Gravity: $g \approx 9.8 m/s^2$ (9.8 will be used as a fixed value in the simulation)
- Speed: $m/s$ or $m \cdot s^-1$
  - kilometer per hour: $km/h$ (SI) or $kph$ (non SI abbreviation)
    - $m/s$ to $km/h$ = $m/s * 3.6$ ($1 m/s = 3.6km/h$)
    - $km/h$ to $m/s$ = $km/h * \frac{1000}{3600}$  $\approx$ $km/h * 0.2778$ ($1km/h \approx 0.2778 m/s$)
- radian ($rad$ or $\theta$)
- Newton ($N$) : 1 $kg \cdot m/s^2$
- Kilogram: $kg$
  - newton ($N$) to kgf ($kgf$): $\frac{N}{g}$ or  ($1N \approx  0,101972 kgf$)
  - kgf to newton: $kgf * g$ ($1kgf = 9.8N$)
- Angular velocity: $\omega = \frac{\theta}{t}$ = $rad/s$ = $rad \cdot s^-1$
- Force: $F=mass * acceleration$ (or $F=m \cdot a$)
- Torque: $\tau$ = radius * Force * sin(angle between arm and lever) (ot $\tau = rF \sin{\theta}$)
- Power & Work
  - Watt
  - horsepower

## Use iRacing as a reference (or better, if possible)

- I need to create a small telemetry dashboard if possible. (otherwise, just use Pi Toolbox)
  - use it as "ground truth" to validate the model
- Need to pick a reference car
  - mx5
    - available in many simulator
    - underpowered
    - road tyre
    - documentation
      - ?
  - F4
    - slick race tyre
    - tons of downforce
    - balanced power
    - documentation
      - ?

## Links

Wassimulator – Programming Vehicles in Games – BSC 2025: 
https://www.youtube.com/watch?v=MrIAw980iYg

HydroElastic contact with distance3d
https://github.com/AlexanderFabisch/distance3d/blob/master/examples/visualizations/vis_pressure_field.py

Pacejka Magic Formula: 
https://www.youtube.com/watch?v=We5iNzg6AAA

How race tyre works ?
https://www.youtube.com/watch?v=y5Y-w4zGW00

AARK: An Open Toolkit for Autonomous Racing Research
- https://adelaideautonomous.racing
- https://github.com/Adelaide-Autonomous-Racing-Kit
- https://arxiv.org/pdf/2410.00358

### The contact patch 

about calculating the contact patch of a tyre:
https://www.thecontactpatch.com/road/c2019-the-contact-patch

for grip: 
https://www.thecontactpatch.com/road/c1717-grip

Tyre deformation: 
https://www.thecontactpatch.com/road/c2015-a-simple-model-for-tyre-deformation

Rolling resistance:
https://www.thecontactpatch.com/general/g0119-rolling-resistance

Suspension model:
https://www.thecontactpatch.com/general/g1115-smoothing-the-ride

Ackermann Geometry:
https://www.thecontactpatch.com/road/c0504-ackermann-geometry

Cornering basic:
https://www.thecontactpatch.com/road/c0415-cornering-basics

The cornering solution:
https://www.thecontactpatch.com/road/c0418-oversteer-and-understeer

## Misc

### Gemini stuff

#### 1. The Car Body: An ODE System

This is the part that you *will* build as a set of differential equations. But they'll be **Ordinary Differential Equations (ODEs)**, not PDEs.

Why? Because the car's *overall* state (its position, velocity, rotation) only depends on **one** variable: **time ($t$)**.

You'll define a set of state variables, like:
* $x, y$ (position)
* $v_x, v_y$ (longitudinal and lateral velocity)
* $\psi$ (yaw angle, or "heading")
* $r$ (yaw rate, or how fast it's spinning)

Your ODEs will be the "clues" from physics, just like our spring example:
* $\frac{dx}{dt} = \dots$ (The change in x-position is based on the car's current velocities and angle)
* $\frac{dv_y}{dt} = \frac{F_y}{m}$ (The change in lateral velocity—i.e., acceleration—is just the **total lateral Force** divided by mass)

Your simulation's job is to solve this system of ODEs. But to do that, you need to know the **Forces** (like $F_y$). And where do those forces come from?

---

#### 2. The Tyre Model: The "Black Box" (Hides the PDEs)

This is the "magic" part. Instead of solving a PDE for the rubber, you use a pre-built formula. The most famous one is **Pacejka's Magic Formula**.

This is *not* a differential equation. It's a complex (but purely algebraic) formula that acts as a "black box":

* **You give it inputs:**
    * **Vertical Load ($F_z$):** How hard is this tyre being pushed into the ground?
    * **Slip Angle ($\alpha$):** What's the difference between the direction the tyre is *pointing* and the direction it's *actually traveling*? 
    * **Slip Ratio ($\kappa$):** How much faster/slower is the tyre spinning than it "should" be for the car's speed? (i.e., are the wheels spinning out or locking up?)

* **It gives you outputs:**
    * **Lateral Force ($F_y$):** The cornering force.
    * **Longitudinal Force ($F_x$):** The braking/acceleration force.
    * **Aligning Torque ($M_z$):** The force you feel in the steering wheel.

The formulas create the famous plots you'll be making in `matplotlib`. They show that as you increase slip angle (turn the wheel more), the cornering force builds up, *peaks*, and then *drops* as the tyre starts to slide.

#### How It All Connects (Your Simulation Loop)

So, your hobby project will look like this, step-by-step:

1.  **Start at Time $t$:** Your car has a certain position, velocity, and spin (your ODE state).
2.  **Ask the Tyres:** Based on the car's velocity and your steering input, you calculate the slip angle ($\alpha$) and slip ratio ($\kappa$) for each of the four tyres.
3.  **Run the Magic Formula:** You plug those slip angles and loads into the Pacejka formula. It instantly gives you the forces $F_x$ and $F_y$ for each tyre.
4.  **Solve the ODE:** You sum up all those tyre forces. Now you have the $F$ for your $F=ma$ equations (your ODEs). You use a numerical solver (like `scipy.integrate.solve_ivp`) to "step forward" a tiny bit in time, $dt$, to find the car's *new* state at time $t+dt$.
5.  **Plot and Repeat:** You save the state (position, velocity, forces) and go back to step 1.

### Pacejka Magic Formula

This is the core mathematical structure of the formula.

$$Y(x) = D \cdot \sin(C \cdot \arctan(B \cdot x - E \cdot (B \cdot x - \arctan(B \cdot x))))$$


#### General Formula Variables

These are the "shape" parameters of the formula itself.

  * **$Y(x)$**: The **output** you are solving for. This can be a force or a torque (e.g., $F_y$, $F_x$, or $M_z$).
  * **$x$**: The **input slip variable** you are feeding into the formula (e.g., slip angle $\alpha$ or slip ratio $\kappa$).
  * **$B$**: The **Stiffness Factor**, which controls the slope of the curve at the origin (the "cornering stiffness").
  * **$C$**: The **Shape Factor**, which controls the overall shape of the curve (e.g., whether it's a wide or narrow peak).
  * **$D$**: The **Peak Factor**, which controls the maximum value, or "peak," of the curve (the maximum grip).
  * **$E$**: The **Curvature Factor**, which controls the "sharpness" of the peak and how quickly the force drops off after the peak.

> **Important Note:** The coefficients $B$, $C$, $D$, and $E$ are **not** simple numbers. They are complex formulas that are calculated *first*, based on the tire's vertical load ($F_z$), camber angle ($\gamma$), and a long list of specific coefficients ($a_1, a_2, b_1, b_2...$) that define one particular tire model.

-----

#### Key Physical Variables (Inputs & Outputs)

These are the real-world vehicle dynamics properties that you will plug into and get out of the formula.

  * **$F_y$** (Lateral Force): The **output** ($Y$) for cornering. This is the grip that turns the car.

  * **$F_x$** (Longitudinal Force): The **output** ($Y$) for acceleration and braking.

  * **$M_z$** (Aligning Torque): The **output** ($Y$) for the twisting force on the steering.

  * **$\alpha$** (alpha): The **input** slip angle ($x$) used to calculate $F_y$ and $M_z$.

  * **$\kappa$** (kappa): The **input** slip ratio ($x$) used to calculate $F_x$.

  * **$F_z$** (Vertical Load): The most important **input** used to calculate the $B, C, D, E$ coefficients. This is the dynamic weight on the tire at any given moment.

  * **$\gamma$** (gamma): The **input** camber angle, which also affects the $B, C, D, E$ coefficients.

