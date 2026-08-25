# Wine component responsibility map

Use this map to choose the first evidence boundary and to avoid applying a fix in a layer that cannot own the failure. Components overlap; a log emitted by one component may report a failure caused by the next layer down.

| Component or layer | Problems it can solve | Typical evidence | Usually not sufficient for |
|---|---|---|---|
| Application and launcher | Wrong arguments, working directory, configuration, feature flags, application-side bugs, missing data or services | application logs, command line, config diff, native-Windows comparison | Missing Wine APIs, host driver faults, container device isolation |
| Launch environment | Wrong `PATH`, `WINEPREFIX`, `WINEDLLOVERRIDES`, locale, display/audio variables, proxy, runtime library lookup | resolved executables/libraries, environment diff, wrapper behavior | Durable registry state, missing packages, Wine implementation defects |
| Wine prefix and registry | Feature detection, COM registration, DLL overrides, Windows version mode, per-user settings, prefix corruption or wrong bitness | clean-prefix A/B, registry export/diff, `wineboot`, module load trace | Missing Unix libraries, broken GPU/ICD, source-level API behavior |
| PE loader and builtin/native DLL selection | Missing DLL or entry point, wrong architecture, load order, side-by-side/runtime selection | `+loaddll`, PE imports, module paths, overrides, bitness | Window-system or GPU failures after a module loads correctly |
| Wine core (`ntdll`, `kernelbase`, `kernel32`, wineserver) | Process/thread, memory, exceptions, handles, files, synchronization, IPC/server semantics | `+seh`, server traces, backtrace, syscall/file traces, targeted tests | D3D shader bugs, missing container devices, application content errors |
| Win32 UI (`user32`, `win32u`, `gdi32`) | Window creation, focus, input dispatch, DPI, clipboard, menus, GDI drawing | window/input traces, DPI and focus state, minimal GUI reproducer | Vulkan translation defects, audio negotiation, installer prerequisites |
| Host window driver (X11/Wayland) | Display connection, window mapping, cursor/grab, monitor enumeration, compositor integration | display variables, driver selection, X11/Wayland events, compositor comparison | Application logic bugs or missing PE dependencies |
| Direct3D/OpenGL/Vulkan translation | D3D feature behavior, device creation, presentation, shaders, formats, synchronization at graphics API boundaries | WineD3D/DXVK/VKD3D logs, API captures, adapter enumeration, targeted graphics tests | Absent `/dev/dri`, wrong host driver/ICD, generic window focus problems |
| Host graphics stack | Kernel GPU device, Mesa/proprietary driver, Vulkan ICD, OpenGL implementation, hardware capability | capability tools, driver/ICD resolution, kernel logs, native API smoke test | Incorrect Wine registry/override or application-specific assumptions |
| Audio stack | Endpoint enumeration, backend selection, format negotiation, latency and stream lifecycle | PipeWire/Pulse/ALSA visibility, Wine audio trace, device/format A/B | Graphics or windowing problems, missing application runtime DLLs |
| Input and device integration | Keyboard/mouse/controller/raw input/HID discovery, mapping, permissions and grabs | device nodes, udev/permissions, input traces, focused minimal test | UI layout bugs or application networking failures |
| Networking and TLS | DNS, sockets, proxy, certificates, WinHTTP/WinINet behavior, container network policy | DNS/socket trace, certificate store, proxy state, native endpoint test | Offline crashes unrelated to network or missing graphics capabilities |
| Installer and Windows runtimes | MSI actions, prerequisites, COM/.NET/VC runtime detection, reboot-pending state | MSI log, prerequisite registry/files, architecture, clean install trace | Runtime defects after installation has completed correctly |
| Fonts, locale, and text | Font discovery/substitution, codepages, shaping, fallback, locale-dependent behavior | installed fonts, locale/codepage, font trace, screenshot/text comparison | GPU device loss or process synchronization faults |
| Container image and runtime | Missing Unix packages, mounts, devices, permissions, seccomp, shared memory, network, display/audio sockets | image digest, package manifest, mounts/devices, runtime config, container/host A/B | A reproducible Wine source regression under identical host conditions |
| Wine source and build | Missing or incorrect Windows API semantics, subsystem regressions, architecture-specific implementation defects | minimal reproducer, source tests, path/symbol history, controlled good/bad builds, bisect | Problems already isolated to the application, prefix, container, or host driver |

## Boundary rules

- Prefer a clean A/B at the boundary: application native vs Wine, host vs container, clean vs existing prefix, system Wine vs branch-built Wine, WineD3D vs DXVK only when both are supported comparisons.
- A workaround in a higher layer does not prove the lower layer is correct. For example, disabling a feature may avoid a graphics regression without fixing it.
- A lower-layer error does not automatically assign root cause there. An invalid application parameter can surface as a driver or Wine error.
- Escalate to source history only after identifying a plausible Wine component or observing a revision-dependent behavioral change.

## Commit-history routing by component

Once a Wine-owned component is plausible, narrow history to the implementing paths and tests rather than searching the entire repository. Examples include the relevant `dlls/<module>/`, `server/`, `programs/`, `loader/`, or architecture-specific files. Search the symbol, behavior-defining constant, registry key, protocol field, and nearby test name. Include renamed paths and downstream patch directories when the runtime is not a pristine upstream build.

