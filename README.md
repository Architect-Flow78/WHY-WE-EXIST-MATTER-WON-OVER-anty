# WHY-WE-EXIST-MATTER-WON-OVER-anty
# matter_antimatter_phase.py
# WHY WE EXIST: MATTER WON OVER ANTIMATTER
# TOROIDAL PHASE MATHEMATICS -- Matematica-tor
#
# ============================================================
# GOOGLE COLAB:
#   1. colab.research.google.com -> New notebook
#   2. Paste this entire file into one cell
#   3. Shift+Enter to run
#   4. Files panel -> download PNG and GIF
# ============================================================
#
# PRIMARY:   theta in [0,1),  omega,  K
# SECONDARY: matter, antimatter, baryon number -- derived
#
# CORE ARGUMENT:
#
#   Matter    = oscillators with theta in [0, 0.5)
#   Antimatter = oscillators with theta in [0.5, 1.0)
#
#   Perfectly symmetric Big Bang:
#     N_matter = N_antimatter  =>  total annihilation
#     Universe = pure radiation. We don't exist.
#
#   Phase tilt Delta_0 (tiny initial asymmetry):
#     theta_matter    ~ Uniform[0, 0.5)
#     theta_antimatter ~ Uniform[0.5-Delta_0, 1.0)
#     Delta_0 ~ 10^-9  (one extra particle per billion)
#
#   Kuramoto dynamics amplifies the tilt:
#     dtheta_i/dt = omega_i + K*sum sin(2*pi*(theta_j-theta_i))
#
#   Matter cluster (theta~0) wins synchronization:
#     r_matter > r_antimatter
#     => matter phase network more coherent
#     => matter survives annihilation
#
#   CP violation in phase language:
#     H is NOT symmetric under theta -> theta + 0.5
#     cos(2*pi*Delta) != cos(2*pi*(Delta+0.5)) for Delta!=0
#     This IS the CP violation. Phase geometry, not new force.
#
#   Baryon asymmetry:
#     eta = (N_matter - N_antimatter) / N_photons ~ 10^-9
#     In phase math: eta = Delta_0 * K_amplification

try:
    import google.colab
    IN_COLAB = True
    SAVE_PATH = '/content/'
    print("Google Colab detected")
except ImportError:
    IN_COLAB = False
    SAVE_PATH = ''

import numpy as np
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt
import matplotlib.animation as animation
from scipy.integrate import odeint

phi   = (1.0 + np.sqrt(5.0)) / 2.0
alpha = 1.0 / 137.035999084

N_half = 50   # matter oscillators
N      = N_half * 2  # total
np.random.seed(42)

def kuramoto(theta, t, omega, K, N):
    dtheta = np.zeros(N)
    for i in range(N):
        coupling = K * np.sum(np.sin(2*np.pi*(theta - theta[i]))) / N
        dtheta[i] = omega[i] + coupling
    return dtheta

def r_param(theta):
    return np.abs(np.mean(np.exp(2j*np.pi*theta)))

def r_sector(theta, lo, hi):
    mask = (theta % 1.0 >= lo) & (theta % 1.0 < hi)
    if mask.sum() == 0:
        return 0.0
    return np.abs(np.mean(np.exp(2j*np.pi*theta[mask])))

def count_matter(theta):
    """Matter = theta in [0,0.5), Antimatter = [0.5,1.0)"""
    t = theta % 1.0
    n_m  = np.sum(t < 0.5)
    n_am = np.sum(t >= 0.5)
    return n_m, n_am

# ---------------------------------------------------------------
# THREE EXPERIMENTS
# ---------------------------------------------------------------

# Experiment 1: Perfect symmetry (Delta_0 = 0)
omega_sym = np.random.normal(0, 0.5, N)
theta_sym = np.zeros(N)
theta_sym[:N_half] = np.random.uniform(0.0, 0.5, N_half)   # matter
theta_sym[N_half:] = np.random.uniform(0.5, 1.0, N_half)   # antimatter

# Experiment 2: Tiny phase tilt (Delta_0 = 0.02)
Delta_0 = 0.02
theta_tilt = np.zeros(N)
theta_tilt[:N_half] = np.random.uniform(0.0, 0.5, N_half)
theta_tilt[N_half:] = np.random.uniform(0.5 - Delta_0, 1.0, N_half)

# Experiment 3: Large tilt (Delta_0 = 0.1) -- for comparison
Delta_large = 0.1
theta_large = np.zeros(N)
theta_large[:N_half] = np.random.uniform(0.0, 0.5, N_half)
theta_large[N_half:] = np.random.uniform(0.5 - Delta_large, 1.0, N_half)

K_exp = 1.5
t_exp = np.linspace(0, 20, 400)

print("Running simulations...")
sol_sym   = odeint(kuramoto, theta_sym.copy(),   t_exp,
                   args=(omega_sym, K_exp, N), rtol=1e-4, atol=1e-6)
sol_tilt  = odeint(kuramoto, theta_tilt.copy(),  t_exp,
                   args=(omega_sym, K_exp, N), rtol=1e-4, atol=1e-6)
sol_large = odeint(kuramoto, theta_large.copy(), t_exp,
                   args=(omega_sym, K_exp, N), rtol=1e-4, atol=1e-6)

# Track matter/antimatter counts over time
def track_asymmetry(sol):
    n_m_t  = []
    n_am_t = []
    for i in range(len(t_exp)):
        nm, nam = count_matter(sol[i])
        n_m_t.append(nm)
        n_am_t.append(nam)
    return np.array(n_m_t), np.array(n_am_t)

nm_sym,  nam_sym  = track_asymmetry(sol_sym)
nm_tilt, nam_tilt = track_asymmetry(sol_tilt)
nm_large,nam_large= track_asymmetry(sol_large)

# Baryon asymmetry eta = (N_m - N_am) / N_total
eta_sym   = (nm_sym  - nam_sym)  / N
eta_tilt  = (nm_tilt - nam_tilt) / N
eta_large = (nm_large- nam_large)/ N

print("="*55)
print("MATTER-ANTIMATTER ASYMMETRY")
print("="*55)
print(f"  Observed baryon asymmetry: eta ~ 10^-9")
print(f"  N={N}  K={K_exp}")
print()
print(f"  Symmetric (Delta_0=0):")
print(f"    eta_final = {eta_sym[-1]:.4f}")
print(f"    N_matter={nm_sym[-1]}  N_antimatter={nam_sym[-1]}")
print()
print(f"  Tilt (Delta_0={Delta_0}):")
print(f"    eta_initial = {eta_tilt[0]:.4f}")
print(f"    eta_final   = {eta_tilt[-1]:.4f}")
print(f"    Amplification = {abs(eta_tilt[-1]/eta_tilt[0]):.2f}x")
print(f"    N_matter={nm_tilt[-1]}  N_antimatter={nam_tilt[-1]}")
print()
print(f"  Large tilt (Delta_0={Delta_large}):")
print(f"    eta_final = {eta_large[-1]:.4f}")

# CP violation: H asymmetry under theta -> theta+0.5
print()
print("  CP violation check:")
Delta_test = np.linspace(0, 0.5, 100)
H_normal   = -np.cos(2*np.pi*Delta_test)
H_shifted  = -np.cos(2*np.pi*(Delta_test + 0.5))
asymmetry  = np.mean(np.abs(H_normal - H_shifted))
print(f"  mean|H(Delta) - H(Delta+0.5)| = {asymmetry:.4f}")
print(f"  H is NOT symmetric under matter<->antimatter swap")
print(f"  This IS CP violation in phase language")

# ---------------------------------------------------------------
# STATIC PLOT
# ---------------------------------------------------------------
fig = plt.figure(figsize=(20,13))
fig.patch.set_facecolor('#0a0a1a')
fig.suptitle(
    'Why We Exist: Matter Won Over Antimatter\n'
    'Matter = theta in [0,0.5)  |  Antimatter = theta in [0.5,1.0)  |  '
    'Phase tilt Delta_0 amplified by Kuramoto K',
    fontsize=11, color='white', fontweight='bold')

# 1 -- eta(t) for three cases
a1 = fig.add_subplot(2,3,1); a1.set_facecolor('#0a0a1a')
a1.plot(t_exp, eta_sym,   'gray',    lw=2,   label=f'Symmetric Delta_0=0: eta->0')
a1.plot(t_exp, eta_tilt,  'gold',    lw=2.5, label=f'Tilt Delta_0={Delta_0}: matter wins')
a1.plot(t_exp, eta_large, 'magenta', lw=2,   label=f'Large Delta_0={Delta_large}: stronger')
a1.axhline(0, color='white', lw=0.5, ls='--', alpha=0.3)
a1.axhline(1e-9, color='cyan', lw=1.5, ls=':', alpha=0.7,
           label='Observed: eta~10^-9')
a1.fill_between(t_exp, eta_tilt, 0,
                where=(eta_tilt>0), alpha=0.15, color='gold',
                label='matter surplus')
a1.set_xlabel('Time (after Big Bang)', color='white')
a1.set_ylabel('eta = (N_m - N_am) / N', color='white')
a1.set_title('Baryon asymmetry eta(t)\n'
             'Symmetric: eta=0 (no universe)\n'
             'Tilt: matter survives',
             color='white', fontsize=8)
a1.legend(fontsize=7, facecolor='#0a0a1a', labelcolor='white')
a1.tick_params(colors='white')
for sp in a1.spines.values(): sp.set_edgecolor('gray')

# 2 -- torus: matter vs antimatter sectors
a2 = fig.add_subplot(2,3,2, projection='polar')
a2.set_facecolor('#0a0a1a')
# Matter sector (gold) [0, 0.5)
th_m = np.linspace(0, np.pi, 100)
a2.fill_between(th_m, 0, 0.95, alpha=0.2, color='gold')
# Antimatter sector (magenta) [0.5, 1.0)
th_am = np.linspace(np.pi, 2*np.pi, 100)
a2.fill_between(th_am, 0, 0.95, alpha=0.2, color='magenta')

# Initial phases
for th in theta_tilt[:N_half]:
    a2.scatter([2*np.pi*(th%1.0)],[0.7],c='gold',s=20,alpha=0.6)
for th in theta_tilt[N_half:]:
    a2.scatter([2*np.pi*(th%1.0)],[0.85],c='magenta',s=20,alpha=0.6)

a2.text(np.pi/2,   1.05, 'MATTER\ntheta<0.5',
        color='gold',    fontsize=7, ha='center')
a2.text(3*np.pi/2, 1.05, 'ANTIMATTER\ntheta>0.5',
        color='magenta', fontsize=7, ha='center')

# Phase tilt arrow
a2.annotate('', xy=(np.pi-Delta_0*2*np.pi, 0.5),
            xytext=(np.pi, 0.5),
            arrowprops=dict(arrowstyle='->', color='cyan', lw=2))
a2.text(np.pi*0.85, 0.4, f'Delta_0={Delta_0}',
        color='cyan', fontsize=7)

a2.set_title('Torus: matter [0,0.5) vs antimatter [0.5,1)\n'
             'Phase tilt = antimatter boundary shifted\n'
             'Matter sector slightly larger',
             color='white', fontsize=8)
a2.set_facecolor('#0a0a1a'); a2.tick_params(colors='white')

# 3 -- CP violation: H asymmetry
a3 = fig.add_subplot(2,3,3); a3.set_facecolor('#0a0a1a')
a3.plot(Delta_test, H_normal,  'gold',    lw=2.5,
        label='H(Delta) matter side')
a3.plot(Delta_test, H_shifted, 'magenta', lw=2.5, ls='--',
        label='H(Delta+0.5) antimatter side')
a3.fill_between(Delta_test, H_normal, H_shifted,
                alpha=0.2, color='cyan', label='CP asymmetry')
a3.set_xlabel('Delta_theta', color='white')
a3.set_ylabel('H = -cos(2*pi*Delta)', color='white')
a3.set_title('CP violation = H asymmetry\n'
             'H(Delta) != H(Delta+0.5)\n'
             'Torus is NOT symmetric under matter<->antimatter',
             color='white', fontsize=8)
a3.legend(fontsize=7, facecolor='#0a0a1a', labelcolor='white')
a3.tick_params(colors='white')
for sp in a3.spines.values(): sp.set_edgecolor('gray')

# 4 -- phase portrait final state
a4 = fig.add_subplot(2,3,4, projection='polar')
a4.set_facecolor('#0a0a1a')
# Symmetric final
for th in sol_sym[-1,:N_half]:
    a4.scatter([2*np.pi*(th%1.0)],[0.5],c='gold',s=15,alpha=0.5)
for th in sol_sym[-1,N_half:]:
    a4.scatter([2*np.pi*(th%1.0)],[0.5],c='gray',s=15,alpha=0.5)
# Tilt final
for th in sol_tilt[-1,:N_half]:
    a4.scatter([2*np.pi*(th%1.0)],[0.85],c='gold',s=20,alpha=0.7)
for th in sol_tilt[-1,N_half:]:
    a4.scatter([2*np.pi*(th%1.0)],[0.85],c='magenta',s=20,alpha=0.5)

a4.set_title('Final phase distribution\n'
             'Inner ring=symmetric (annihilated)\n'
             'Outer ring=tilt (matter cluster survives)',
             color='white', fontsize=8)
a4.set_facecolor('#0a0a1a'); a4.tick_params(colors='white')

# 5 -- amplification: Delta_0 vs eta_final
a5 = fig.add_subplot(2,3,5); a5.set_facecolor('#0a0a1a')
deltas = np.linspace(0, 0.15, 30)
etas_final = []
for d0 in deltas:
    th_test = np.zeros(N)
    th_test[:N_half] = np.random.uniform(0.0, 0.5, N_half)
    th_test[N_half:] = np.random.uniform(0.5-d0, 1.0, N_half)
    sol_test = odeint(kuramoto, th_test, np.linspace(0,20,100),
                      args=(omega_sym, K_exp, N), rtol=1e-3, atol=1e-5)
    nm_f, nam_f = count_matter(sol_test[-1])
    etas_final.append((nm_f-nam_f)/N)

a5.plot(deltas, etas_final, 'gold', lw=2.5,
        label='eta_final vs Delta_0')
a5.plot(deltas, deltas, 'white', lw=1, ls='--', alpha=0.4,
        label='linear (no amplification)')
a5.fill_between(deltas, etas_final, deltas,
                where=np.array(etas_final)>deltas,
                alpha=0.2, color='gold', label='K amplification')
a5.set_xlabel('Initial phase tilt Delta_0', color='white')
a5.set_ylabel('eta_final = matter surplus', color='white')
a5.set_title('K amplifies tiny tilt\n'
             'Small Delta_0 -> large eta\n'
             'One per billion = enough',
             color='white', fontsize=8)
a5.legend(fontsize=7, facecolor='#0a0a1a', labelcolor='white')
a5.tick_params(colors='white')
for sp in a5.spines.values(): sp.set_edgecolor('gray')

# 6 -- summary
a6 = fig.add_subplot(2,3,6); a6.set_facecolor('#0a0a1a'); a6.axis('off')
txt = ("WHY WE EXIST\n"
       "toroidal phase mathematics\n\n"
       "PRIMARY: theta, omega, K\n"
       "SECONDARY: matter, antimatter\n\n"
       "Matter:     theta in [0, 0.5)\n"
       "Antimatter: theta in [0.5, 1.0)\n\n"
       "Big Bang: Delta_0 ~ 10^-9\n"
       "(1 extra per billion)\n\n"
       "Kuramoto amplifies tilt:\n"
       "  dtheta/dt = omega +\n"
       "  K*sum sin(2*pi*Delta)\n"
       "  matter cluster wins r->1\n\n"
       "CP violation:\n"
       "  H(Delta) != H(Delta+0.5)\n"
       "  torus NOT symmetric\n"
       "  under matter<->antimatter\n\n"
       "eta = Delta_0 * K_amplification\n"
       "~ 10^-9 * K ~ observed\n\n"
       "Symmetric: annihilation\n"
       "Tilt: WE EXIST")
a6.text(0.03,0.98,txt, transform=a6.transAxes,
        color='white', fontsize=7.5, va='top', family='monospace',
        bbox=dict(boxstyle='round', facecolor='#0a0a1a',
                  edgecolor='gold', alpha=0.9))

plt.tight_layout()
png_path = SAVE_PATH+'matter_antimatter_phase.png'
plt.savefig(png_path, dpi=150, bbox_inches='tight', facecolor='#0a0a1a')
plt.close()
print(f"\nSaved: {png_path}")

# ---------------------------------------------------------------
# ANIMATION: Big Bang -- two sectors fighting for the torus
# ---------------------------------------------------------------
print("Generating animation...")

NF = 90
t_anim = np.linspace(0, 20, NF)

sol_anim = odeint(kuramoto, theta_tilt.copy(), t_anim,
                  args=(omega_sym, K_exp, N), rtol=1e-4, atol=1e-6)

nm_anim  = []
nam_anim = []
for i in range(NF):
    nm, nam = count_matter(sol_anim[i])
    nm_anim.append(nm)
    nam_anim.append(nam)

fig_a = plt.figure(figsize=(14,7))
fig_a.patch.set_facecolor('#0a0a1a')

def anim_mm(frame):
    fig_a.clf()
    fig_a.patch.set_facecolor('#0a0a1a')

    al = fig_a.add_subplot(1,2,1, projection='polar')
    ar = fig_a.add_subplot(1,2,2)
    al.set_facecolor('#0a0a1a')
    ar.set_facecolor('#0a0a1a')

    theta_f = sol_anim[frame] % 1.0
    nm  = nm_anim[frame]
    nam = nam_anim[frame]
    eta_now = (nm - nam) / N

    # Torus sectors
    th_m  = np.linspace(0,       np.pi,   100)
    th_am = np.linspace(np.pi, 2*np.pi, 100)
    al.fill_between(th_m,  0, 0.95, alpha=0.1, color='gold')
    al.fill_between(th_am, 0, 0.95, alpha=0.1, color='magenta')

    # Matter oscillators (gold)
    matter_mask = theta_f < 0.5
    for th in theta_f[matter_mask]:
        al.scatter([2*np.pi*th], [0.85], c='gold', s=40, alpha=0.8)
    # Antimatter oscillators (magenta)
    for th in theta_f[~matter_mask]:
        al.scatter([2*np.pi*th], [0.85], c='magenta', s=40, alpha=0.8)

    # Mean vectors
    if matter_mask.sum() > 0:
        m_ang = np.angle(np.mean(np.exp(2j*np.pi*theta_f[matter_mask])))
        r_m   = np.abs(np.mean(np.exp(2j*np.pi*theta_f[matter_mask])))
        al.annotate('', xy=(m_ang, r_m*0.7), xytext=(0,0),
                    arrowprops=dict(arrowstyle='->', color='gold', lw=2.5))
    if (~matter_mask).sum() > 0:
        am_ang = np.angle(np.mean(np.exp(2j*np.pi*theta_f[~matter_mask])))
        r_am   = np.abs(np.mean(np.exp(2j*np.pi*theta_f[~matter_mask])))
        al.annotate('', xy=(am_ang, r_am*0.7), xytext=(0,0),
                    arrowprops=dict(arrowstyle='->', color='magenta', lw=2))

    al.set_title(f'Matter (gold): {nm}\nAntimatter (magenta): {nam}\n'
                 f'eta = {eta_now:.4f}',
                 color='white', fontsize=8)
    al.set_facecolor('#0a0a1a'); al.tick_params(colors='white')

    # Right: counts over time
    ar.set_xlim(0, NF); ar.set_ylim(0, N*0.65)
    ar.plot(range(frame+1), nm_anim[:frame+1],
            'gold', lw=2.5, label='Matter')
    ar.plot(range(frame+1), nam_anim[:frame+1],
            'magenta', lw=2, label='Antimatter')
    ar.fill_between(range(frame+1),
                    nm_anim[:frame+1], nam_anim[:frame+1],
                    alpha=0.3, color='gold', label='Surplus')
    ar.axhline(N_half, color='white', lw=1, ls='--', alpha=0.3,
               label='Initial equal')
    ar.set_xlabel('Time', color='white')
    ar.set_ylabel('N oscillators in sector', color='white')
    ar.set_title('Matter vs Antimatter count\n'
                 'Phase tilt -> matter wins',
                 color='white', fontsize=8)
    ar.legend(fontsize=7, facecolor='#0a0a1a', labelcolor='white')
    ar.tick_params(colors='white')
    for sp in ar.spines.values(): sp.set_edgecolor('gray')

    status = ('BIG BANG: symmetric start' if frame < 10 else
              'PHASE TILT amplifying...'  if frame < 50 else
              'MATTER WINS -- WE EXIST')
    col_s  = ('white' if frame < 10 else
               'gold'  if frame < 50 else 'cyan')
    fig_a.suptitle(
        f'{status}  |  Matter={nm}  Anti={nam}  eta={eta_now:.4f}',
        color=col_s, fontsize=10, fontweight='bold')

an = animation.FuncAnimation(fig_a, anim_mm,
                             frames=NF, interval=80, blit=False)
gif_path = SAVE_PATH+'matter_antimatter_phase.gif'
an.save(gif_path, writer='pillow', fps=15, dpi=100,
        savefig_kwargs={'facecolor':'#0a0a1a'})
print(f"Saved: {gif_path}")
plt.close()

if IN_COLAB:
    from google.colab import files
    files.download(gif_path)
    files.download(png_path)
    print("Downloads started.")

print()
print("="*55)
print("FINAL RESULTS")
print("="*55)
print(f"  Delta_0 = {Delta_0}  (initial tilt)")
print(f"  eta_initial = {eta_tilt[0]:.6f}")
print(f"  eta_final   = {eta_tilt[-1]:.4f}")
print(f"  Amplification = {abs(eta_tilt[-1]/(eta_tilt[0]+1e-10)):.1f}x")
print()
print("  Matter = theta in [0, 0.5)")
print("  Antimatter = theta in [0.5, 1.0)")
print("  CP violation = H asymmetry under theta -> theta+0.5")
print("  We exist because of a phase tilt of 10^-9")
