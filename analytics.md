# Generating Meaningful Analytics form Data

Insight 1: Volunteer Engagement and Retention

Understanding how consistently volunteers stay engaged is crucial for organizational success.

Volunteer retention across events

- returning_volunteers
- retention_percentage
```
-- Find retention rate of volunteers across multiple events
SELECT 
    COUNT(DISTINCT v1.user_id) AS returning_volunteers,
    (COUNT(DISTINCT v1.user_id) * 100.0 / NULLIF(COUNT(DISTINCT v.user_id), 0)) AS retention_percentage
FROM volunteers v
LEFT JOIN volunteers v1 ON v.user_id = v1.user_id 
    AND v1.created_at > v.created_at
    AND v1.event_id != v.event_id
GROUP BY v.event_id
ORDER BY retention_percentage DESC;
```
Most active volunteers

- user_id
- volunteer_name
- total_hours
- events_participated
```

-- Identify most active volunteers (by hours)
SELECT 
    u.id AS user_id,
    CONCAT(u.first_name, ' ', u.last_name) AS volunteer_name,
    SUM(ht.hours) AS total_hours,
    COUNT(DISTINCT v.event_id) AS events_participated
FROM users u
JOIN volunteers v ON u.id = v.user_id
JOIN hourly_tracking ht ON v.id = ht.volunteer_id
GROUP BY u.id, volunteer_name
ORDER BY total_hours DESC
LIMIT 20;
```

Insight 2: Action Item Completion Analysis

Measuring how effectively action items are being completed and identifying bottlenecks.

Completion rates by category

- category_name
- total_items
- completed_items
- completion_rate
- avg_days_to_complete
```
-- Action item completion rates by category
SELECT 
    aic.name AS category_name,
    COUNT(ai.id) AS total_items,
    SUM(CASE WHEN ai.is_completed THEN 1 ELSE 0 END) AS completed_items,
    (SUM(CASE WHEN ai.is_completed THEN 1 ELSE 0 END) * 100.0 / COUNT(ai.id)) AS completion_rate,
    AVG(COALESCE(aic.completion_days, 0)) AS avg_days_to_complete
FROM action_items ai
JOIN action_item_categories aic ON ai.action_item_category_id = aic.id
GROUP BY aic.name
ORDER BY completion_rate DESC;
```

Overdue action items

- id
- category
- assignment_date
- due_date
- days_overdue
```
-- Identify action items that are overdue
SELECT 
    ai.id,
    aic.name AS category,
    ai.assignment_date,
    (ai.assignment_date + (ai.Due_days * INTERVAL '1 day')) AS due_date,
    CURRENT_DATE - (ai.assignment_date + (ai.Due_days * INTERVAL '1 day')) AS days_overdue
FROM action_items ai
JOIN action_item_categories aic ON ai.action_item_category_id = aic.id
WHERE ai.is_completed = FALSE 
AND (ai.assignment_date + (ai.Due_days * INTERVAL '1 day')) < CURRENT_DATE
ORDER BY days_overdue DESC;
```

Insight 3: Event Attendance Patterns

Understanding attendance patterns helps optimize event planning and volunteer allocation.

Attendance rate calculation

- event_id
- event_name
- registered_attendees
- checked_in_attendees
- attendance_rate
```
-- Calculate attendance rate for events
SELECT 
    e.id AS event_id,
    e.name AS event_name,
    COUNT(ea.id) AS registered_attendees,
    COUNT(c.id) AS checked_in_attendees,
    (COUNT(c.id) * 100.0 / NULLIF(COUNT(ea.id), 0)) AS attendance_rate
FROM events e
LEFT JOIN event_attendees ea ON e.id = ea.base_event_id
LEFT JOIN checkIn c ON ea.id = c.attendee_id
WHERE ea.is_registered = TRUE
GROUP BY e.id, e.name
ORDER BY attendance_rate DESC;
```

Cancellation patterns

- month
- cancellations
- cancellation_rate
```
-- Analyze cancellation patterns
SELECT 
    TO_CHAR(DATE_TRUNC('month', ac.cancelled_date), 'YYYY-MM') AS month,
    COUNT(ac.id) AS cancellations,
    COUNT(ac.id) * 100.0 / NULLIF(COUNT(ea.id), 0) AS cancellation_rate
FROM event_attendees ea
LEFT JOIN attendee_cancellations ac ON ea.id = ac.attendee_id
GROUP BY month
ORDER BY month;
```

Insight 4: Volunteer Group Effectiveness

Analyzing how volunteer groups perform compared to individual volunteers.

Group vs individual task completion

- assignee_type
- total_tasks
- completed_tasks
- completion_rate
- avg_days_to_complete
```
-- Compare task completion between groups and individuals
SELECT 
    ai.assignee_type,
    COUNT(ai.id) AS total_tasks,
    SUM(CASE WHEN ai.is_completed THEN 1 ELSE 0 END) AS completed_tasks,
    (SUM(CASE WHEN ai.is_completed THEN 1 ELSE 0 END) * 100.0 / COUNT(ai.id)) AS completion_rate,
    AVG(ai.Completion_days) AS avg_days_to_complete
FROM action_items ai
WHERE ai.assignee_type IN ('Volunteer', 'VolunteerGroup')
GROUP BY ai.assignee_type;
```

Group utilization analysis

- group_id
- group_name
- member_count
- max_capacity
- utilization_rate
```
-- Analyze volunteer group utilization
SELECT 
    vg.id AS group_id,
    vg.name AS group_name,
    COUNT(vgm.volunteer_id) AS member_count,
    vg.maxVolunteerCount AS max_capacity,
    (COUNT(vgm.volunteer_id) * 100.0 / NULLIF(vg.maxVolunteerCount, 0)) AS utilization_rate
FROM volunteer_groups vg
LEFT JOIN volunteer_group_memberships vgm ON vg.id = vgm.group_id
WHERE vgm.status = 'approved'
GROUP BY vg.id, vg.name, vg.maxVolunteerCount
ORDER BY utilization_rate DESC;
```

Insight 5: Volunteer Hours Contribution Analysis

Tracking volunteer hours over time helps understand engagement patterns.

Monthly volunteer hours trend

- month
- total_hours
- active_volunteers
- avg_hours_per_volunteer
```
-- Monthly volunteer hours trend
SELECT 
    TO_CHAR(DATE_TRUNC('month', ht.date), 'YYYY-MM') AS month,
    SUM(ht.hours) AS total_hours,
    COUNT(DISTINCT ht.volunteer_id) AS active_volunteers,
    SUM(ht.hours) / COUNT(DISTINCT ht.volunteer_id) AS avg_hours_per_volunteer
FROM hourly_tracking ht
GROUP BY month
ORDER BY month;
```

Hours by volunteer type

- volunteer_type
- volunteer_count
- total_hours
- avg_hours_per_volunteer
```

-- Analyze hours by volunteer type
SELECT 
    v.volunteer_type,
    COUNT(DISTINCT v.id) AS volunteer_count,
    SUM(ht.hours) AS total_hours,
    AVG(ht.hours) AS avg_hours_per_volunteer
FROM volunteers v
JOIN hourly_tracking ht ON v.id = ht.volunteer_id
GROUP BY v.volunteer_type;
```

Insight 6: Recurring vs. One-time Event Performance

Compare metrics between recurring and one-time events. 

Attendance patterns comparison

- event_type
- event_count
- total_registrations
- avg_registrations_per_event
- total_check_ins
- attendance_rate
```
-- Compare attendance patterns between recurring and one-time events
SELECT 
    e.event_type, -- Assuming events table has event_type column
    COUNT(DISTINCT e.id) AS event_count,
    COUNT(ea.id) AS total_registrations,
    COUNT(ea.id) / COUNT(DISTINCT e.id) AS avg_registrations_per_event,
    COUNT(c.id) AS total_check_ins,
    (COUNT(c.id) * 100.0 / NULLIF(COUNT(ea.id), 0)) AS attendance_rate
FROM events e
LEFT JOIN event_attendees ea ON e.id = ea.base_event_id
LEFT JOIN checkIn c ON ea.id = c.attendee_id
GROUP BY e.event_type;
```

Volunteer recurring participation

- user_id
- total_volunteer_signups
- recurring_signups
- recurring_percentage
```
-- Analyze volunteer satisfaction through recurring participation
SELECT 
    u.id AS user_id,
    COUNT(DISTINCT v.id) AS total_volunteer_signups,
    SUM(CASE WHEN v.volunteer_type = 'recurring' THEN 1 ELSE 0 END) AS recurring_signups,
    (SUM(CASE WHEN v.volunteer_type = 'recurring' THEN 1 ELSE 0 END) * 100.0 / COUNT(DISTINCT v.id)) AS recurring_percentage
FROM users u
JOIN volunteers v ON u.id = v.user_id
GROUP BY u.id
HAVING COUNT(DISTINCT v.id) > 1
ORDER BY recurring_percentage DESC;
```

Insight 7: Action Item Effectiveness by Assignee Type

Understand which types of assignees complete tasks most effectively.

Completion metrics by assignee

- assignee_type
- total_items
- completed_items
- completion_rate
- avg_days_to_complete
- avg_allotted_hours
```
-- Compare action item completion metrics by assignee type
SELECT 
    ai.assignee_type,
    COUNT(ai.id) AS total_items,
    SUM(CASE WHEN ai.is_completed THEN 1 ELSE 0 END) AS completed_items,
    (SUM(CASE WHEN ai.is_completed THEN 1 ELSE 0 END) * 100.0 / COUNT(ai.id)) AS completion_rate,
    AVG(EXTRACT(EPOCH FROM (aic.updated_at - ai.assignment_date))/86400) AS avg_days_to_complete,
    AVG(ai.allotted_hours) AS avg_allotted_hours
FROM action_items ai
JOIN action_item_completions aic ON ai.id = aic.action_item_id
GROUP BY ai.assignee_type
ORDER BY completion_rate DESC;
```

Insight 8: Volunteer Exception Pattern Analysis

Understanding when and why volunteers miss events helps improve scheduling.

Exception patterns by day

- day_of_week
- exception_count
- percentage
```
-- Analyze exception patterns by day of week
SELECT 
    EXTRACT(DOW FROM exception_date) AS day_of_week,
    COUNT(*) AS exception_count,
    COUNT(*) * 100.0 / (SELECT COUNT(*) FROM volunteer_exceptions) AS percentage
FROM volunteer_exceptions
GROUP BY day_of_week
ORDER BY exception_count DESC;
```

Common exception reasons

- reason
- frequency
- percentage
```
-- Find most common exception reasons
SELECT 
    reason,
    COUNT(*) AS frequency,
    COUNT(*) * 100.0 / (SELECT COUNT(*) FROM volunteer_exceptions WHERE reason IS NOT NULL) AS percentage
FROM volunteer_exceptions
WHERE reason IS NOT NULL
GROUP BY reason
ORDER BY frequency DESC
LIMIT 10;
```

Insight 9: Volunteer-to-Attendee Ratio Analysis

Optimize staffing by analyzing the relationship between volunteers and attendees.

Ratio analysis by event

- event_id
- event_name
- volunteer_count
- attendee_count
- attendee_per_volunteer_ratio
```
-- Calculate volunteer-to-attendee ratios by event
SELECT 
    e.id AS event_id,
    e.name AS event_name,
    COUNT(DISTINCT v.id) AS volunteer_count,
    COUNT(DISTINCT ea.id) AS attendee_count,
    COUNT(DISTINCT ea.id)::FLOAT / NULLIF(COUNT(DISTINCT v.id), 0) AS attendee_per_volunteer_ratio
FROM events e
LEFT JOIN volunteers v ON e.id = v.event_id
LEFT JOIN event_attendees ea ON e.id = ea.base_event_id AND ea.is_registered = TRUE
GROUP BY e.id, e.name
HAVING COUNT(DISTINCT v.id) > 0
ORDER BY attendee_per_volunteer_ratio DESC;
```

Insight 10: Comprehensive Event Success Dashboard

Create a holistic view of event success metrics.

Comprehensive event metrics

- event_id
- event_name
- registered_attendees
- checked_in_attendees
- attendance_rate
- volunteer_count
- total_volunteer_hours
- action_items_count
  completed_action_items
- action_completion_rate


```
-- Comprehensive event performance metrics
SELECT 
    e.id AS event_id,
    e.name AS event_name,
    COUNT(DISTINCT ea.id) AS registered_attendees,
    COUNT(DISTINCT c.id) AS checked_in_attendees,
    (COUNT(DISTINCT c.id) * 100.0 / NULLIF(COUNT(DISTINCT ea.id), 0)) AS attendance_rate,
    COUNT(DISTINCT v.id) AS volunteer_count,
    SUM(ht.hours) AS total_volunteer_hours,
    COUNT(DISTINCT ai.id) AS action_items_count,
    SUM(CASE WHEN ai.is_completed THEN 1 ELSE 0 END) AS completed_action_items,
    (SUM(CASE WHEN ai.is_completed THEN 1 ELSE 0 END) * 100.0 / NULLIF(COUNT(DISTINCT ai.id), 0)) AS action_completion_rate
FROM events e
LEFT JOIN event_attendees ea ON e.id = ea.base_event_id AND ea.is_registered = TRUE
LEFT JOIN checkIn c ON ea.id = c.attendee_id
LEFT JOIN volunteers v ON e.id = v.event_id
LEFT JOIN hourly_tracking ht ON v.id = ht.volunteer_id AND e.id = ht.event_id
LEFT JOIN action_items ai ON e.id = ai.event_id
GROUP BY e.id, e.name
ORDER BY attendance_rate DESC, action_completion_rate DESC;
```
