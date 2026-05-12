CREATE OR REPLACE PROCEDURE LOAD_GOLD_AGENTS
AS
    v_start_time TIMESTAMP := SYSTIMESTAMP;
    v_status      VARCHAR2(20);
    v_message     VARCHAR2(4000);
BEGIN
    -- Merge to handle upsert logic
    MERGE INTO GOLD_AGENTS_D tgt
    USING (
        SELECT 
            AGENT_ID,
            AGENT_NAME,
            SOURCE_SYSTEM
        FROM STAGE_AGENTS
    ) src
    ON (tgt.AGENT_ID = src.AGENT_ID)

    WHEN MATCHED THEN
        UPDATE SET
            tgt.AGENT_NAME = src.AGENT_NAME,
            tgt.UPDATE_DATE = SYSTIMESTAMP,
            tgt.SOURCE_SYSTEM = src.SOURCE_SYSTEM

    WHEN NOT MATCHED THEN
        INSERT (
            AGENT_ID,
            AGENT_NAME,
            LOAD_DATE,
            UPDATE_DATE,
            SOURCE_SYSTEM
        ) VALUES (
            src.AGENT_ID,
            src.AGENT_NAME,
            SYSTIMESTAMP,
            SYSTIMESTAMP,
            src.SOURCE_SYSTEM
        );

    -- Log success
    v_status := 'SUCCESS';
    v_message := 'Agents data loaded successfully.';

    INSERT INTO AUDIT_LOG (MODULE_NAME, RUN_TIME, STATUS, MESSAGE)
    VALUES ('LOAD_GOLD_AGENTS', v_start_time, v_status, v_message);

    COMMIT;

EXCEPTION
    WHEN OTHERS THEN
        v_status := 'FAILED';
        v_message := 'Error: ' || SQLERRM;

        -- Log the error
        BEGIN
            INSERT INTO AUDIT_LOG (MODULE_NAME, RUN_TIME, STATUS, MESSAGE)
            VALUES ('LOAD_GOLD_AGENTS', v_start_time, v_status, v_message);
            COMMIT;
        EXCEPTION
            WHEN OTHERS THEN
                NULL; -- suppress audit failure
        END;

        RAISE;
END;
/
