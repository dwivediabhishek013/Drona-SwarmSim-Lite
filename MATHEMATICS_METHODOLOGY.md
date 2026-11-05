# Drona SwarmSim-Lite — Mathematics and Methodology

## Verified configuration

- Four drones, arena: $x\in[-1,1]$ m, $y\in[-0.6,0.6]$ m, $z\in[0.6,1.4]$ m.
- Minimum separation: $0.35$ m; controller: 25 Hz; physics: 240 Hz; camera: 30 FPS.
- Camera: $(0,0,3)$ m looking down; 1280×720; $f_x=f_y=900$ px; $(c_x,c_y)=(640,360)$ px.
- ArUco: `DICT_4X4_50`, physical marker side $L=0.08$ m; delay 80 ms; Gaussian pixel noise $\sigma=3$ px; 5% frame drop.
- Vehicle: $m=0.033$ kg, inertia $(2,2,4)\times10^{-5}$ kg·m², maximum thrust 0.72 N, maximum tilt 15°, maximum yaw rate 90°/s.
- Drift is one bounded constant acceleration vector per drone, each axis in $[-0.15,0.15]$ m/s². Battery thrust scale is linear from 1.00 to 0.93 in 120 s.

## Dynamics and underactuation

The backend uses the translational model

$$m\ddot p = T R e_3 - m g e_3 + m d,$$

where $p$ is world position, $T$ is total thrust, $R$ maps the body thrust axis into world axes, and $d$ is unknown drift acceleration. Angular dynamics are

$$J\dot\omega = \tau - \omega\times J\omega.$$

Only collective thrust and attitude are commanded. Horizontal acceleration therefore comes from tilt, making the quadrotor underactuated in world $x,y,z$.

The RC mapping is: roll/pitch 1000–2000 maps to ±15°, yaw 1000–2000 maps to ±90°/s, and throttle 1000–2000 maps to zero–maximum thrust. Hover throttle is configured as 1540, not assumed to be neutral.

## Wave kinematics

For $z_i(t)=1+A\sin(\omega t-\phi_i)$ with $A=0.2$ m and $\omega=2\pi/6=1.0472$ rad/s,

$$\dot z_i=A\omega\cos(\omega t-\phi_i),\qquad \ddot z_i=-A\omega^2\sin(\omega t-\phi_i).$$

Thus $|\dot z|_{max}=0.2094$ m/s and $|\ddot z|_{max}=0.2193$ m/s². At 93% battery the model maximum thrust is 0.6696 N, versus hover weight $mg=0.3237$ N; the nominal wave vertical acceleration remains below the available vertical acceleration margin before tilt coupling.

## Formation geometry

Line slots use

$$x_i=(i-(N-1)/2)d,\quad y_i=0,\quad z_i=1+0.2\sin(\omega t-i\pi/2).$$

Circle slots use $\theta_i=2\pi i/N$, $(x_i,y_i)=R(\cos\theta_i,\sin\theta_i)$, and phase $\phi_i=\theta_i$. The planar adjacent spacing for a circle is $2R\sin(\pi/N)$. N=10 at $R=0.45$ m yields approximately 0.278 m, below 0.35 m; the validator reports required $R\ge0.566$ m, which does not fit the fixed y-arena. N=10 line at 0.45 m spacing requires width 4.05 m, exceeding the fixed 2 m arena.

## Camera geometry

For a horizontal marker at camera depth $D=3-z$, ideal side length is

$$s_{px}=fL/D.$$

At z=0.6 m, $D=2.4$ m and $s\approx30$ px. At z=1.4 m, $D=1.6$ m and $s\approx45$ px. Approximate lateral sensitivity is $\delta x\approx D\delta u/f$; one pixel is 2.67 mm at z=0.6 and 1.78 mm at z=1.4. Depth sensitivity from $D=fL/s$ is $|\delta D|\approx fL\delta s/s^2$, so depth is more sensitive to apparent-size error, especially at the lower-altitude/smaller-marker end.

Runtime uses four-corner `solvePnP` with configured camera intrinsics and converts camera coordinates to world coordinates. A fronto-parallel IPPE NaN edge case is explicitly rejected/falls back to iterative PnP before filtering.

## Control

The cascaded control path is

$$e_p=p^*-\hat p\rightarrow v^*\rightarrow e_v=v^*-\hat v\rightarrow a^*\rightarrow (RC_{roll},RC_{pitch},RC_{throttle}),$$

with wrapped yaw error mapped to RC yaw. Position gains are $(1.2,0.02,0.05)$ and velocity gains $(1.2,0.03,0.03)$ for $(K_p,K_i,K_d)$. Conditional integration and integral/output limits provide anti-windup. Around hover, small roll/pitch satisfy $a_x\approx g\theta$ and $a_y\approx-g\phi$; this is the basis of the acceleration-to-tilt map.

## Latency-compensated estimator methodology

A future Kalman implementation can use $x=[p,v]^T$ and

$$F=\begin{bmatrix}I&\Delta t I\\0&I\end{bmatrix},\quad H=[I\;0],$$

with $Q$ from acceleration uncertainty and $R$ from image/pose uncertainty. Delayed measurements should update at capture time then predict forward through the 80 ms camera and 40 ms command delays. The current implementation uses an alpha-beta filter with capture-time increments; rejected candidates do not mutate state or accepted timestamps.

## Additional methodology

- A yaw drift of 0.5°/s rotates body-frame horizontal commands; world-to-body correction uses the estimated yaw before RC roll/pitch mapping.
- A future downwash model can mark a cone beneath each vehicle and incorporate vertical separation constraints in preflight validation.
- F2 (1.2 s hide of ID 3) and F4 (1 s command denial of drone 0) are methodology-only extensions; the required F1/F3 drills are implemented separately.
- Scaling has assignment cost $O(N^3)$ and pairwise safety cost $O(N^2)$; camera marker crowding and fixed arena geometry make N=10 infeasible under the supplied values.

## Evidence policy

Measured tracking, localization, phase, and shape metrics are reported only from completed stable runs in their run artifacts. This document does not claim metric targets without a corresponding verified run file.
