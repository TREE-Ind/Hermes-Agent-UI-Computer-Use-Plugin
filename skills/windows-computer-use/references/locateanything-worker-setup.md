# LocateAnything Worker Setup

LocateAnything is optional visual grounding. Basic mouse/keyboard/screenshot tools should work without it.

## Why use an external worker

Do not install CUDA PyTorch into the live Hermes environment unless the user explicitly wants that risk. Use an isolated worker venv so Hermes still runs if ML dependencies break.

## Install worker dependencies

From the plugin repo:

```powershell
.\scripts\install_locate_worker.ps1 -Python C:\Python312\python.exe -Cuda cu121
```

For CPU-only mode, omit `-Cuda` if the script supports that path.

## Configure Hermes/session environment

Set these environment variables, then restart Hermes:

```powershell
setx COMPUTER_USE_LOCATE_BACKEND external
setx COMPUTER_USE_LOCATE_PYTHON "$env:LOCALAPPDATA\hermes\windows-computer-use\.venv\Scripts\python.exe"
setx COMPUTER_USE_LOCATE_PERSISTENT true
```

Optional tuning:

```powershell
setx COMPUTER_USE_LOCATE_MAX_SIDE 640
setx COMPUTER_USE_LOCATE_MAX_NEW_TOKENS 32
setx COMPUTER_USE_LOCATE_GENERATION_MODE hybrid
setx COMPUTER_USE_LOCATE_DO_SAMPLE false
setx COMPUTER_USE_LOCATE_STRATEGY direct
setx COMPUTER_USE_LOCATE_PROMPT_STYLE direct
```

## Warm and test

Inside Hermes:

```text
computer_use_warm(backend="external")
computer_use_capture_screen()
computer_use_locate(description="the Start button", output_type="point", backend="external")
```

Expected behavior:

- First call may take time while the model loads.
- Persistent worker calls should be faster after warmup.
- If the worker fails, action tools should still remain available.

## Good defaults

For UI clicking:

```text
output_type="point"
strategy="direct"
prompt_style="direct"
max_side=640
max_new_tokens=32
generation_mode="hybrid"
do_sample=false
```

Use `output_type="box"` or refinement strategies only when you need region bounds or the direct point is unreliable.

## Failure handling

If locate fails:

1. Capture a fresh screenshot.
2. Use a clearer description with visible label + app context.
3. Try a smaller region screenshot.
4. Fall back to manual coordinates if safe.
5. Report that visual grounding is unavailable if the worker/dependencies are broken.
