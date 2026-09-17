c60_ratchet_project — file listing
===================================

Simulation inputs (run in this order):
  1. Bring_to_TEMP.input   - minimizes structure, thermalizes to 90K, writes restart
                              (corrected: adds per-atom PE/stress computes for OVITO)
  2. c60_ratchet.input     - reads restart, drives the negx pusher, runs 1,000,000 steps
                              (corrected: same per-atom outputs added, fixed missing
                              write_restart filename bug from original upload)

Supporting files (must stay in the same directory as the .input scripts):
  c60_ratchetOP.data       - initial atomic structure (61,212 atoms, hydrocarbon bearing)
  CH.airebo                - AIREBO potential parameter file (Stuart/Brenner C-H potential)

analysis/ - geometry renders produced while inspecting the structure:
  geometry_overview.png    - full system, 3 orthographic views, colored by group
  wheel_zoom.png           - zoom on the wheel/cap end of one shaft
  wheel_endon.png          - end-on view down the shaft axis, showing the nut/bearing collar

Run order:
  lmp -in Bring_to_TEMP.input -log log.bring2temp
  lmp -in c60_ratchet.input   -log log.ratchet
