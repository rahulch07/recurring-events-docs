ADMIN

1.  Listing Action Items for a specific Event Instance:
```
-- For a specific event instance on a given date
SELECT ai.*
FROM action_items ai
WHERE 
  (ai.event_id = :base_event_id AND ai.is_recurring = FALSE AND ai.instance_date = :instance_date)
  OR
  (ai.event_id = :base_event_id AND ai.is_recurring = TRUE 
   AND :instance_date BETWEEN ai.series_start_date AND COALESCE(ai.series_end_date, '9999-12-31')
   AND NOT EXISTS (
     SELECT 1 FROM action_item_exceptions aie 
     WHERE aie.action_item_id = ai.id AND aie.exception_date = :instance_date
   ))
```
2. Finding Volunteers Available for an Event Instance:
```
-- For a specific event instance on a given date
SELECT v.*
FROM volunteers v
WHERE 
  (v.event_id = :base_event_id AND v.volunteer_type = 'single' AND v.instance_date = :instance_date)
  OR
  (v.event_id = :base_event_id AND v.volunteer_type = 'recurring' 
   AND :instance_date BETWEEN v.series_start_date AND COALESCE(v.series_end_date, '9999-12-31')
   AND NOT EXISTS (
     SELECT 1 FROM volunteer_exceptions ve 
     WHERE ve.volunteer_id = v.id AND ve.exception_date = :instance_date
   ))
```
3. Finding Groups Available for an Event Instance:
```
-- For a specific event instance on a given date
SELECT vg.*
FROM volunteer_groups vg
WHERE 
  (vg.event_id = :base_event_id AND vg.group_type = 'single' AND vg.instance_date = :instance_date)
  OR
  (vg.event_id = :base_event_id AND vg.group_type = 'recurring' 
   AND :instance_date BETWEEN vg.series_start_date AND COALESCE(vg.series_end_date, '9999-12-31')
   AND NOT EXISTS (
     SELECT 1 FROM group_exceptions ge 
     WHERE ge.group_id = vg.id AND ge.exception_date = :instance_date
   ))
```


USERS:
Fetch His/Her Action Items:
```
-- Query to list all action items for a specific user
SELECT ai.*, aic.category_name, evt.name as event_name, 
       CASE
           WHEN ai.is_recurring = TRUE AND ai_exceptions.exception_date IS NULL
              THEN 'Active Recurring'
           WHEN ai.is_recurring = TRUE AND ai_exceptions.exception_date IS NOT NULL
              THEN 'Completed Instance'
           WHEN ai.is_completed = TRUE 
              THEN 'Completed'
           WHEN ai.due_date < NOW() 
              THEN 'Overdue'
           ELSE 'Active'
       END as status
FROM action_items ai
LEFT JOIN action_item_categories aic ON ai.action_item_category_id = aic.id
LEFT JOIN events evt ON ai.event_id = evt.id
LEFT JOIN action_item_exceptions ai_exceptions ON ai.id = ai_exceptions.action_item_id
WHERE 
    -- Direct assignments to user
    (ai.assignee_type = 'User' AND ai.assignee_id = :user_id)
    
    -- Assignments to user as volunteer
    OR (ai.assignee_type = 'Volunteer' AND ai.assignee_id IN 
        (SELECT id FROM volunteers WHERE user_id = :user_id))
    
    -- Assignments to groups the user is part of
    OR (ai.assignee_type = 'VolunteerGroup' AND ai.assignee_id IN 
        (SELECT vg.id 
         FROM volunteer_groups vg
         JOIN volunteer_group_memberships vgm ON vg.id = vgm.group_id
         JOIN volunteers v ON vgm.volunteer_id = v.id
         WHERE v.user_id = :user_id
         AND (
             -- For single instance group membership
             (vgm.membership_type = 'single' AND vgm.instance_date = CURRENT_DATE)
             OR
             -- For recurring group membership
             (vgm.membership_type = 'recurring' 
              AND CURRENT_DATE BETWEEN vgm.series_start_date AND COALESCE(vgm.series_end_date, '9999-12-31')
              AND NOT EXISTS (
                SELECT 1 FROM membership_exceptions me
                WHERE me.membership_id = vgm.id AND me.exception_date = CURRENT_DATE
              ))
         )))

ORDER BY ai.due_date ASC;
```
