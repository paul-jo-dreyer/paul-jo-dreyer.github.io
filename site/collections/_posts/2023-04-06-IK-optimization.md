---
title:  Optimal Kinematic Posing
date:   2025-10-16 15:01:35 +0300
image:  '/images/planar_arm/cover_no_words.png'
description: Viewing inverse kinematics as a constrained optimization problem
tags:   [Kinematics, Optimization]
---

***Forward Kinematics (FK)*** - Given a set of joint angles, determine the pose of the end effector.

***Inverse Kinematics (IK)*** - Given a pose of the end effector, determine a set of valid joint angles.

# Introduction

Suppose we want to solve the inverse kinematics (IK) for a robot in order to do something useful, like preparing to grasp an object. In many kinematic problems, it’s possible to derive an explicit solution to the IK equations, yielding one or more valid joint configurations. However, many tutorials that cover these analytical methods overlook real-world constraints such as joint limits, torque limits, etc.

Instead, we can treat IK as an optimization problem. This approach lets us shape the solution space to reflect the robot’s physical limits and operating goals. The field of Trajectory Optimization (TrajOpt) applies similar principles to plan motion trajectories that minimize time, energy, or other costs while respecting constraints like collision avoidance. While TrajOpt typically starts from a fixed initial configuration, we can borrow the same ideas to find optimal static poses for a fixed-base robot arm.

As an example, let’s use the <a href="https://www.google.com/url?sa=t&source=web&rct=j&opi=89978449&url=https://www.universal-robots.com/products/ur10e/&ved=2ahUKEwjZzJ_i76KQAxU-mIkEHeDQAfIQFnoECBcQAQ&usg=AOvVaw1S9W4mTZJFHXiCrMXvr34G" target="_blank" rel="noopener"> <strong>UR10</strong></a> robot arm from Universal Robots, which has six joints. Here, we’ll define optimal as the configuration that requires the least torque to hold the end effector at a given position.

The gif below shows two different joint configurations that achieve the same end effector position. Both are valid IK solutions, but one requires significantly more torque to maintain. That’s the key idea behind pose optimization: not all valid IK solutions are equally efficient or stable.

![Elbow Up Elbow Down](/images/planar_arm/ik.gif){: width="1200" height="900"}

--------

# Formulating The Problem

## Unconstrained IK: Baseline

Let's start by walking through how we formulate the IK problem in it's simplest form: a least-squares regression on the error between the current end effector position and it's desired position. We will build on top of this foundation next.

We can define the following:

$$
\begin{equation}
k_f(\theta) = \prod_{i=1}^{6} g_i(\theta)\\
\end{equation}
$$

where:

$$
\begin{align*}
k_f(\theta) &\in \mathbb{R}^3 &&\text{(} \textit{Forward kinematics} \text{ mapping)} \\
g_i &\in \mathbb{R}^{4 \times 4} &&\text{(Transformation matrix between joint frames)} \\
\theta &\in \mathbb{R}^6 &&\text{(Joint configuration vector)} \\[8pt]
\end{align*}
$$

Then we can formulate the IK solution as the following unconstrained minimization problem:

$$
\begin{equation}
\min_{\theta} || k_f(\theta) - x_{goal} ||_2  \\[8pt]
\end{equation}
$$

Let's solve this unconstrained problem, using mujoco to handle the derivatives. Example below, full code <a href="https://github.com/paul-jo-dreyer/mj_inverse_kinematics" target="_blank" rel="noopener"><strong>(Here)</strong></a>.:

<details class="code-block" style="margin:1em 0; padding:1em; border:1px solid #444; border-radius:8px; background-color:#222;">
  <summary>💻 View Python</summary>
  <div class="highlight">
    {% highlight python %}
def solve_unconstrained_ik(
    model: mujoco.MjModel,
    data: mujoco.MjData,
    target_pos: np.ndarray,
    end_effector_name = "wrist_3_link",
    max_steps=100,
    tol=1e-5,
    damping=1e-4,
    verbose=True,
):
    """
    Iterative inverse kinematics solver for a given site target.
    Uses damped least squares on MuJoCo's analytic Jacobian.
    """
    nq = model.nq

    ee_id = model.body(end_effector_name).id # End effector

    convergence = False
    for step in range(max_steps):
 
        # Forward kinematics
        mujoco.mj_forward(model, data)

        curr_pos = data.body(ee_id).xpos.copy()
        
        # position error
        pos_err = target_pos - curr_pos

        # Check convergence
        if np.linalg.norm(pos_err) < tol:
            if verbose:
                print(f"IK converged in {step} iterations.")
                convergence = True
            break

        # Compute Jacobian (translation and rotation)
        J_pos = np.zeros((3, model.nv))
        J_rot = np.zeros((3, model.nv))
        mujoco.mj_jacBody(model, data, J_pos, J_rot, ee_id)

        rhs = J_pos @ J_pos.T + damping * np.eye(J_pos.shape[0])

        # Damped least-squares inverse
        dq = J_pos.T @ np.linalg.solve(rhs, pos_err)

        # update
        data.qpos[:] += dq

    # Final forward pass
    mujoco.mj_forward(model, data)

    if not convergence:
        print(f"IK failed to converge.")

    return data.qpos.copy()
    {% endhighlight %}
  </div>
</details>

--------

## Constrained IK: Torque-Optimal

The unconstrained version of inverse kinematics is straightforward to implement and computationally lightweight. However, to make it more realistic, we need to introduce constraints that capture physical limits, and we can reshape our problem to do something useful like minimizing holding torque.

The <a href="https://underactuated.mit.edu/multibody.html" target="_blank" rel="noopener">
<strong>manipulator dynamics</strong></a> are defined by:

$$
\begin{equation}
M(\theta)\,\ddot{\theta} + C(\theta,\dot{\theta})\,\dot{\theta} + G(\theta) = \tau + J^\top(\theta)\,F_{\text{ext}}
\end{equation}
$$

where:

$$
\begin{align*}
M(\theta) &\in \mathbb{R}^{6 \times 6} &&\text{(Mass matrix)} \\
C(\theta, \dot{\theta}) &\in \mathbb{R}^{6 \times 6} &&\text{(Coriolis matrix)} \\
G(\theta) &\in \mathbb{R}^{6} &&\text{(Gravity vector)} \\
\tau &\in \mathbb{R}^6 &&\text{(Joint torques)} \\
J(\theta) &\in \mathbb{R}^{3 \times 6} &&\text{(Kinematic jacobian)} \\
F_{\text{ext}} &\in \mathbb{R}^{3} &&\text{(External Forces at the tip)} \\
\end{align*}
$$

At static equilibrium, (holding a pose with zero velocity and acceleration):

$$
\begin{equation}
\quad \dot{\theta} = 0, \quad \ddot{\theta} = 0
\end{equation}
$$

Thus we can cancel terms, and are left with:

$$
\begin{equation}
\Rightarrow \quad G(\theta) = \tau + J^\top(\theta)\,F_{\text{ext}}
\end{equation}
$$

$$
\begin{equation}
\boxed{\tau = G(\theta) - J^\top(\theta)\,F_{\text{ext}}}
\end{equation}
$$

$$
\begin{equation}
\text{If } F_{\text{ext}} = 0, \quad \boxed{\tau = G(\theta)}
\end{equation}
$$

Now, instead of minimizing only the IK error, we use that error as an equality constraint and minimize the joint torques required to hold the pose:

$$
\begin{equation}
\min_{\theta} j(\theta) = \frac{1}{2} G(\theta)^TWG(\theta)\\[4pt]
\end{equation}
$$

$$
\begin{equation}
c(\theta) = k_f(\theta) - x_{goal}
\end{equation}
$$

$$
\begin{equation}
h(\theta) \;=\; \text{log}\big(q_{\text{target}}\otimes q(\theta)^{-1}\big)
\end{equation}
$$

$$
\begin{align*}
\text{s.t.}\quad
& c(\theta) = 0 && \text{(IK Position)}\\
& h(\theta) = 0 && \text{(IK Orientation)}\\
& \theta_{\min} \le \theta \le \theta_{\max} &&\text{(Joint limits)}\\
& \tau_{\min} \le \tau \le \tau_{\max} &&\text{(Torque limits)}\\
\end{align*}
$$


Where $$W$$ is an optional square matrix used to weight the independent joint torques if we wish to, or set to identity if we don’t want to bias any joint. Additionally, we have inequality constraints specifying that our torque limits and joint limits must be respected for the solution to be valid. For more details on how the manipulator dynamics are derived, see <a href="https://underactuated.mit.edu/multibody.html" target="_blank" rel="noopener">
<strong>(Here)</strong></a>.

It's important to note that we are now solving a non-convex, constrained optimization problem. Writing a solver from scratch to handle these constraints is a bit more involved, but essentially we would write out the <a href="https://en.wikipedia.org/wiki/Karush%E2%80%93Kuhn%E2%80%93Tucker_conditions" target="_blank" rel="noopener"><strong>(KKT Conditions)</strong></a> for the problem, and introduce a change of variables to handle the inequality constraints. For anyone interested in learning about constrained optimization meathods such as the Augmented Lagrangian method or Interior Point methods, you should 1000% check out "16-745: Optimal Control and RL", a class by professor Zac Manchester at CMU. This class is absolutely kick ass, and the whole series is on youtube <a href="https://www.youtube.com/watch?v=-f1pu8vsnYw&list=PLZnJoM76RM6Jv4f7E7RnzW4rijTUTPI4u&index=5" target="_blank" rel="noopener">
<strong>(Here)</strong></a>. 

That being said, we're just gonna rip a Scipy solver here and use MuJoCo for the derivatives. Take a look at the partial code below to see how we setup the constraints and interface with the api, again the full code is in this repo <a href="https://github.com/paul-jo-dreyer/mj_inverse_kinematics" target="_blank" rel="noopener">
<strong>(Here)</strong></a>. 

Additional note: This is a non-convex problem = no globally optimal solution guarantees. This particular problem is low dimensional enough that you can just probably just perform a coarse sweep over the work space and land on a globally optimal solution. 

<details class="code-block" style="margin:1em 0; padding:1em; border:1px solid #444; border-radius:8px; background-color:#222;">
  <summary>💻 View Python</summary>
  <div class="highlight">
    {% highlight python %}

import mujoco
import mujoco.viewer
import numpy as np
from scipy.optimize import minimize, Bounds, NonlinearConstraint
from scipy.spatial.transform import Rotation as R

# ==============================
# MuJoCo Dynamics Helpers
# ==============================
def static_torque(q, model, data):
    """Compute the static torque required to hold a pose."""
    mujoco.mj_resetData(model, data)
    data.qpos[:] = q
    data.qvel[:] = 0
    mujoco.mj_forward(model, data)
    return data.qfrc_bias.copy()

# ==============================
# Optimization Objective
# ==============================
def torque_objective(q, model, data, R):
    """Minimize joint torques at static equilibrium."""
    tau = static_torque(q, model, data)
    return 0.5 * tau @ R @ tau

def torque_objective_grad(q, model, data, R, eps=1e-6):
    """Finite-difference gradient of torque objective."""
    n = q.size
    tau = static_torque(q, model, data)
    grad = np.zeros(n)
    for i in range(n):
        dq = np.zeros_like(q)
        dq[i] = eps
        tau_p = static_torque(q + dq, model, data)
        tau_m = static_torque(q - dq, model, data)
        dtau_dqi = (tau_p - tau_m) / (2 * eps)
        grad[i] = dtau_dqi @ (R @ tau)
    return grad

def ik_constraint(q, model, data, ee_name, target_pos, target_quat=None):
    """Compute end-effector (body or site) position and orientation error.

    Args:
        q (np.ndarray): Joint configuration vector.
        model (mujoco.MjModel): MuJoCo model.
        data (mujoco.MjData): MuJoCo data.
        ee_name (str): Name of the end-effector body or site.
        target_pos (np.ndarray): Desired world position [x, y, z].
        target_quat (np.ndarray, optional): Desired world orientation [w, x, y, z].
    
    Returns:
        np.ndarray: Concatenated position and orientation error (6D).
    """
    SOLVER_HIST.append(q)

    mujoco.mj_resetData(model, data)
    data.qpos[:] = q
    mujoco.mj_forward(model, data)

    # Try to interpret ee_name as a site first, fallback to body
    try:
        ee_id = mujoco.mj_name2id(model, mujoco.mjtObj.mjOBJ_SITE, ee_name)
        ee_pos = np.copy(data.site_xpos[ee_id])
        # ee_mat = data.site_xmat[ee_id].reshape(3, 3)
        # ee_quat = R.from_matrix(ee_mat).as_quat()  # [x, y, z, w] order
        # ee_quat = np.roll(ee_quat, 1)  # convert to [w, x, y, z]

    except ValueError:
        ee_id = mujoco.mj_name2id(model, mujoco.mjtObj.mjOBJ_BODY, ee_name)
        ee_pos = np.copy(data.xpos[ee_id])
        ee_quat = np.copy(data.xquat[ee_id])  # already [w, x, y, z]

    # --- Position error ---
    pos_err = ee_pos - target_pos

    # --- Orientation error ---
    if target_quat is not None:
        q_err = quat_mul(quat_conjugate(target_quat), ee_quat)
        ori_err = quat_to_axis_angle(q_err)
    else:
        ori_err = np.zeros(3)

    return np.concatenate([pos_err, ori_err])

def ik_constraint_jac(q, model, data, ee_name):
    """6×N Jacobian for end-effector position + orientation.

    Automatically handles both body and site names.
    """
    mujoco.mj_resetData(model, data)
    data.qpos[:] = q
    mujoco.mj_forward(model, data)

    # Allocate
    Jp = np.zeros((3, model.nv))
    Jr = np.zeros((3, model.nv))

    # Try as site first
    try:
        ee_id = mujoco.mj_name2id(model, mujoco.mjtObj.mjOBJ_SITE, ee_name)
        mujoco.mj_jacSite(model, data, Jp, Jr, ee_id)
    except ValueError:
        ee_id = mujoco.mj_name2id(model, mujoco.mjtObj.mjOBJ_BODY, ee_name)
        mujoco.mj_jacBody(model, data, Jp, Jr, ee_id)

    # Stack [linear; angular] → shape (6, nq)
    return np.vstack([Jp, Jr])[:, :model.nq]

# ==============================
# Main Solver
# ==============================
def solve_constrained_ik(model, data, ee_name, target_pos, target_quat=None,
                         q0=None, R=None, tol=1e-3):
    """Solve constrained IK minimizing static torque subject to IK equality."""
    if q0 is None:
        q0 = data.qpos.copy()
    if R is None:
        R = np.eye(model.nv)

    # flatten functions to single arg (scipy api)
    def obj(q): return torque_objective(q, model, data, R)
    def obj_grad(q): return torque_objective_grad(q, model, data, R)
    def con_fun(q): return ik_constraint(q, model, data, ee_name, target_pos, target_quat)
    def con_jac(q): return ik_constraint_jac(q, model, data, ee_name)

    # IK solution constraint
    constraint = NonlinearConstraint(con_fun, -tol*np.ones(6), tol*np.ones(6), jac=con_jac)

    # joint limits constraint
    bounds = Bounds(model.jnt_range[:, 0], model.jnt_range[:, 1])

    # Pose solution
    res = minimize(obj, q0, method='trust-constr', jac=obj_grad,
                   constraints=[constraint], bounds=bounds,
                   options={'verbose': 3, 'maxiter': 300, 'gtol': 1e-6, 'xtol': 1e-6})

    return res
    {% endhighlight %}
  </div>
</details>


Let's see the benefits in action. Suppose our target end effector position is:

$$
\begin{equation}
k_f(\theta) = [x, y, z]^T = [0.5, 0.0, 0.5]^T
\end{equation}
$$

Both solvers converge to a valid solution, however the sum of torques from the unconstrained solution result in **147 Nm**, while our constrained solver finds a pose in which the sum of torques is just **34 Nm**. 

--------

# Conclusion

The take away here is that we can bootstrap our original intent to solve the inverse kinematics problem with additional information about what we want the solution to look like by formulating it as a constrained optimization. 


Now it's important to point out that this is a non-convex problem, which means that your initial guess at the joint angles $$\theta$$ can entirely determine if the solver will converge to a feasible solution. Given that we will typically run something like this in an offline setting, and that robot manipulators often contain a fairly low dimensional decision vector we can probably get decent solution by trying a few uniformly distributed initial guesses over the joint space, but no globally optimal guarantees here.

Please note that this does not include a constraint on robot self-collisions, meaning a solution may respect joint limits but still cause a self intersection.

Addtionally, MuJoCo is pretty awesome. It runs much faster than Bullet Physics, and it comes with a stacked toolbox to help you handle dynamics and derivatives. Trust me, if there is one thing that will slow your development speed, it's bugs in the dynamics or derivatives.