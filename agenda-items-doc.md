
SCHEMA
```
-- Agenda Categories
CREATE TABLE agenda_categories (
  id UUID PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  organization_id UUID REFERENCES organizations(id) NOT NULL,
  created_by UUID REFERENCES users(id) NOT NULL,
  updated_by UUID REFERENCES users(id),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Agenda Items
CREATE TABLE agenda_items (
  id UUID PRIMARY KEY,
  title VARCHAR(200) NOT NULL,
  description TEXT,
  event_id UUID REFERENCES events(id) NOT NULL,
  duration VARCHAR(50) NOT NULL,  -- Could be "30 min", "1hr 15min", etc.
  sequence INT NOT NULL,  -- For ordering items within an event
  item_type VARCHAR(50),  -- Type of agenda item (presentation, break, etc.)
  organization_id UUID REFERENCES organizations(id) NOT NULL,
  created_by UUID REFERENCES users(id) NOT NULL,
  updated_by UUID REFERENCES users(id),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  
  -- For recurring handling
  is_recurring BOOLEAN DEFAULT FALSE,
  series_start_date DATE,
  series_end_date DATE
);

-- Agenda Item Categories (junction table)
CREATE TABLE agenda_item_categories (
  id UUID PRIMARY KEY,
  agenda_item_id UUID REFERENCES agenda_items(id) ON DELETE CASCADE,
  category_id UUID REFERENCES agenda_categories(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Agenda Item Participants
CREATE TABLE agenda_item_participants (
  id UUID PRIMARY KEY,
  agenda_item_id UUID REFERENCES agenda_items(id) ON DELETE CASCADE,
  user_id UUID REFERENCES users(id),
  role VARCHAR(50),  -- "presenter", "facilitator", etc.
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Agenda Item Attachments
CREATE TABLE agenda_item_attachments (
  id UUID PRIMARY KEY,
  agenda_item_id UUID REFERENCES agenda_items(id) ON DELETE CASCADE,
  file_url TEXT NOT NULL,
  file_name VARCHAR(255),
  file_type VARCHAR(50),
  created_by UUID REFERENCES users(id),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Agenda Item Notes
CREATE TABLE agenda_item_notes (
  id UUID PRIMARY KEY,
  agenda_item_id UUID REFERENCES agenda_items(id) ON DELETE CASCADE,
  content TEXT NOT NULL,
  created_by UUID REFERENCES users(id) NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Agenda Item URLs
CREATE TABLE agenda_item_urls (
  id UUID PRIMARY KEY,
  agenda_item_id UUID REFERENCES agenda_items(id) ON DELETE CASCADE,
  url TEXT NOT NULL,
  description VARCHAR(255),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- For recurring events - exceptions
CREATE TABLE agenda_item_exceptions (
  id UUID PRIMARY KEY,
  agenda_item_id UUID REFERENCES agenda_items(id) ON DELETE CASCADE,
  exception_date DATE NOT NULL,
  reason TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_agenda_items_event ON agenda_items(event_id);
CREATE INDEX idx_agenda_items_recurring ON agenda_items(is_recurring, series_start_date, series_end_date);
CREATE INDEX idx_agenda_item_exceptions_date ON agenda_item_exceptions(agenda_item_id, exception_date);
```
