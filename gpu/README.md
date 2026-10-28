# GPU metrics

GPU engine utilization, adapter memory, and process GPU memory on Windows (WDDM).

Status: all entries unverified.

Source of GPU counter definitions: System Monitor counters exposed by WDDM and surfaced by PIX (https://learn.microsoft.com/windows/pix/timing-captures/timing-captures-overview#gpu-profiling).

## Engine utilization

### GPU Engine Utilization Percentage

- **Unit:** percent per GPU engine instance (3D, Copy, Video Decode, Video Encode, Compute).
- **Formula:** Busy time of the engine / elapsed time × 100, per engine (adapter LUID and engine type form the instance name).
- **Source:** Performance counter set `GPU Engine` (instance names include `phys_<n>_eng_<n>_engtype_<type>`).
- **Reference:** https://learn.microsoft.com/windows/win32/direct3dtools/pix/articles/timing-captures/pix-timing-captures#gpu-profiling (lists GPU utilization by engine)
- **Pitfall:** Utilization is per engine, not per adapter. A busy copy engine with idle 3D engine looks different from a fully loaded GPU. Sum or max across engines depends on the question. Engine instance names vary by driver.

### D3D10_COUNTER values (legacy)

- **Unit:** percent.
- **Source:** Direct3D 10 query counters: `D3D10_COUNTER_GPU_IDLE`, `D3D10_COUNTER_VERTEX_PROCESSING`, `D3D10_COUNTER_GEOMETRY_PROCESSING`, `D3D10_COUNTER_PIXEL_PROCESSING`, `D3D10_COUNTER_OTHER_GPU_PROCESSING`.
- **Reference:** https://learn.microsoft.com/windows/win32/api/d3d10/ne-d3d10-d3d10_counter
- **Pitfall:** Legacy API, Direct3D 10 only. Use GPU Engine counters for current systems.

## Adapter memory

### GPU Adapter Memory: Dedicated Usage, Shared Usage

- **Unit:** bytes.
- **Formula:** Dedicated Usage = VRAM in use. Shared Usage = system memory mapped for GPU use.
- **Source:** Performance counter set `GPU Adapter Memory` (instance per adapter LUID).
- **Reference:** https://learn.microsoft.com/windows-hardware/drivers/display/memory-charts (local video memory vs system memory on adapter charts); https://learn.microsoft.com/windows/win32/direct3dtools/pix/articles/timing-captures/pix-timing-captures#gpu-profiling (memory counters).
- **Pitfall:** Shared usage is not a dedicated resource. A high Shared Usage with full Dedicated Usage means the GPU spills to system memory and performance drops.

### GPU Process Memory (per process)

- **Unit:** bytes.
- **Formula:** Dedicated GPU memory and shared GPU memory charged to a process.
- **Source:** Performance counter set `GPU Process Memory`; Task Manager Details pane (`Dedicated GPU memory`).
- **Reference:** https://learn.microsoft.com/troubleshoot/windows-client/performance/gpu-process-memory-counters-report-wrong-value (known issue: values can rise after apps flush caches; Windows 10).
- **Pitfall:** The source documents a known Windows 10 issue where the reported value climbed by about 100 MB each time an Office app re-activated. Confirm against WPR/WPA before reporting a leak.

## Presentation

### Frames per second, MsBetweenPresents, MsUntilRenderComplete

- **Unit:** frames per second; milliseconds.
- **Formula:** MsBetweenPresents = time between present calls. FPS = 1000 / MsBetweenPresents. MsUntilRenderComplete = time from present start to GPU completion.
- **Source:** PIX SysMon counters (captured with "Capture SysMon counters" in timing captures).
- **Reference:** https://learn.microsoft.com/windows/win32/direct3dtools/pix/articles/timing-captures/pix-timing-captures#gpu-profiling
- **Pitfall:** Presentation metrics come from PIX captures, not from base Windows counters. Require the app to call Present.

## Residency and migrations

### Residence operations, demoted allocations, allocation migrations (PIX timing capture)

- **Unit:** events (count) or durations.
- **Source:** PIX timing capture, Range Details categories: Residence Operations, Demoted Allocations, Allocation Migrations.
- **Reference:** https://learn.microsoft.com/windows/pix/timing-captures/timing-captures-overview#gpu-profiling (demoted allocations occur when DXGK cannot allocate in VRAM; migrations require suspending the GPU).
- **Pitfall:** Migrations are expensive. A count of demoted allocations > 0 under load is a memory-pressure signal for the GPU.

## Windows Internals and driver references

- **WDDM architecture:** the kernel-mode driver (dxgkrnl, the graphics kernel) schedules GPU work and manages VRAM. Reference: https://learn.microsoft.com/windows-hardware/drivers/display/windows-display-driver-model-architecture
- **GPUView:** visualizes GPU queues and memory charts per adapter and process. Reference: https://learn.microsoft.com/windows-hardware/drivers/display/memory-charts
- **Sysinternals:** Process Explorer shows per-process GPU use on Windows 10+. https://learn.microsoft.com/en-us/sysinternals/downloads/process-explorer
