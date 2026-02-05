# Operator Dashboard

Read-only Flutter web app for operators: financial and operational summary. Responsive (desktop sidebar, mobile bottom nav).

## Purpose

- **Not** a live validator monitor.
- **Is** a viewer: trips, shifts, vehicles, earnings. Backend is source of truth; dashboard never mutates data.

## Phase 1 (implemented)

- Login (operator PIN → `validate_operator_pin`, then resolve `operators.id` for scoping)
- Dashboard home: today’s total trips, gross fare, platform fee, operator earnings, active vehicles/validators
- Vehicles list and vehicle detail (trips)
- Shifts list and shift detail (trips reconciliation)
- Responsive layout: sidebar on desktop, bottom nav on mobile

## Phase 2 (later)

- CSV export, date range filter
- Live validator status, alerts, merchant reconciliation, LGU role, audit logs

## Run

1. Apply backend migration **035_operator_dashboard_rpcs.sql** (creates RPCs scoped by `operator_id`).
2. Configure Supabase URL and anon key (env or `--dart-define`):
   ```bash
   flutter run -d chrome --dart-define=SUPABASE_URL=your_url --dart-define=SUPABASE_ANON_KEY=your_key
   ```
   Or set in `lib/main.dart` for local dev.
3. From project root: `cd operator_dashboard && flutter pub get && flutter run -d chrome`.

## Backend RPCs (read-only)

- `get_operator_dashboard_summary(p_operator_id UUID)` — today’s summary
- `get_operator_vehicles(p_operator_id UUID)` — vehicles with today’s trip/earnings
- `get_operator_shifts(p_operator_id UUID, p_limit INT)` — shifts for operator’s vehicles
- `get_operator_shift_by_id(p_operator_id UUID, p_shift_id UUID)` — one shift if owned
- `get_shift_trips(p_shift_id UUID)` — trips for a shift
- `get_operator_vehicle_by_id(p_operator_id UUID, p_vehicle_id UUID)` — one vehicle if owned
- `get_operator_vehicle_trips(p_operator_id UUID, p_vehicle_id UUID, p_date DATE)` — trips for vehicle

All data filtered by `vehicles.operator_id` or shift → vehicle → operator. Uses `shifts` and `trips` (completed only).

## Rules

- Dashboard never mutates data, never fixes balances, never retries trips.
- Validator devices are the only writers for trips; backend is the only accounting authority.
