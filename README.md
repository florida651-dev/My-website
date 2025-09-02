<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>ThermoCalc – Molecular Thermodynamics (MVP)</title>
  <style>
    :root { --max: 980px; }
    body { font-family: system-ui, -apple-system, Segoe UI, Roboto, Inter, Arial, sans-serif; margin: 0; color: #0f172a; background: #f8fafc; }
    header { padding: 28px 20px; background: #0ea5e9; color: white; }
    header h1 { margin: 0 0 6px; font-size: 24px; }
    header p { margin: 0; opacity: .95; }
    main { max-width: var(--max); margin: 24px auto; padding: 0 16px 40px; }
    .card { background: white; border-radius: 14px; box-shadow: 0 6px 20px rgba(2,6,23,.06); padding: 18px; margin-bottom: 18px; }
    .grid { display: grid; gap: 14px; grid-template-columns: repeat(12, 1fr); align-items: end; }
    .col-3 { grid-column: span 3; }
    .col-4 { grid-column: span 4; }
    .col-6 { grid-column: span 6; }
    .col-12 { grid-column: span 12; }
    label { display:block; font-size: 13px; color:#334155; margin-bottom: 6px; }
    input, select { width: 100%; padding: 10px 12px; border: 1px solid #cbd5e1; border-radius: 10px; background: #fff; font-size: 14px; }
    button { padding: 10px 14px; border-radius: 10px; border: 0; background: #0ea5e9; color: white; font-weight: 600; cursor: pointer; }
    button:disabled { opacity: .6; cursor: not-allowed; }
    table { width: 100%; border-collapse: collapse; }
    th, td { padding: 10px 8px; border-bottom: 1px solid #e2e8f0; text-align: left; }
    th { background: #f1f5f9; font-weight: 600; }
    .muted { color:#475569; font-size: 13px; }
    .tag { display:inline-block; padding: 2px 8px; border-radius: 999px; background:#f1f5f9; font-size:12px; color:#334155; margin-left:8px; }
    footer { max-width: var(--max); margin: 20px auto 60px; padding: 0 16px; color:#64748b; font-size:13px; }
    .row { display:flex; gap:10px; flex-wrap: wrap; align-items:center; }
    .good { color:#059669; font-weight:600; }
    .warn { color:#b45309; font-weight:600; }
    .mono { font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; }
    details summary { cursor:pointer; }
  </style>
</head>
<body>
  <header>
    <h1>ThermoCalc (MVP)</h1>
    <p>Partition-function based thermodynamic properties at 1 bar: <span class="tag">S°</span> <span class="tag">C<sub>p</sub>°</span> <span class="tag">H°(T)−H°(0)</span></p>
  </header>
  <main>
    <section class="card">
      <div class="grid">
        <div class="col-6">
          <label for="molecule">Molecule</label>
          <select id="molecule"></select>
        </div>
        <div class="col-3">
          <label for="temp">Temperature (K)</label>
          <input id="temp" type="number" step="0.01" value="298.15" />
        </div>
        <div class="col-3">
          <label>&nbsp;</label>
          <button id="calc">Calculate</button>
        </div>
        <div class="col-12 muted">Assumptions: ideal gas, rigid rotor, harmonic oscillator (electronic ground state only; electronic degeneracy handled via <span class="mono">g<sub>el</sub></span>). Standard pressure <span class="mono">p° = 1.00000×10<sup>5</sup> Pa</span>.</div>
      </div>
    </section>

    <section class="card">
      <h3 style="margin-top:2px">Results</h3>
      <table>
        <thead>
          <tr>
            <th>Quantity</th>
            <th class="mono">Computed</th>
            <th>Units</th>
          </tr>
        </thead>
        <tbody id="results"></tbody>
      </table>
      <div class="row" style="margin-top:12px">
        <button id="downloadCsv">Download CSV (T grid)</button>
        <span class="muted">CSV covers 200–1000 K in 50 K increments.</span>
      </div>
    </section>

    <section class="card">
      <h3 style="margin-top:2px">Reference & Method</h3>
      <p class="muted">Formulas follow standard statistical thermodynamics. For a nonlinear molecule:</p>
      <ul class="muted">
        <li><span class="mono">C<sub>p</sub>° = 4R + R ∑[(θ/T)<sup>2</sup> e^{θ/T} / (e^{θ/T} − 1)^2]</span></li>
        <li><span class="mono">H°(T)−H°(0) = U + RT</span>, with <span class="mono">U = 3RT + R ∑[θ/(e^{θ/T}-1)]</span></li>
        <li><span class="mono">S° = S<sub>trans</sub> + S<sub>rot</sub> + S<sub>vib</sub> + S<sub>el</sub></span></li>
        <li><span class="mono">S<sub>trans</sub>/R = ln[((2π m k<sub>B</sub>T / h^2)^{3/2}(k<sub>B</sub>T/p°)N<sub>A</sub>)] + 5/2</span></li>
        <li><span class="mono">S<sub>rot</sub>/R = ln[(√π / σ) T^{3/2} / (θ<sub>A</sub>θ<sub>B</sub>θ<sub>C</sub>)^{1/2}] + 3/2</span></li>
        <li><span class="mono">S<sub>vib</sub>/R = ∑[ (θ/T)/(e^{θ/T}-1) − ln(1-e^{-θ/T}) ]</span>; <span class="mono">S<sub>el</sub> = R ln(g<sub>el</sub>)</span></li>
      </ul>
      <details>
        <summary>Constants & unit notes</summary>
        <ul class="muted">
          <li>R = 8.314462618 J·K⁻¹·mol⁻¹; k<sub>B</sub> = 1.380649×10⁻²³ J·K⁻¹; h = 6.62607015×10⁻³⁴ J·s; N<sub>A</sub> = 6.02214076×10²³ mol⁻¹; c = 299792458 m/s.</li>
          <li>Characteristic temps: θ = (h c / k<sub>B</sub>) × õν (cm⁻¹) with (h c / k<sub>B</sub>) = 1.438776877 K·cm.</li>
        </ul>
      </details>
    </section>
  </main>

  <footer>
    <div>© <span id="year"></span> ThermoCalc (MVP). For education & quick checks; compare against NIST/ATcT for critical work.</div>
  </footer>

  <script>
    // Physical constants (SI)
    const R = 8.314462618; // J/K/mol
    const kB = 1.380649e-23; // J/K
    const h = 6.62607015e-34; // J*s
    const NA = 6.02214076e23; // 1/mol
    const c = 299792458; // m/s
    const p0 = 1.0e5; // Pa
    const HC_over_kB_per_cm = 1.438776877; // K*cm

    function cm1_to_theta(cm1) { return HC_over_kB_per_cm * cm1; }

    // Utility: build vib theta array from cm^-1 array
    const vib = (...cm) => cm.map(cm1_to_theta);

    // Molecule database (subset for MVP). Mass in kg/mol.
    // Rotational constants in cm^-1; symmetry number sigma; electronic degeneracy g_el.
    const molecules = {
      "CH3 (radical, D3h)": {
        mass: 0.015035,
        sigma: 6,
        g_el: 2,
        rot_cm: {A: 9.57789, B: 9.57789, C: 4.74202},
        vib_cm: [3004, 606, 3161, 1396], // minimal set for MVP; extend as needed
      },
      "O3 (ozone, C2v)": {
        mass: 0.047997,
        sigma: 2,
        g_el: 1,
        rot_cm: {A: 3.55348, B: 0.44525, C: 0.39479},
        vib_cm: [1110, 705, 1042],
      },
      "cis-C2H2Cl2": {
        mass: 0.0969,
        sigma: 2,
        g_el: 1,
        // Example rotational constants (cm^-1) – replace with your best refs if desired
        rot_cm: {A: 0.384, B: 0.0849, C: 0.0695},
        vib_cm: [3077,1587,1179,711,173,876,406,697,3072,1303,857,571],
      },
      "trans-C2H2Cl2": {
        mass: 0.0969,
        sigma: 2,
        g_el: 1,
        rot_cm: {A: 1.75, B: 0.0516, C: 0.0501},
        vib_cm: [3073,1578,1274,846,350,900,227,763,3090,1200,828,250],
      },
    };

    // Populate select
    const molSel = document.getElementById('molecule');
    Object.keys(molecules).forEach(name => {
      const opt = document.createElement('option'); opt.value = name; opt.textContent = name; molSel.appendChild(opt);
    });

    document.getElementById('year').textContent = new Date().getFullYear();

    function computeProps(mol, T){
      const { mass, sigma, g_el, rot_cm, vib_cm } = mol;
      const thetaA = cm1_to_theta(rot_cm.A);
      const thetaB = cm1_to_theta(rot_cm.B);
      const thetaC = cm1_to_theta(rot_cm.C);
      const thetas = vib_cm.map(cm1_to_theta);

      // U (molar)
      let U = 3*R*T; // trans + rot for nonlinear = 3RT
      // vibrational U contribution
      for(const th of thetas){
        const x = th / T; const ex = Math.exp(x); U += R * th / (ex - 1);
      }

      // Cp (molar)
      let Cp = 4*R; // base for nonlinear
      for(const th of thetas){
        const x = th / T; const ex = Math.exp(x);
        Cp += R * (x*x) * ex / ((ex - 1)*(ex - 1));
      }

      // Entropy pieces
      // Translational entropy (molar)
      const translFactor = Math.pow((2*Math.PI* (mass/NA) * kB*T)/(h*h), 1.5) * (kB*T/p0) * NA; // dimensionless
      const S_trans = R * (Math.log(translFactor) + 2.5);

      // Rotational entropy (nonlinear rigid rotor)
      const q_rot = Math.sqrt(Math.PI) / sigma * Math.pow(T, 1.5) / Math.sqrt(thetaA*thetaB*thetaC);
      const S_rot = R * (Math.log(q_rot) + 1.5);

      // Vibrational entropy
      let S_vib = 0;
      for(const th of thetas){
        const x = th / T; const ex = Math.exp(x);
        S_vib += x/(ex - 1) - Math.log(1 - Math.exp(-x));
      }
      S_vib *= R;

      // Electronic entropy (degeneracy only)
      const S_el = R * Math.log(g_el || 1);

      const S = S_trans + S_rot + S_vib + S_el; // J/K/mol
      const HminusH0 = U + R*T; // J/mol

      return { U, Cp, S, HminusH0 };
    }

    function fmt(x, digits=2){ return x.toFixed(digits); }

    function render(T){
      const mol = molecules[molSel.value];
      const {U, Cp, S, HminusH0} = computeProps(mol, T);
      const tbody = document.getElementById('results');
      tbody.innerHTML = '';
      const rows = [
        ["H°(T) - H°(0)", fmt(HminusH0/1000, 3), "kJ/mol"],
        ["S°", fmt(S, 2), "J/(K·mol)"],
        ["Cₚ°", fmt(Cp, 2), "J/(K·mol)"],
      ];
      for(const r of rows){
        const tr = document.createElement('tr');
        r.forEach((cell,i)=>{
          const td = document.createElement(i===0? 'th':'td'); td.textContent = cell; tr.appendChild(td);
        });
        tbody.appendChild(tr);
      }
    }

    document.getElementById('calc').addEventListener('click', () => {
      const T = parseFloat(document.getElementById('temp').value || '298.15');
      render(T);
    });

    // CSV download for a T grid (200..1000 K step 50)
    document.getElementById('downloadCsv').addEventListener('click', () => {
      const mol = molecules[molSel.value];
      let csv = 'T (K),H(T)-H(0) (kJ/mol),S (J/K/mol),Cp (J/K/mol)\n';
      for(let T=200; T<=1000; T+=50){
        const {HminusH0, S, Cp} = computeProps(mol, T);
        csv += `${T},${(HminusH0/1000).toFixed(4)},${S.toFixed(3)},${Cp.toFixed(3)}\n`;
      }
      const blob = new Blob([csv], {type:'text/csv'});
      const a = document.createElement('a');
      a.href = URL.createObjectURL(blob);
      a.download = `${molSel.value.replace(/\s+/g,'_')}_thermo.csv`;
      a.click();
      URL.revokeObjectURL(a.href);
    });

    // Initial render
    molSel.selectedIndex = 0; render(298.15);
  </script>
</body>
</html>
