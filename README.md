# Airfoil Load Analyzer

A dependency-free browser workbench for turning supplied airfoil aerodynamic data into physical force and moment resultants.

## Open

**Hosted app:** [Airfoil Load Analyzer](https://luismezalam.github.io/project-eclipse3.0/)

You can also open `index.html` in a modern browser. Calculations run locally in the page; there is no backend or package installation.

## Hosting

The `Deploy to GitHub Pages` workflow publishes the root of the `main` branch whenever it changes. After its first successful run, GitHub Pages serves the app at the link above. To manage the deployment, open **Settings → Pages** in this repository and select **GitHub Actions** as the build and deployment source if GitHub has not selected it automatically.

## Input routes

- **2D coefficients:** enter body-axis (CA, CN) or wind-axis (CD, CL) coefficients and a section moment coefficient about the declared reference point.
- **2D surface panels:** enter an ordered, counterclockwise contour and either dimensional pressure/shear stresses (Pa) or pressure/skin-friction coefficients (Cp, Cf) at each panel endpoint. Coefficients are dimensionalized by dynamic pressure. Coordinates are normalized by chord. Constant endpoint values recover piecewise-constant panels; varying values are linearly interpolated, with exact force and moment integration over each straight panel. Six-column constant-panel CSV and eight-column endpoint CSV are accepted; pressure/shear units follow the selected input basis. A positive section-area modifier scales y/c (thickness) about the chord line, reports the resulting enclosed section area, and recomputes panel geometry loads using the same supplied pressure/shear values.
- **3D coefficients:** enter body or wind force coefficients and three moment coefficients. Each moment component uses its own declared reference length before the wrench is rotated and transported.
- **Flight state:** enter temperature and viscosity directly or apply a dry-air ISA estimate from −500 to 20,000 m. The workbench reports sound speed, Mach, chord Reynolds number, and regime flags; these indicators do not modify imported aerodynamic loads.
- **Airfoil polar library:** import CSV rows `alpha_deg,CL,CD,Cm_c/4` with an optional header. Tag the dataset's Reynolds and Mach numbers. Linear interpolation is available only inside the imported angle range; no extrapolation or condition correction is performed. The imported moment sign can be declared nose-up-positive or workbench right-hand-positive.
- **Finite-wing screening:** an optional unswept lifting-line estimate reports aspect ratio, finite-wing lift slope, linear lift, induced drag, and profile-plus-induced drag when a polar value is available. It uses the entered 2D slope, zero-lift angle, and span efficiency; it does not solve stall, sweep, taper, twist, or a 3D pressure field.
- **Longitudinal stability:** uses user-supplied whole-aircraft lift and pitching-moment slopes to estimate neutral point and static margin. Derivatives use per-radian units and nose-up-positive pitching moment; section data alone cannot account for tail or fuselage effects.
- **Convergence log:** records 2D panel or 3D triangle resolutions and compares force/moment changes against prior runs with the same operating state and reference settings. It is a manual sensitivity record, not an accuracy certification.
- **Reproducible study cases:** save a versioned JSON case containing all form inputs, route and axis selections, 2D panel rows, imported 3D triangle rows, polar data, study/source notes, and convergence records. Open a saved case to restore its inputs and recalculate it in the browser (80 MB import limit). The case stores a schema version and tool build; it does not freeze the calculation engine. Case files are processed locally; verify the health check and assumptions after reopening.
- **Dimensional polar sweep export:** export every imported polar angle row to CSV with its original and workbench-sign coefficients, source and current Reynolds/Mach values, dynamic pressure, reference area/chord, and dimensional lift, drag, and quarter-chord moment. This uses the current flow state and reference geometry; it does not correct for Reynolds/Mach mismatch or extrapolate beyond the imported angle range.
- **Study record and result export:** optionally attach a case title, student/researcher, course/lab, data citation, and run notes. The result JSON includes this record, current form settings, tool build, and computed resultants.
- **2D flow estimate:** solve a closed section contour with a Hess–Smith source/vortex panel model and sharp trailing-edge Kutta condition to estimate incompressible Cp. Approximate Cf uses smooth-flat-plate laminar/turbulent correlations with a simple transition threshold. Generated values are written to the panel table as fixed coefficients; re-run after changing operating conditions or geometry.
- **3D CFD triangle surface:** import CSV triangle geometry with pressure and body-axis wall-shear vectors at each vertex (21 values per row), or constant face traction (13 values per row). The calculator maps the values onto each triangle, projects shear into the face plane, and integrates force and moment. A header row is optional.

The triangular benchmark uses CA = 0, CN = 0.6, and Cm,LE = 0.2. Its center of pressure is at x/c = 1/3, matching a triangular pressure-difference distribution.

## Conventions and limits

- SI units: metres, pascals, kilograms per cubic metre, metres per second, newtons, and newton-metres.
- 2D body-to-wind transform: D = A cos(alpha) + N sin(alpha) and L = -A sin(alpha) + N cos(alpha).
- Surface pressure acts as -p n_out; panel shear is signed along the entered start-to-end tangent. Closed contours should be ordered counterclockwise and include all exposed faces.
- Section modifier λA maps (x/c,y/c) to (x/c,λA·y/c), so enclosed section area and thickness scale linearly while chord remains fixed. It is a geometry sensitivity tool: pressure/shear values remain as supplied and are not recomputed for the adjusted profile. The separate reference area S is the coefficient normalization area, usually planform area.
- The 2D pressure model is inviscid and incompressible; the wall-shear estimate is a leading-edge-distance flat-plate approximation. Neither models pressure-gradient boundary layers, transition physics, separation, compressibility, or shocks.
- Mach and Reynolds numbers are condition indicators only. ISA fill is a simplified dry-air state estimate; it does not substitute for measured local weather or correct a coefficient polar to another condition.
- Polar interpolation is linear and bounded by supplied angle data. Confirm the dataset's airfoil geometry, reference area, moment reference, sign, roughness/transition, Reynolds number, and Mach number before applying it.
- The finite-wing equation is an unswept, low-speed, attached-flow lifting-line screen. A swept-wing normal-Mach display is informational; no sweep correction is applied. Aircraft stability results use complete-configuration derivatives entered by the user.
- The convergence log compares only runs with matching route, flow state, area, moment references, and coefficient basis. Refinement trends do not validate source data or prove physical accuracy.
- CFD triangles use vertices ordered so the cross-product normal points outward. Pressure is gauge pressure or Cp; shear is a global body-axis vector in Pa or Cf. The tool imports mapped triangle rows; it does not read native CFD formats or solve the flow.
- The Library tab maps the 17 standard cases in the supplied general-force manual to the coefficient and surface-panel inputs, including pressure-only, shear-only, flat-plate, wedge, symmetric, blunt-base, uniform-pressure, tabulated-data, reversed-shear, and selected-moment-reference cases.
- Panel loads can be entered, imported, or generated by the approximate 2D method above. Compressible, separated, or reversed-flow fields must be supplied from another source; the 2D estimator does not solve those flow states.
- Point transport uses M_B = M_O - (r_B - r_O) x R. The reported pressure-footprint centroid is a nonnegative intensity diagnostic, separate from the physical center of pressure.
- Beam transfer assumes a straight, symmetric wing with uniform or elliptical spanwise loading. It reports root load resultants only; beam stress, deflection, native CFD solving, and arbitrary spanwise beam loading are outside this calculator. Imported CFD triangle fields can still be integrated into a total 3D wrench.
- Six-column CSV: x1/c,y1/c,x2/c,y2/c,p,tau. Eight-column endpoint CSV: x1/c,y1/c,x2/c,y2/c,p1,tau1,p2,tau2. Values are Pa or Cp/Cf according to the selected surface-data basis. A header row is optional.
- CFD per-vertex CSV: x1,y1,z1,p1,taux1,tauy1,tauz1, then the same seven fields for vertices 2 and 3. Constant face CSV: x1,y1,z1,x2,y2,z2,x3,y3,z3,p,taux,tauy,tauz. Coordinates are metres. Values are Pa or Cp/Cf according to the selected mesh-data basis.

The app follows the STRAS engineering-workbench pattern of configuration, capability disclosure, health checks, a schematic, result cards, and diagrams. Optional symmetric-wing beam estimates sit in an expandable section of Airfoil analysis and use the shared full-span input; the Library tab collects equations, definitions, sign and input rules, data handling, and model boundaries. Exported JSON includes flight-state indicators, current estimates, the loaded polar, and recorded convergence snapshots alongside the active wrench result.
