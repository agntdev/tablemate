# Restaurant Booking Bot — Bot specification

**Archetype:** booking

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot for restaurant reservations that shows real-time available slots, prevents double-booking, issues booking codes, sends reminders, and provides owner management tools for bookings and capacity tracking.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- restaurant guests
- restaurant owner/staff

## Success criteria

- Real-time availability display prevents overbooking
- Guests receive booking codes and reminders
- Owner sees live capacity and manages bookings

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main booking menu
- **Make Reservation** (button, actor: user, callback: booking:start) — Initiates guest booking flow
  - inputs: party size, date/time
  - outputs: booking confirmation with code
- **Reschedule** (button, actor: user, callback: booking:reschedule) — Reschedule existing booking
- **Cancel Booking** (button, actor: user, callback: booking:cancel) — Cancel booking with confirmation prompt
- **View Bookings** (button, actor: owner, callback: owner:dashboard) — Owner dashboard for managing bookings

## Flows

### Guest Booking Flow
_Trigger:_ /start

1. Party size selection
2. Date selection within booking window
3. Time slot selection based on availability
4. Optional contact info collection
5. Booking confirmation with code
6. Inline reschedule/cancel buttons

_Data touched:_ bookings, tables

### Reschedule Flow
_Trigger:_ booking:reschedule

1. Show available slots for current party size
2. Select new slot with availability check
3. Update booking with new slot
4. Send confirmation

_Data touched:_ bookings, tables

### Owner Dashboard Flow
_Trigger:_ owner:dashboard

1. Show upcoming bookings list
2. Display today's available seats
3. No-show marking interface
4. Booking edit capabilities

_Data touched:_ bookings, tables

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **restaurant_settings** _(retention: persistent)_ — Restaurant configuration parameters
  - fields: opening_hours, slot_duration, booking_window, reminder_offset, timezone
- **tables** _(retention: persistent)_ — Table inventory with capacities
  - fields: table_id, capacity, combined_tables_allowed
- **bookings** _(retention: persistent)_ — Reservation records
  - fields: booking_code, party_size, date_time, assigned_tables, status, reminder_scheduled, guest_name, guest_phone
- **guests** _(retention: persistent)_ — Minimal guest contact info
  - fields: guest_name, guest_phone

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure tables and capacities
- Set opening hours and slot duration
- View/edit bookings
- Mark no-shows
- Adjust booking window and reminders

## Notifications

- Booking confirmation to guest
- Pre-visit reminder to guest
- New booking notification to owner
- No-show alerts
- Capacity alerts

## Permissions & privacy

- Guest contact info stored privately and only visible to owner
- Booking data encrypted at rest
- Owner-only access to management features

## Edge cases

- Party size requiring table combinations
- Last-minute cancellations before slot
- Time zone changes
- Concurrent booking attempts

## Required tests

- End-to-end booking flow with availability checks
- Double-booking prevention test
- Reminder delivery verification
- Owner dashboard data accuracy

## Assumptions

- Default 30-day booking window
- 90-minute slot duration
- 15-minute time granularity
- Combining tables allowed by default
