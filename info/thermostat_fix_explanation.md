# Thermostat input/output handling fix

Recent updates adjusted thermostat switch handling to address interference when multiple controllers are defined.

## What changed
- **Input selection** now validates and reads the thermostat's configured switch index directly instead of reusing the first switch slot.
- **Output state reads** now check the relay index before reading `TasmotaGlobal.power`, returning `0` for invalid indices instead of reading arbitrary bits.

## Why this fixes interference
Previously, every thermostat controller fetched its input from the first switch entry (`SWT1`) regardless of its own configuration. When two controllers were configured with different input switches, both would react to the same physical switch, so one controller could toggle or reset the other unexpectedly. The corrected logic subtracts the `THERMOSTAT_INPUT_SWT1` offset from the specific switch passed to `ThermostatInputStatus`, so each thermostat samples only its assigned input.

Similarly, output status checks previously read `TasmotaGlobal.power` without confirming the relay index. If a thermostat referenced a relay number outside the available range, the bit read could correspond to another relay or garbage data, causing one thermostat's output check to reflect another's state. Adding `ThermostatRelayIdValid` ensures the function returns a neutral `0` when the relay id is invalid, preventing cross-controller contamination of output state.

## New troubleshooting helpers
- The `InputSwitchUse` command now validates the thermostat index just like other thermostat setters, so you can reliably enable or disable input-triggered manual mode on any controller.
- A new `Thermostat<x>` command (for example, `Thermostat1`) reports each controller's configured input switch, relay, output enable flag, and their current states. This makes it easier to confirm that multiple thermostats are mapped to distinct hardware and to spot unintended overlaps.
