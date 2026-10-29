# GPU metrics

GPU engine utilization, adapter memory, and process GPU memory on Windows (WDDM).

Status: source-checked against Microsoft Learn on 2026-10-29 for the behavior described. Counter-set names and instance formats are not confirmed by a Microsoft page. Not yet verified on a live machine. Status values are defined in the [root README](../README.md#verification-status).

Primary source for GPU counter families and what they measure: [Profile the CPU and GPU with timing captures](https://learn.microsoft.com/windows/pix/timing-captures/timing-captures-overview#gpu-profiling) (PIX). It lists GPU utilization by engine, presentation counters, GPU memory, residency, and allocation migrations. Enable "Capture SysMon counters" in a PIX timing capture to collect the System Monitor GPU counters.

## Engine utilization

### GPU Engine Utilization Percentage

- **Unit:** percent per GPU engine instance (3D, Copy, Video Decode, Video Encode, Compute).
- **Formula:** Busy time of the engine / elapsed time × 100, per engine.
- **Source:** Performance counter set `GPU Engine`.
- **Reference:** [PIX timing captures: GPU utilization and presentation counters](https://learn.microsoft.com/windows/pix/timing-captures/timing-captures-overview#gpu-profiling) (utilization broken down by GPU engine). Instance name format was not checked on this pass.
- **Status:** source-checked (behavior); instance format not checked.
- **Pitfall:** Utilization is per engine, not per adapter. A busy copy engine with an idle 3D engine looks different from a fully loaded GPU. Choose sum or max across engines based on the question.

### D3D10_COUNTER values (legacy)

- **Unit:** percent.
- **Source:** Direct3D 10 query counters: `D3D10_COUNTER_GPU_IDLE`, `D3D10_COUNTER_VERTEX_PROCESSING`, `D3D10_COUNTER_GEOMETRY_PROCESSING`, `D3D10_COUNTER_PIXEL_PROCESSING`, `D3D10_COUNTER_OTHER_GPU_PROCESSING`.
- **Reference:** [D3D10_COUNTER enumeration](https://learn.microsoft.com/windows/win32/api/d3d10/ne-d3d10-d3d10_counter).
- **Status:** partly checked (reference exists; values not read).
- **Pitfall:** Legacy API, Direct3D 10 only. Use GPU Engine counters for current systems.

## Adapter memory

### GPU Adapter Memory: Dedicated Usage, Shared Usage

- **Unit:** bytes.
- **Formula:** Dedicated Usage = local video memory in use. Shared Usage = system memory mapped for GPU use.
- **Source:** Performance counter set `GPU Adapter Memory` (instance per adapter). Counter names were not checked against a Microsoft page.
- **Reference:** [Memory charts (WDDM)](https://learn.microsoft.com/windows-hardware/drivers/display/memory-charts) (adapter memory charts show local video memory and system memory in separate colors). [PIX timing captures: GPU memory](https://learn.microsoft.com/windows/pix/timing-captures/timing-captures-overview#gpu-profiling) (local and non-local memory usage, residence, and budgets).
- **Status:** source-checked (concepts); counter names not checked.
- **Pitfall:** Shared usage is system memory, not a dedicated resource. A high Shared Usage with full Dedicated Usage means the GPU spills to system memory and performance drops.

### GPU Process Memory (per process)

- **Unit:** bytes.
- **Formula:** Dedicated GPU memory and shared GPU memory charged to a process.
- **Source:** Performance counter set `GPU Process Memory`; Task Manager Details pane (`Dedicated GPU memory`).
- **Reference:** [GPU process memory counters report incorrect values](https://learn.microsoft.com/troubleshoot/windows-client/performance/gpu-process-memory-counters-report-wrong-value) (KB 4490156; applies to Windows 10 version 1709 and later). The article says that after an application flushes its caches, the reported value can rise by about 100 MB or more each time the app is reactivated. Task Manager's Performance pane, WPR, and WPA show the expected value.
- **Status:** source-checked.
- **Pitfall:** Confirm against the Performance pane, WPR, or WPA before reporting a GPU memory leak from these counters.

## Presentation

### Frames per second, MsBetweenPresents, MsUntilRenderComplete

- **Unit:** frames per second; milliseconds.
- **Formula:** MsBetweenPresents = time between present calls. FPS = 1000 / MsBetweenPresents. MsUntilRenderComplete = time from present start to GPU work completion.
- **Source:** PIX SysMon counters (captured with "Capture SysMon counters" in timing captures).
- **Reference:** [PIX timing captures: presentation counters](https://learn.microsoft.com/windows/pix/timing-captures/timing-captures-overview#gpu-profiling) (frames per second, MsBetweenPresents, MsUntilRenderComplete).
- **Status:** source-checked.
- **Pitfall:** These presentation metrics come from applications that call Present. They are not base Windows counters.

## Residency and migrations

### Residence operations, demoted allocations, allocation migrations (PIX timing capture)

- **Unit:** events (count) or durations.
- **Source:** PIX timing capture, Range Details categories: Residence Operations, Demoted Allocations, Allocation Migrations.
- **Reference:** [PIX timing captures: residency](https://learn.microsoft.com/windows/pix/timing-captures/timing-captures-overview#gpu-profiling) (demoted allocations occur when the graphics kernel cannot allocate a resource in VRAM; migrations are expensive because they suspend the GPU).
- **Status:** source-checked.
- **Pitfall:** Migrations are expensive. Demoted allocations under load are a memory-pressure signal for the GPU.

## Windows Internals and driver references

- **WDDM architecture:** the kernel-mode display driver and the graphics kernel (dxgkrnl) schedule GPU work and manage VRAM. [Windows Vista and later display driver model architecture](https://learn.microsoft.com/windows-hardware/drivers/display/windows-vista-and-later-display-driver-model-architecture) (content not checked on this pass).
- **GPUView:** visualizes GPU queues and memory charts per adapter and process. The [memory charts page](https://learn.microsoft.com/windows-hardware/drivers/display/memory-charts) describes these charts.
- **Sysinternals Process Explorer:** [Process Explorer](https://learn.microsoft.com/en-us/sysinternals/downloads/process-explorer) is listed here for process-level context. Its GPU columns were not checked on this pass.
