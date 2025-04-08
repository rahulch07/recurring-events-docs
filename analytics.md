SCHEMA
```
CREATE TABLE recurring_event_series_analytics (
  id UUID PRIMARY KEY,
  organization_id UUID REFERENCES organizations(id) NOT NULL,
  base_recurring_event_id UUID REFERENCES events(id) NOT NULL,
  series_name VARCHAR(255) NOT NULL,
  series_start_date DATE NOT NULL,
  series_end_date DATE,
  analysis_period_start DATE NOT NULL,
  analysis_period_end DATE NOT NULL,
  instances_count INTEGER NOT NULL DEFAULT 0,
  
  -- Attendance Metrics
  total_unique_registrants INTEGER NOT NULL DEFAULT 0,
  avg_registrants_per_instance DECIMAL(7,2) NOT NULL DEFAULT 0,
  total_unique_attendees INTEGER NOT NULL DEFAULT 0,
  avg_attendees_per_instance DECIMAL(7,2) NOT NULL DEFAULT 0,
  attendance_rate DECIMAL(5,2) NOT NULL DEFAULT 0,  -- % of registrants who attended
  series_retention_rate DECIMAL(5,2) NOT NULL DEFAULT 0,  -- % who returned for multiple instances
  instance_to_instance_retention DECIMAL(5,2) NOT NULL DEFAULT 0,  -- avg % returning from previous instance
  
  -- Volunteer Metrics
  total_unique_volunteers INTEGER NOT NULL DEFAULT 0,
  avg_volunteers_per_instance DECIMAL(7,2) NOT NULL DEFAULT 0,
  volunteer_fulfillment_rate DECIMAL(5,2) NOT NULL DEFAULT 0,  -- % of volunteer positions filled
  volunteer_return_rate DECIMAL(5,2) NOT NULL DEFAULT 0,  -- % of volunteers who returned
  
  -- Action Item Metrics
  total_action_items_created INTEGER NOT NULL DEFAULT 0,
  action_items_completion_rate DECIMAL(5,2) NOT NULL DEFAULT 0,
  avg_action_item_completion_time DECIMAL(7,2) NOT NULL DEFAULT 0,  -- hours
  
  -- Growth & Trend Metrics
  registrant_growth_rate DECIMAL(5,2) NOT NULL DEFAULT 0,  -- % change over series
  attendee_growth_rate DECIMAL(5,2) NOT NULL DEFAULT 0,  -- % change over series
  
  -- Time Efficiency Metrics
  avg_setup_time_minutes INTEGER NOT NULL DEFAULT 0,
  avg_cleanup_time_minutes INTEGER NOT NULL DEFAULT 0,
  
  -- Engagement Metrics
  avg_feedback_score DECIMAL(3,2) NOT NULL DEFAULT 0,  -- on scale of 1-5
  feedback_submission_rate DECIMAL(5,2) NOT NULL DEFAULT 0,  -- % of attendees giving feedback
  
  -- Financial Metrics (if applicable)
  total_revenue DECIMAL(12,2) NOT NULL DEFAULT 0,
  total_costs DECIMAL(12,2) NOT NULL DEFAULT 0,
  avg_cost_per_attendee DECIMAL(8,2) NOT NULL DEFAULT 0,
  
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_rec_analytics_base_event ON recurring_event_series_analytics(base_recurring_event_id);
CREATE INDEX idx_rec_analytics_period ON recurring_event_series_analytics(analysis_period_start, analysis_period_end);
CREATE INDEX idx_rec_analytics_org ON recurring_event_series_analytics(organization_id);
```
