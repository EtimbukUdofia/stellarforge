# forge-vesting-factory

A single-deployment factory that manages multiple vesting schedules without requiring a separate contract per beneficiary.

## Usage

```rust
// Create a schedule — returns a schedule_id
let id = client.create_schedule(&token, &beneficiary, &admin, &total_amount, &cliff_seconds, &duration_seconds);

// Beneficiary claims unlocked tokens
client.claim(&id);

// Admin cancels early — vested tokens go to beneficiary, unvested return to admin
client.cancel(&id);

// Read current status (vested, claimed, claimable, cancelled)
let status = client.get_status(&id);
```

## Storage Strategy

All persistent data uses `env.storage().persistent()`. There is no instance storage.

| Key | Storage | TTL extended in |
| :--- | :--- | :--- |
| `DataKey::ScheduleCount` | persistent | `create_schedule` |
| `DataKey::Schedule(id)` | persistent | `create_schedule`, `claim`, `cancel` |
| `DataKey::Claimed(id)` | persistent | `claim`, `cancel` |
| `DataKey::VestedAtCancel(id)` | persistent | `cancel` |

### Why `ScheduleCount` must be persistent

`ScheduleCount` is the monotonically increasing counter that assigns IDs to new schedules. If it were stored in instance storage and that entry expired, the counter would silently reset to 0. The next `create_schedule` call would reuse ID 0, overwriting the existing schedule in persistent storage and permanently destroying a beneficiary's vesting data.

Storing `ScheduleCount` in persistent storage and bumping its TTL on every `create_schedule` call ensures the counter survives as long as the individual schedule entries do.

### TTL parameters

All `extend_ttl` calls use `(17280, 34560)` — a minimum of 17 280 ledgers (~1 day) and a target of 34 560 ledgers (~2 days). These match the values used across other StellarForge contracts. Integrators running long-lived schedules should call `create_schedule`, `claim`, or `cancel` at least once every ~30 days, or submit a dedicated TTL-bump transaction, to keep entries alive on mainnet.
