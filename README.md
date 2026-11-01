# pid-maglev

Floating a neodymium magnet under an electromagnet. A Hall effect sensor reads how far away the magnet is and a PID loop adjusts the coil current to hold it in place.

The system is unstable on its own: if the magnet drifts closer, the pull gets stronger and it snaps up to the coil. If it drifts away, it falls. So the whole project comes down to the controller.

## The math

I derived the model by hand, then checked it in MATLAB. Short version:

Force balance on the magnet, with magnetic force falling off as 1/x²:

$$m\ddot{x} = mg - \frac{k i^2}{x^2}$$

Linearizing around the hover point $(x_0, i_0)$ gives

$$\delta\ddot{x} = a \delta x - b \delta i, \qquad a = \frac{2k i_0^2}{m x_0^3}, \quad b = \frac{2k i_0}{m x_0^2}$$

and the open-loop transfer function

$$\frac{X(s)}{I(s)} = \frac{-b}{s^2 - a}$$

That pole at $+\sqrt{a}$ is why it can't hover without feedback. Closing the loop with PD control and applying Routh-Hurwitz gives two conditions for stability:

$$K_d > 0, \qquad K_p > \frac{a}{b} = \frac{i_0}{x_0}$$

The full handwritten derivation is in `assets/`:

![Derivation page 1](assets/page%201.png)
![Derivation page 2](assets/page%202.png)

## What's in here

| File | What it does |
|---|---|
| `matlab/math_verification.mlx` | Checks the hand derivation symbolically |
| `matlab/root_locus.mlx` | Root locus of the linearized plant, used to pick gains |
| `matlab/simulink_maglev.slx` | Simulink model of the closed loop |

## Hardware

An Arduino runs the control loop, reading the Hall sensor and driving the electromagnet.
