# Forge Vesting Factory

A single deployment that manages many vesting schedules.

## Purpose

- Create multiple vesting schedules without deploying one contract per beneficiary.
- Reduce deployment and operational overhead for team/investor token distributions.

## Core Functions

- `create_schedule(token, beneficiary, admin, total_amount, cliff_seconds, duration_seconds) -> schedule_id`
- `claim(schedule_id)`
- `cancel(schedule_id)`
- `get_status(schedule_id)`
- `get_schedule_count()`

## Notes

- Each schedule has its own token, beneficiary, admin, amount, cliff, and duration.
- On creation, tokens are transferred from admin into the factory contract.
- On cancel, vested amounts remain claimable while unvested amounts return to admin.
