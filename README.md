# EventHub — Backend API

A Django REST Framework API for a simplified event ticketing platform. Users can browse events, reserve seats, and cancel reservations.

---

## How to Run

### 1. Clone and install dependencies

```bash
git clone <your-repo-url>
cd eventhub
pip install -r requirements.txt
```

### 2. Run migrations

```bash
python manage.py makemigrations
python manage.py migrate
```

### 3. Start the server

```bash
python manage.py runserver
```

The API is available at `http://127.0.0.1:8000/api/`

---

## Endpoints

### Events

| Method | URL | Description |
|--------|-----|-------------|
| GET | `/api/events/` | List all events. Filter by `?status=upcoming` or `?venue=bangalore` |
| POST | `/api/events/` | Create a new event |
| GET | `/api/events/{id}/` | Retrieve a single event |
| PUT | `/api/events/{id}/` | Update an event |
| PATCH | `/api/events/{id}/` | Partially update an event |
| DELETE | `/api/events/{id}/` | Delete an event |

### Reservations

| Method | URL | Description |
|--------|-----|-------------|
| GET | `/api/reservations/` | List all reservations. Filter by `?event_id=1` |
| POST | `/api/reservations/` | Create a reservation (deducts seats from event) |
| GET | `/api/reservations/{id}/` | Retrieve a single reservation |
| PUT | `/api/reservations/{id}/` | Update a reservation |
| PATCH | `/api/reservations/{id}/` | Partially update a reservation |
| DELETE | `/api/reservations/{id}/` | Delete a reservation |
| POST | `/api/reservations/{id}/cancel/` | Cancel a reservation (restores seats to event) |

---

## Sample Requests

### Create an event
```json
POST /api/events/
{
  "title": "PyCon India 2025",
  "venue": "NIMHANS Convention Centre, Bangalore",
  "date": "2025-09-20",
  "total_seats": 500,
  "available_seats": 500,
  "status": "upcoming"
}
```

### Reserve seats
```json
POST /api/reservations/
{
  "event": 1,
  "attendee_name": "Priya Sharma",
  "attendee_email": "priya@example.com",
  "seats_reserved": 2
}
```

### Cancel a reservation
```
POST /api/reservations/1/cancel/
```
No request body required. Returns the updated reservation with `"status": "cancelled"`.

---

## Design Decision: Atomic seat deduction inside the serializer `create()` method

Seat deduction and reservation creation are both handled inside `ReservationSerializer.create()`. This keeps both writes in one place — the seat count is decremented on the `Event` and the `Reservation` row is inserted in the same method call, making the logic easy to read and trace.

The `validate()` method runs a guard check before `create()` is called, so a request for more seats than available is rejected with a `400` before any database write happens. This prevents the most common overbooking scenario.

In a high-traffic production system, these two writes would be wrapped in `transaction.atomic()` combined with `select_for_update()` on the Event row, to prevent a race condition where two simultaneous requests both pass the availability check and both deduct seats. That pattern was kept out of scope for this assignment per the brief.

---

## Middleware

`RequestLoggingMiddleware` (in `events/middleware.py`) logs every request in the format:

```
METHOD /path - STATUS_CODE - duration_in_seconds
```

Example terminal output:
```
INFO POST /api/reservations/ - 201 - 0.04s
INFO GET /api/events/ - 200 - 0.01s
```

## Postman Screenshots
Go to screenshots folder and see all the api calls 