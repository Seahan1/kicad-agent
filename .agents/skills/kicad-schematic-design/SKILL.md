---
name: kicad-schematic-design
description: Create, edit, inspect, and validate KiCad schematics for digital, analog, power, interface, and mixed-signal circuits. Use for requests to open KiCad, create or draw schematics, place library symbols, assign footprints, define external interfaces, wire nets, review screenshots, or diagnose ERC and netlist issues.
---

# KiCad Schematic Design

Use this skill for every KiCad schematic task in the active workspace. Treat the schematic as an electrical contract, a manufacturing input, and a document that another engineer can read without guessing.

## Operating Modes

Choose the mode from the latest user request.

- **Create or edit:** inspect the project, define requirements, make the requested changes, validate, and show the changed-file diff.
- **Review or diagnose:** perform read-only inspection and validation. Do not edit the schematic, project, PCB, reports, or settings.
- **Open or inspect:** open KiCad and inspect the visible state. Do not infer permission to edit from an open request.

Respect workspace instructions and preserve user changes. Do not overwrite an existing project until the requested target is explicit.

## Design Contract Before Drawing

Collect or state assumptions for:

- Circuit function, signal direction, operating modes, and failure behavior.
- Every external net: source, sink, connector or port, voltage or logic level, and expected current or frequency.
- Power rails: minimum, nominal, and maximum voltage; continuous and peak current; startup and shutdown behavior.
- Environment, temperature, mechanical constraints, EMC constraints, and board manufacturing limits.
- Component MPN, datasheet, tolerance, voltage rating, package, lifecycle, and approved alternatives.

Create an endpoint ledger before placing symbols. Each external net must have a visible source and destination, or an explicit hierarchical or global port. A rail label alone does not define where power enters the circuit. For a board-level circuit, expose required inputs, outputs, grounds, buses, test points, and measurement nodes with connectors or ports.

When a requirement is unknown, record the assumption and mark the affected claim unverified. Do not silently invent load current, connector pinout, operating temperature, or external source behavior.

## Project and Library Rules

1. Inspect `.kicad_pro`, `.kicad_sch`, `.kicad_pcb`, `sym-lib-table`, and `fp-lib-table` before placement.
2. Use installed KiCad symbol and footprint libraries through the KiCad GUI, local KiCad MCP, or a library parser. Record every exact `lib_id` and footprint identifier.
3. Resolve inherited symbols through their declared parent definition. Preserve all pins, graphics, electrical types, UUIDs, and instance records.
4. Compare critical IC, connector, protection, sensor, and power-device pin maps with the manufacturer datasheet. Library consistency alone does not establish physical correctness.
5. Assign complete library footprint identifiers. Reject malformed or concatenated library and footprint names.
6. Use one authoritative reference per component. Annotate after placement and recheck duplicate references.
7. Use real MPNs for fabrication designs. Keep generic symbols only when the user accepts a conceptual schematic.

Treat a missing exact footprint as a blocking condition. Notify the user immediately when no installed footprint matches the component package, pin count, pad numbering, exposed pad, and assembly method. Report the component, required package, searched libraries, and available choices. Do not silently substitute a similar footprint, leave the footprint blank, or create a temporary footprint without user approval.

## Drawing Workflow

Follow this sequence once, with a checkpoint after each stage:

1. Inspect the project and determine the target file.
2. Write the endpoint ledger and net naming plan.
3. Place the circuit structure: external connectors or ports, major ICs, sources, loads, protection, and test points.
4. Place passive and support components next to the pins they serve.
5. Confirm exact footprints for every fabrication component before completing the drawing. Stop and notify the user when an exact footprint is unavailable.
6. Connect wires and labels from the ledger. Add junctions only at real T connections.
7. Assign values, MPNs, footprints, datasheets, and assembly properties.
8. Save, reopen, run validation, and inspect the rendered schematic.

Do not restart from a blank schematic after a local error. Preserve the last valid checkpoint, identify the exact failing object or operation, and make the smallest targeted correction. Keep a temporary working copy when experimenting with format conversion or library extraction.

## Prevent Repeated Redrawing

Repeated redraws usually come from five process failures:

- The source and destination endpoints were not defined before placement, so missing interfaces are discovered late.
- GUI operations and direct file edits were mixed while KiCad held stale in-memory state, causing overwrites, duplicates, or old geometry to return.
- Accessibility element indexes, cursor positions, or symbol chooser selections were reused after the UI changed.
- A generated S-expression contained incomplete inherited symbols, invalid pin arrays, malformed footprints, or stale UUID records.
- ERC and visual checks were postponed until the whole drawing was rebuilt, allowing several errors to accumulate.

Apply these controls:

- Choose one editing path for the task: GUI, KiCad MCP, or validated file transformation. Do not switch paths during an active edit without saving and closing the current KiCad editor.
- After every GUI action that changes the dialog or canvas, fetch fresh app state before using an element index. Confirm the attached symbol, reference, and value before placing it.
- Keep a stage log: baseline, structure, connectivity, properties, validation. Record the first failure and continue from the last valid stage.
- Never use a fallback symbol, guessed pin list, fake UUID set, or substitute footprint to make a failed generation appear complete.
- Validate each atomic group before placing the next group. A failed group must be repaired in place or discarded as a controlled checkpoint operation.
- If a file-level transformation fails to load, stop editing that file, preserve it for diagnosis, and reopen the last KiCad-generated checkpoint.

## External Interfaces and Net Connectivity

Arrange the main signal or power flow from source to destination. Put external interfaces at the page edge or at clearly marked ports.

For every externally supplied rail or signal:

- Choose one source or interface representation. A physical connector or explicit port defines where an external rail enters or leaves the board; a standalone power symbol defines a rail without a physical interface.
- When a connector defines the physical entry or exit, identify the connected wire with a net label when needed. Do not place an additional voltage power symbol beside that connector as a second source or destination representation.
- Show the connector, hierarchical label, or global label that defines the entry point.
- Show the destination component or output interface.
- Use a `PWR_FLAG` only on a net that is externally driven and connect its pin to the source wire or source-net label at the pin endpoint.
- Keep the source marker visible and away from unrelated text. A floating-looking `PWR_FLAG` is a review finding even when same-name labels merge nets.
- Use readable net names and consistent voltage or signal-domain naming.

## Power Symbol Semantics and Placement

Use the standard library symbol that matches the electrical meaning:

- Use `power:GND` for ground. Do not represent ground with `PWR_FLAG`, a text label, or a generic power marker.
- Use the project's standard power symbols for named rails when available. Use global or hierarchical labels for nets that cross sheets or require a project-specific name.
- Use `PWR_FLAG` only as an ERC source declaration. It does not identify ground, a voltage rail, a connector, or a load.
- Keep a rail name, a power symbol, and a PWR_FLAG as separate concepts. Do not stack their texts at one pin or use a PWR_FLAG as the visible rail interface.

Follow human schematic placement conventions:

- Place the symbol pin exactly on a wire endpoint or a real junction.
- Put a ground symbol at the end of a short ground branch, usually below the circuit. Keep the ground symbol visually attached to that branch.
- Put a power-rail symbol at the end of a rail or at the end of a short branch from a real junction. For an inline rail connection, terminate the wire at the symbol pin.
- Do not drop a power or ground symbol on the middle of a wire without a junction and a deliberate branch.
- Do not rely on a same-name label to make a floating-looking symbol appear connected. The symbol pin, wire endpoint, junction, and label must agree in position and meaning.
- Keep symbols on the grid, orient them consistently, and leave enough space for the symbol name and value to remain readable.

For every component pin in scope:

- Connect it to a wire, label, power symbol, no-connect marker, or documented intentional omission.
- Verify pin number, pin name, electrical type, and actual net in the raw schematic and analyzer output.
- Keep wire endpoints on the grid. Add junctions at actual branch points.

For boards, include input, output, ground, programming, communication, and measurement interfaces required by the stated use. Do not leave a functional block visually isolated from its external source or load.

## Component and Parameter Rules

For every active component, verify the datasheet-required support network, operating limits, protection, enable state, reset state, biasing, and unused-pin treatment.

For every passive component, specify or flag:

- Value, tolerance, voltage or current rating, technology, package, MPN, and lifecycle.
- Temperature coefficient, DC-bias derating, ESR, ripple-current rating, or frequency behavior when relevant.
- The design reason for the value and the required corner cases.

For regulators, converters, references, amplifiers, oscillators, ADCs, transceivers, and protection parts, validate the application circuit against the exact manufacturer datasheet. ERC cannot prove loop stability, timing margin, thermal safety, EMC performance, or physical pinout correctness.

## Thermal and Electrical Margins

Calculate component stress from the actual circuit conditions. For a dissipative device:

```text
P_loss = sum(voltage_drop * current) + static_power
Tj = Ta + P_loss * thetaJA_effective
```

Evaluate minimum and maximum supply, continuous and peak load, startup, short-circuit, minimum-load, temperature, and tolerance corners when applicable. Use PCB copper, airflow, package, and ambient assumptions for `thetaJA_effective`.

Do not approve a component from its headline current or voltage rating alone. State the missing load, thermal, derating, or transient data when it prevents a conclusion.

## Drawing Quality

Use a grid-supported layout with consistent spacing.

- Keep source to destination flow left to right where the circuit allows it.
- Put ground below or as a clear return path and keep it visibly continuous.
- Keep external inputs and outputs on the page edge or at labeled ports.
- Keep reference, value, pin numbers, pin names, net labels, and power markers from overlapping.
- Place power and ground symbols at wire endpoints or real junctions. Review their symbol type and pin position separately from the net name.
- Put labels at readable wire endpoints or pin endpoints with intentional orientation.
- Use short direct wires for critical paths. Avoid decorative loops and ambiguous crossings.
- Keep the full circuit centered in the active sheet area and make the first viewport useful.
- Use connector and test-point symbols where the physical board needs attachment or measurement.

Before handoff, inspect a zoomed screenshot or PDF. Read every critical pin name, net label, reference, value, source marker, input, and output without relying on hidden properties.

## Editing Method

Prefer KiCad GUI actions or local KiCad MCP operations that add symbols from installed libraries. Use direct S-expression editing only when the file format, embedded library definitions, inherited symbols, UUIDs, instance records, and project path are known and the result will be opened by KiCad for validation.

When file transformation is necessary:

1. Start from a complete KiCad-generated file.
2. Preserve embedded `lib_symbols`, instance metadata, pin UUIDs, project paths, and KiCad version fields.
3. Use real UUIDs and valid KiCad S-expression structure.
4. Close the KiCad editor before changing the file externally, then reopen it after the change.
5. Keep the previous valid file until the new file passes load, ERC, analyzer, and visual checks.

Never create a partial schematic containing guessed symbol blocks. Never use fake pin arrays, invalid footprint strings, unconnected helper wires, or a file that KiCad has not parsed successfully.

## Verification Gate

Run all applicable checks after creation or modification:

1. Save and reopen the schematic in the installed KiCad version.
2. Upgrade or resave with the installed `kicad-cli` when the format is older.
3. Run ERC with all severities and return a nonzero exit code for violations:

```bash
/Applications/KiCad/KiCad.app/Contents/MacOS/kicad-cli sch erc --severity-all --exit-code-violations schematic.kicad_sch
```

4. Run the KiCad schematic analyzer when available. Compare component count, net count, pin-to-net mapping, external endpoint coverage, rail-source findings, footprint fields, and component findings with the raw schematic.
5. Audit every external net: source, destination, connector or port, label, ground return, and test access.
6. Audit power semantics: `power:GND` for ground, standard power symbols for named rails, and `PWR_FLAG` only for ERC source declaration.
7. Check every power or ground symbol for endpoint or junction placement, grid alignment, orientation, and readable text.
8. Confirm that every fabrication component has an installed footprint matching package, pin count, pad numbering, exposed pad, and mounting technology.
9. Check the raw file for every critical pin, wire endpoint, label, junction, source marker, reference, and footprint.
10. Export a PDF or screenshot and inspect the rendered drawing at a readable zoom.
11. When a PCB exists, compare schematic footprints and pin-to-net assignments with PCB pads. Run PCB DRC when the PCB is in scope.
12. Run thermal, timing, signal-integrity, or power-integrity analysis when the circuit and available data support it. State missing inputs when they prevent a conclusion.

Treat ERC and analyzer output as separate evidence. Report both when they disagree. Passing ERC does not establish datasheet compliance, thermal safety, connector usability, complete external interfaces, or physical pinout correctness.

## Review Output

Lead with the highest-risk findings. For each finding, include:

- Severity: blocker, high, medium, low, or informational.
- Evidence: screenshot observation, raw-file line, ERC message, analyzer rule, datasheet section, or calculation.
- Impact: what could fail electrically, thermally, in manufacturing, or during review.
- Required decision: what specification, datasheet, or physical assumption must be resolved.

Report positive checks separately: component library identity, pin-to-net topology, external endpoint coverage, ERC result, footprints, source declarations, parameter completeness, and visual readability.

For a read-only review, state explicitly that no files were modified. For a create or edit request, show the changed-file diff after validation passes. Do not report completion while KiCad sessions or required validation commands are still running.
