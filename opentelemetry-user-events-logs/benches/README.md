# User Events Logs Benchmark

This benchmark suite compares the performance of user events logging with and without an active listener.

## Overview

The benchmark measures the performance impact of enabling the user events listener by comparing two states:
- **Disabled**: User events are created but no listener is active (baseline performance)
- **Enabled**: User events are created with an active listener (actual tracing overhead)

## Requirements

- **Linux Kernel**: 6.4+ with user_events support enabled
- **Permissions**: Root privileges or CAP_SYS_ADMIN capability
- **Filesystem**: tracefs mounted at `/sys/kernel/tracing` (preferred) or debugfs at `/sys/kernel/debug/tracing`

## Running the Benchmark

```bash
# Run the benchmark (requires root privileges)
sudo -E ~/.cargo/bin/cargo bench --bench logs --all-features

# Run with specific criterion options
sudo -E ~/.cargo/bin/cargo bench --bench logs --all-features -- --sample-size 1000
```

## Expected Results

Typical performance results on modern hardware:

| Test                        | Average time | Notes |
|-----------------------------|--------------|-------|
| user_events_logs/disabled   | ~19 ns       | Baseline (no listener) |
| user_events_logs/enabled    | ~530 ns      | With active listener |

The ~25x performance difference demonstrates the efficiency of the user_events subsystem when listeners are not active, and the overhead when tracing is actually enabled.

## Graceful Degradation

The benchmark automatically detects system capabilities:
- **No user_events support**: Benchmark is skipped with informative message
- **Insufficient permissions**: Clear error message with guidance
- **Missing tracefs/debugfs**: Automatic fallback between filesystem types

## Technical Details

- Uses `criterion` for reliable, statistical performance measurement
- Implements RAII pattern for automatic listener enable/disable
- Supports both tracefs (preferred) and debugfs (fallback) paths
- Provides detailed error messages for troubleshooting

## Troubleshooting

### "User events not supported on this system"
- Ensure kernel version is 6.4+
- Check if user_events is enabled: `cat /proc/config.gz | gunzip | grep USER_EVENTS`
- Verify tracefs is mounted: `mount | grep tracefs`

### "Insufficient permissions"
- Run with `sudo` or ensure CAP_SYS_ADMIN capability
- Check filesystem permissions on `/sys/kernel/tracing`

### "User events subsystem not available"
- Verify tracefs or debugfs is mounted
- Check if user_events module is loaded
