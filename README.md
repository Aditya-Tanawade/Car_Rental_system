String sql = "SELECT " +
                    "i.interview_id, " +
                    "CASE " +
                    "  WHEN i.team_leader_id IS NOT NULL THEN " +
                    "    (SELECT ed.full_name FROM employee_details ed WHERE ed.employee_id = i.team_leader_id) " +
                    "  WHEN i.project_manager_id IS NOT NULL THEN " +
                    "    (SELECT ed.full_name FROM employee_details ed WHERE ed.employee_id = i.project_manager_id) " +
                    "  WHEN i.hr_id IS NOT NULL THEN " +
                    "    (SELECT ed.full_name FROM employee_details ed WHERE ed.employee_id = i.hr_id) " +
                    "END AS interviewer_name, " +
                    "CASE " +
                    "  WHEN i.team_leader_id IS NOT NULL THEN " +
                    "    (SELECT ed.role_key FROM employee_details ed WHERE ed.employee_id = i.team_leader_id) " +
                    "  WHEN i.project_manager_id IS NOT NULL THEN " +
                    "    (SELECT ed.role_key FROM employee_details ed WHERE ed.employee_id = i.project_manager_id) " +
                    "  WHEN i.hr_id IS NOT NULL THEN " +
                    "    (SELECT ed.role_key FROM employee_details ed WHERE ed.employee_id = i.hr_id) " +
                    "END AS interviewer_role, " +
                    "i.employee_email AS interviewer_email, " +
                    "jr.title AS job_title, " +
                    "TO_CHAR(i.interview_date, 'DD-MM-YYYY') AS interview_date, " +
                    "i.start_time, " +
                    "i.end_time, " +
                    "i.google_meet_link, " +
                    "i.summary, " +
                    "i.description, " +
                    "i.round_number " +
                    "FROM interviews i " +
                    "INNER JOIN job_requests jr ON i.job_request_id = jr.job_request_id " +
                    "WHERE i.candidate_id = ? " +
                    "AND i.status in('PENDING','INTERVIEW1_CLEARED','INTERVIEW2_CLEARED')  " +
                    "AND i.interview_date >= TRUNC(SYSDATE) " +
                    "AND i.google_meet_link IS NOT NULL " +
                    "ORDER BY i.interview_date ASC, i.start_time ASC";

