### Knob
# Entities
# --------
# Tx:       transaction with id, read_set, write_set, and an execute(state) -> diff | abort
# State:    key/value state; apply(diff) mutates state
# Diff:     {key -> new_value} (plus optional read_versions for validation)
# Telemetry: cpu_util(), slot_latency(), mem_pressure(), spill_ratio()

# Parameters / "knobs"
# --------------------
# s0            : initial window size (e.g., 256)
# s_min,s_max   : min/max window
# tau_low, tau_high : spill ratio thresholds (e.g., 0.05, 0.25)
# alpha, beta   : shrink/grow factors (e.g., 0.5, 1.25)
# max_retries   : cap on requeue attempts (e.g., 3)
# order_cmp     : canonical comparator (e.g., by tx_id or (slot,tx_index,hash))

# Optional knobs
#  - window_mode: {fixed, adaptive}
#  - hot_key_backoff: true/false
#  - fairness: aging weight to avoid starvation
#  - rw_validation: enable read-version validation at commit
