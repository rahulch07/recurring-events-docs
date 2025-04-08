Database Schema Extensions:

1. Venue Availability Rules Table
```
CREATE TABLE venue_availability_rules (
  id UUID PRIMARY KEY,
  venue_id UUID NOT NULL REFERENCES venues(id),
  day_of_week INTEGER[], -- [0,1,2,3,4,5,6] for days available
  start_time TIME NOT NULL, -- Operating hours start
  end_time TIME NOT NULL, -- Operating hours end
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```
This defines when venues are generally available (operating hours).
2. Venue Booking Instances Table
```
CREATE TABLE venue_booking_instances (
  id UUID PRIMARY KEY,
  venue_id UUID NOT NULL REFERENCES venues(id),
  event_id UUID NOT NULL REFERENCES events(id),
  booking_date DATE NOT NULL,
  start_time TIMESTAMPTZ NOT NULL,
  end_time TIMESTAMPTZ NOT NULL,
  is_exception BOOLEAN DEFAULT FALSE,
  base_recurring_event_id UUID REFERENCES events(id),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_venue_bookings_venue ON venue_booking_instances(venue_id);
CREATE INDEX idx_venue_bookings_date ON venue_booking_instances(booking_date);
CREATE INDEX idx_venue_bookings_time ON venue_booking_instances(start_time, end_time);
CREATE INDEX idx_venue_bookings_recurring ON venue_booking_instances(base_recurring_event_id);

CREATE TABLE venue_booking_instances_YYYY_MM PARTITION OF venue_booking_instances
FOR VALUES FROM ('YYYY-MM-01') TO ('YYYY-MM+1-01');
```
This table stores all venue booking instances, including those from recurring events.

3. Venue Booking Exceptions Table
```
CREATE TABLE venue_booking_exceptions (
  id UUID PRIMARY KEY,
  venue_id UUID NOT NULL REFERENCES venues(id),
  base_recurring_event_id UUID NOT NULL REFERENCES events(id),
  exception_date DATE NOT NULL,
  reason TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_booking_exceptions_venue ON venue_booking_exceptions(venue_id);
CREATE INDEX idx_booking_exceptions_event ON venue_booking_exceptions(base_recurring_event_id);
CREATE INDEX idx_booking_exceptions_date ON venue_booking_exceptions(exception_date);
This tracks canceled/modified instances in a recurring venue booking series.
```
