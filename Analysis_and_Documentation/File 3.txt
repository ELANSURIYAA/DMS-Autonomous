
CREATE OR REPLACE PROCEDURE FlightManagementBeltAndFoodCoordination IS
BEGIN
    MERGE INTO Flights f
    USING (
        SELECT
            flight_id,
            total_seats,
            confirmed_seats,
            night_flight,
            transnational
        FROM
            Flights
    ) flight_rec
    ON (f.flight_id = flight_rec.flight_id)
    WHEN MATCHED THEN
        UPDATE
        SET f.price = CASE WHEN flight_rec.night_flight = 'Y' THEN f.price - (f.price * 0.10) ELSE f.price END, -- discount for night flight
            f.confirmation_status = CASE WHEN flight_rec.confirmed_seats = flight_rec.total_seats THEN 'YES' ELSE 'NO' END, -- confirmation status for all seats
            f.promotion_status = CASE WHEN flight_rec.confirmed_seats / flight_rec.total_seats >= 0.8 THEN 'AVAILABLE' ELSE 'SOLD OUT' END, -- add extra promotion if few sells
            f.crew_count = CASE
                WHEN flight_rec.night_flight = 'Y' AND flight_rec.confirmed_seats < 0.25 * flight_rec.total_seats THEN 3
                WHEN flight_rec.night_flight = 'Y' AND flight_rec.confirmed_seats >= 0.6 * flight_rec.total_seats THEN 6
                WHEN flight_rec.night_flight = 'Y' THEN 5
                WHEN flight_rec.confirmed_seats >= 0.6 * flight_rec.total_seats THEN 5
                ELSE 2
            END
    WHEN NOT MATCHED THEN
        INSERT (flight_id, total_seats, confirmed_seats, night_flight, transnational, price, confirmation_status, promotion_status, crew_count)
        VALUES (flight_rec.flight_id, flight_rec.total_seats, flight_rec.confirmed_seats, flight_rec.night_flight, flight_rec.transnational, NULL, NULL, NULL, NULL);

    -- Check for transnational flights and add services
    INSERT INTO Services (service_id, flight_id, service_name)
    SELECT service_id_seq.NEXTVAL, flight_id, 'Wine'
    FROM Flights
    WHERE transnational = 'Y';

    INSERT INTO Services (service_id, flight_id, service_name)
    SELECT service_id_seq.NEXTVAL, flight_id, 'Extra Blanket'
    FROM Flights
    WHERE transnational = 'Y';

    INSERT INTO Services (service_id, flight_id, service_name)
    SELECT service_id_seq.NEXTVAL, flight_id, 'Pillows'
    FROM Flights
    WHERE transnational = 'Y';

    INSERT INTO Services (service_id, flight_id, service_name)
    SELECT service_id_seq.NEXTVAL, flight_id, 'Food'
    FROM Flights
    WHERE transnational = 'Y';

    -- Check for short flights and add dessert service
    INSERT INTO Services (service_id, flight_id, service_name)
    SELECT service_id_seq.NEXTVAL, flight_id, 'Dessert'
    FROM Flights
    WHERE total_seats < 500;

    -- Award points to passengers for each flight
    INSERT INTO passenger_miles_points (passenger_id, flight_id, points_earned)
    SELECT p.passenger_id, p.flight_id, 5
    FROM Passengers p
    JOIN Flights f ON p.flight_id = f.flight_id;

    -- Check for children under 10 years and assign special belt requirement
    INSERT INTO passengers_with_special_belt (passenger_id, flight_id, special_belt_requirement)
    SELECT p.passenger_id, p.flight_id, 'REQUIRED'
    FROM Passengers p
    JOIN Flights f ON p.flight_id = f.flight_id
    WHERE MONTHS_BETWEEN(SYSDATE, p.DATE_OF_BIRTH) / 12 < 10;

    INSERT INTO special_food_requirements (passenger_id, flight_id, special_food_requirement, souvenir)
    SELECT p.passenger_id, p.flight_id, 'REQUIRED', 'YES'
    FROM Passengers p
    JOIN Flights f ON p.flight_id = f.flight_id
    WHERE MONTHS_BETWEEN(SYSDATE, p.DATE_OF_BIRTH) / 12 < 10;

    COMMIT;
END;
