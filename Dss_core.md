### core scheduler
function DSS_Run(transactions, state):
    queue        := list(transactions)
    retry_count  := map<tx_id, int>(default=0)
    s            := s0
    window_mode  := "adaptive"   # can be "fixed"
    committed    := 0
    total_spills := 0

    while not queue.empty():
        batch := queue.take_first(s)

        # 1) Speculative execution (NO global mutation)
        specs := []
        parallel_for tx in batch:
            diff, status := SpecExecute(tx, state)
            specs.append({tx, diff, status})

        # 2) Deterministic order for commit
        specs.sort_by(lambda e: order_cmp(e.tx))

        # 3) Conflict-aware commit (and optional read validation)
        writes_seen  := set()          # committed write keys this window
        read_ver_seen:= map()          # optional: key->version if validating reads
        spilled_now  := 0
        committed_now:= 0
        next_queue   := []

        for each entry in specs:
            tx   := entry.tx
            diff := entry.diff
            ok   := (entry.status == "OK") and not Conflicts(tx, diff, writes_seen)

            if ok and RWValidate(diff, state):   # RWValidate can be a no-op if disabled
                Apply(state, diff)
                writes_seen |= Keys(diff)
                # optional: update read_ver_seen if tracking versions
                committed_now += 1
            else:
                if retry_count[tx.id] < max_retries:
                    retry_count[tx.id] += 1
                    next_queue.append(tx)
                    spilled_now += 1
                else:
                    MarkAbort(tx)    # exceeded retries; drop or log

        # 4) Adaptive window sizing (based on spill ratio + telemetry)
        sigma := spill_ratio(committed_now, spilled_now)     # spilled / (committed+spilled)
        s     := AdaptWindow(s, sigma, window_mode)

        # 5) Optional: prioritize requeues (e.g., age or backoff for hot keys)
        queue := Prioritize(next_queue) + queue

        committed    += committed_now
        total_spills += spilled_now

    return {state, committed, total_spills}

### Speculative Execution
function SpecExecute(tx, state):
    # Execute against a snapshot (copy-on-write or read-only view).
    # Return (diff, "OK") or (nil, "ABORT").
    try:
        diff := tx.execute(snapshot_of(state))
        if diff == nil: return (nil, "ABORT")
        return (diff, "OK")
    catch:
        return (nil, "ABORT")

  ### Conflict Detection
        function Conflicts(tx, diff, writes_seen):
    # Conservative rule: conflict if tx writes any key already written in this window
    # or (optional) if tx reads a key written earlier in this window.
    W := WriteKeys(diff)
    if Intersects(W, writes_seen): return true
    # Optional (stricter): R := tx.read_set; if Intersects(R, writes_seen): return true
    return false

   ### Adaptive Window Sizing
   function AdaptWindow(s, sigma, mode):
    if mode == "fixed": return s

    # Use dual-threshold control on spill ratio; combine with CPU/latency if desired.
    u  := cpu_util()        # 0..1
    L  := slot_latency()    # ms
    Lh := LAT_HIGH; Ll := LAT_LOW

    if sigma > tau_high or L > Lh:
        s_new := max(s_min, floor(alpha * s))
    elif sigma < tau_low and u < 0.85 and L < Ll:
        s_new := min(s_max, ceil(beta * s))
    else:
        s_new := s

    return s_new
    
