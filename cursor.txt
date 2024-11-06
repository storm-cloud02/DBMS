DECLARE
    -- Define a parameterized cursor to fetch records from N_Roll_Call
    CURSOR roll_call_cur IS
        SELECT Roll_no, Name, Class FROM N_Roll_Call;

    -- Define variables to hold the cursor data
    v_rollno N_Roll_Call.Roll_no%TYPE;
    v_name N_Roll_Call.Name%TYPE;
    v_class N_Roll_Call.Class%TYPE;

    -- Variable to check if record exists
    v_exists NUMBER;
BEGIN
    -- Open the cursor and loop through all records
    FOR rec IN roll_call_cur LOOP
        -- Check if the current record from N_Roll_Call exists in O_Roll_Call
        SELECT COUNT(*)
        INTO v_exists
        FROM O_Roll_Call
        WHERE Roll_no = rec.Roll_no;

        -- If the record doesn't exist, insert it into O_Roll_Call
        IF v_exists = 0 THEN
            INSERT INTO O_Roll_Call (Roll_no, Name, Class)
            VALUES (rec.Roll_no, rec.Name, rec.Class);
            DBMS_OUTPUT.PUT_LINE('Inserted: ' || rec.Name || ' into O_Roll_Call');
        ELSE
            DBMS_OUTPUT.PUT_LINE('Skipped: ' || rec.Name || ' (already exists in O_Roll_Call)');
        END IF;
    END LOOP;

    -- Commit the transaction
    COMMIT;
END;
/
