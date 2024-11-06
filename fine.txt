CREATE OR REPLACE PROCEDURE CalculateFine (
    p_rollno IN NUMBER,
    p_bookname IN VARCHAR2
) AS
    v_date_of_issue DATE;
    v_status CHAR(1);
    v_days NUMBER;
    v_fine_amount NUMBER := 0;
BEGIN
    -- Fetch the details of the borrower
    SELECT Date_of_Issue, Status
    INTO v_date_of_issue, v_status
    FROM Borrower
    WHERE Roll_no = p_rollno AND Name_of_Book = p_bookname;  -- Adjusted for case sensitivity

    -- Check if the book is still issued
    IF v_status = 'I' THEN
        -- Calculate the number of days since the book was issued
        v_days := TRUNC(SYSDATE) - TRUNC(v_date_of_issue);

        -- Determine the fine amount based on the number of days
        IF v_days > 30 THEN
            v_fine_amount := (v_days - 30) * 50 + 15 * 5; -- First 30 days fine at Rs. 5, then Rs. 50/day
        ELSIF v_days > 15 THEN
            v_fine_amount := (v_days - 15) * 5; -- Rs. 5/day after 15 days
        END IF;

        -- If there's a fine, insert it into the Fine table
        IF v_fine_amount > 0 THEN
            INSERT INTO Fine (Roll_no, FineDate, Amt)  -- Ensure column names match
            VALUES (p_rollno, SYSDATE, v_fine_amount);
        END IF;

        -- Update the status of the book to 'Returned'
        UPDATE Borrower
        SET Status = 'R'
        WHERE Roll_no = p_rollno AND Name_of_Book = p_bookname;  -- Adjusted for case sensitivity

        COMMIT;
    ELSE
        DBMS_OUTPUT.PUT_LINE('Book already returned.');
    END IF;
END;
/
