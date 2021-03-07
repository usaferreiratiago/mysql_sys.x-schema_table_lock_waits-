# mysql_sys.x-schema_table_lock_waits-

SELECT 
    `g`.`OBJECT_SCHEMA` AS `object_schema`,
    `g`.`OBJECT_NAME` AS `object_name`,
    `pt`.`THREAD_ID` AS `waiting_thread_id`,
    `pt`.`PROCESSLIST_ID` AS `waiting_pid`,
    `sys`.`ps_thread_account`(`p`.`OWNER_THREAD_ID`) AS `waiting_account`,
    `p`.`LOCK_TYPE` AS `waiting_lock_type`,
    `p`.`LOCK_DURATION` AS `waiting_lock_duration`,
    `pt`.`PROCESSLIST_INFO` AS `waiting_query`,
    `pt`.`PROCESSLIST_TIME` AS `waiting_query_secs`,
    `ps`.`ROWS_AFFECTED` AS `waiting_query_rows_affected`,
    `ps`.`ROWS_EXAMINED` AS `waiting_query_rows_examined`,
    `gt`.`THREAD_ID` AS `blocking_thread_id`,
    `gt`.`PROCESSLIST_ID` AS `blocking_pid`,
    `sys`.`ps_thread_account`(`g`.`OWNER_THREAD_ID`) AS `blocking_account`,
    `g`.`LOCK_TYPE` AS `blocking_lock_type`,
    `g`.`LOCK_DURATION` AS `blocking_lock_duration`,
    CONCAT('KILL QUERY ', `gt`.`PROCESSLIST_ID`) AS `sql_kill_blocking_query`,
    CONCAT('KILL ', `gt`.`PROCESSLIST_ID`) AS `sql_kill_blocking_connection`
FROM
    (((((`performance_schema`.`metadata_locks` `g`
    JOIN `performance_schema`.`metadata_locks` `p` ON (((`g`.`OBJECT_TYPE` = `p`.`OBJECT_TYPE`)
        AND (`g`.`OBJECT_SCHEMA` = `p`.`OBJECT_SCHEMA`)
        AND (`g`.`OBJECT_NAME` = `p`.`OBJECT_NAME`)
        AND (`g`.`LOCK_STATUS` = 'GRANTED')
        AND (`p`.`LOCK_STATUS` = 'PENDING'))))
    JOIN `performance_schema`.`threads` `gt` ON ((`g`.`OWNER_THREAD_ID` = `gt`.`THREAD_ID`)))
    JOIN `performance_schema`.`threads` `pt` ON ((`p`.`OWNER_THREAD_ID` = `pt`.`THREAD_ID`)))
    LEFT JOIN `performance_schema`.`events_statements_current` `gs` ON ((`g`.`OWNER_THREAD_ID` = `gs`.`THREAD_ID`)))
    LEFT JOIN `performance_schema`.`events_statements_current` `ps` ON ((`p`.`OWNER_THREAD_ID` = `ps`.`THREAD_ID`)))
WHERE
    (`g`.`OBJECT_TYPE` = 'TABLE')
