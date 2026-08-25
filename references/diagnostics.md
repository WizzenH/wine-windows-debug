# Adaptive Wine diagnostics

Start narrow and add channels or tools only when the current evidence justifies them. Keep every log tied to one run ID and exact command.

## Baseline signals

Capture ordinary stdout/stderr and container logs first. Record Wine version, binary resolution, prefix, environment overrides, process exit, signal, and timestamps. Check that the selected executable and working directory are correct.

Useful early Wine channels commonly include process/thread timing, exceptions, module loading, and server interactions. Select only the subset relevant to the symptom, for example:

```text
WINEDEBUG=+timestamp,+pid,+tid,+seh,+loaddll,+module
```

Treat this as an example, not a universal default. Reduce channels once the failing subsystem is known.

## Symptom-driven escalation

| Symptom | Evidence to collect next |
|---|---|
| Crash or hang | exception trace, backtrace/core/minidump, process/thread state, targeted `strace`, last successful API boundary |
| Missing DLL or entry point | `+loaddll`, module search paths, bitness, overrides, PE imports, package contents |
| D3D/Vulkan/OpenGL | adapter enumeration, Vulkan/OpenGL capability tools, driver/ICD selection, DXVK/VKD3D logs, targeted Wine graphics channels |
| Window/input/display | X11/Wayland driver selection, display variables, window manager events, DPI, focus/input channels |
| Audio | device enumeration, PipeWire/Pulse/ALSA visibility, Wine audio driver, format negotiation |
| Network | DNS/socket evidence, proxy/TLS/certificate state, targeted socket trace, container network policy |
| Installer/runtime | MSI logs, prerequisite detection, filesystem/registry access, architecture and reboot-pending state |
| Fonts/locale/text | installed fonts, locale/codepage, font substitution, shaping/rendering evidence |
| Performance/timing | controlled repeated timing, CPU/GPU counters, synchronization mode, logging overhead comparison |

Use [components.md](components.md) to decide which boundary owns the next probe and which apparent fixes would only mask the symptom. When evidence identifies a Wine source component, inspect its relevant commit history before assuming the current behavior is an unimplemented feature.

Do not use full `+relay` indiscriminately. If function-level relay is needed, constrain it to the suspected module or call boundary and keep the reproduction short.

## Mutation ladder

Test the lowest adequate layer first:

1. launch arguments and environment variables;
2. dedicated-prefix configuration, DLL override, component install, or registry key;
3. container package, device, mount, permission, image, or runtime configuration;
4. Wine source change.

For every mutation, capture the pre-state, exact action, post-state, and rollback. Recreate a clean prefix or container when contamination is plausible instead of piling changes onto an unknown state.

## Evidence quality

Strong evidence changes one variable and predicts the result. Weak evidence is a large configuration bundle that happens to work. When a bundle is unavoidable, bisect it until the necessary subset is known before documenting the final solution.

Separate:

- **observation** — directly measured output;
- **inference** — explanation supported by observations;
- **decision** — why the next experiment was selected.

