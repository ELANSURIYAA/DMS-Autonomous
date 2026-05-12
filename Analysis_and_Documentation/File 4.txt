CREATE OR REPLACE PROCEDURE UpdateCrewAndAircraft AS
    v_crew_id NUMBER;
    v_aircraft_id NUMBER;
    v_crew_id_fm NUMBER;
    v_crew_flight_id_fm NUMBER;
    v_unavailable_id NUMBER := 1;
BEGIN
    FOR crew_c IN (
        SELECT crew_id
        FROM Crew
        WHERE flight_id = 1
    )
    LOOP
        v_crew_id := crew_c.crew_id;
        UPDATE Crew
        SET on_board = 'YES'
        WHERE crew_id = v_crew_id;
    END LOOP;

    FOR a IN (
        SELECT aircraft_id
        FROM Aircraft
        WHERE flight_id = 1
    )
    LOOP
        v_aircraft_id := a.aircraft_id;
        UPDATE Aircraft
        SET ground_status = 'NO', on_flight = 'YES'
        WHERE aircraft_id = v_aircraft_id;
    END LOOP;

    FOR cur_flying_member IN (
        SELECT crew_id, crew_name, on_board, flight_id
        FROM Crew
        WHERE on_board = 'YES' AND flight_id = 1
    )
    LOOP
        v_crew_id_fm := cur_flying_member.crew_id;
        v_crew_flight_id_fm := cur_flying_member.flight_id;
        
        -- insert unvailable crew on a new table
        INSERT INTO currently_unavailable (id, crew_id, flight_id, unavailable_reason)
        VALUES (v_unavailable_id, v_crew_id_fm, v_crew_flight_id_fm, 'Flying');
        
        v_unavailable_id := v_unavailable_id + 1;
    END LOOP;

    COMMIT;
END;
