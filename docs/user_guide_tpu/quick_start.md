# Google TPU Quick Start

> **There is no TPU training quick start yet.** TPU support is at the platform layer only. A GRPO
> or PPO run cannot be launched on TPU from stock verl today — see the status note in the
> [User Guide](./README.md). This page covers what you can verify now.

## 1. Select the Platform

Always select TPU explicitly:

```bash
export VERL_PLATFORM=tpu
export PJRT_DEVICE=TPU
```

Auto-detection also works on a TPU host, because each registered platform is probed in turn and only
the TPU probe succeeds. But that outcome depends on registration order, which is not a stable
contract. `VERL_PLATFORM=tpu` is the supported way to select TPU.

## 2. Verify Platform Resolution

```bash
python3 -c '
from verl.plugin.platform.platform_manager import get_platform
p = get_platform()
print("device:   ", p.device_name)
print("vendor:   ", p.vendor_name)
print("backend:  ", p.communication_backend_name())
print("ray res:  ", p.ray_resource_name())
print("ipc:      ", p.is_ipc_supported())
print("colocate: ", p.supports_colocated_worker_groups())
'
```

Expected output:

```text
device:    tpu
vendor:    google
backend:   tpu_dist
ray res:   TPU
ipc:       False
colocate:  False
```

## 3. Verify Ray Resource Requests

The platform requests chips as a custom Ray resource named `TPU`, not as `num_gpus`:

```bash
python3 -c '
from verl.plugin.platform.platform_manager import get_platform
print(get_platform().ray_resource_options(4))
'
```

Expected output: `{'resources': {'TPU': 4}}`

Your Ray cluster must advertise a `TPU` resource for scheduling to succeed. On KubeRay this comes
from the TPU node pool's resource annotations.

## 4. Run the Plugin Test Suite

```bash
pytest tests/test_plugin_registration.py -k tpu -v
```

Expected: all TPU cases pass. These run on any host, with or without a TPU attached.

## Next Steps

The core-side hook call sites are still unmerged in verl. The TPU training engine will land in a
follow-up PR to this plugin.
