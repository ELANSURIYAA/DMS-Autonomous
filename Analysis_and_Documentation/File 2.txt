CREATE OR REPLACE PROCEDURE InsertPassengersFlightsSeatsServices AS
    CURSOR PassengerCursor IS
        SELECT * FROM Passenger;
    
    CURSOR FlightCursor IS
        SELECT * FROM Flight_Details;
    
    CURSOR SeatCursor IS
        SELECT * FROM Seat_Details;
    
    CURSOR ServiceCursor IS
        SELECT * FROM Flight_Service;
    
    TYPE PassengerType IS TABLE OF PassengerCursor%ROWTYPE;
    TYPE FlightType IS TABLE OF FlightCursor%ROWTYPE;
    TYPE SeatType IS TABLE OF SeatCursor%ROWTYPE;
    TYPE ServiceType IS TABLE OF ServiceCursor%ROWTYPE;
    
    PassengerData PassengerType;
    FlightData FlightType;
    SeatData SeatType;
    ServiceData ServiceType;
BEGIN
    -- Retrieve Passenger data
    OPEN PassengerCursor;
    LOOP
        FETCH PassengerCursor BULK COLLECT INTO PassengerData LIMIT 100;
        EXIT WHEN PassengerData.COUNT = 0;

        -- Insert Passenger data
        FORALL i IN 1..PassengerData.COUNT
            INSERT INTO Passenger_backup (Passenger_ID, P_FirstName, P_LastName, P_Email, P_PhoneNumber, P_Address, P_City, P_State, P_Zipcode, P_Country)
            VALUES (PassengerData(i).Passenger_ID, PassengerData(i).P_FirstName, PassengerData(i).P_LastName, PassengerData(i).P_Email, PassengerData(i).P_PhoneNumber, PassengerData(i).P_Address, PassengerData(i).P_City, PassengerData(i).P_State, PassengerData(i).P_Zipcode, PassengerData(i).P_Country);
    END LOOP;
    CLOSE PassengerCursor;

    -- Retrieve Flight data
    OPEN FlightCursor;
    LOOP
        FETCH FlightCursor BULK COLLECT INTO FlightData LIMIT 100;
        EXIT WHEN FlightData.COUNT = 0;

        -- Insert Flight data
        FORALL i IN 1..FlightData.COUNT
            INSERT INTO Flight_Details_backup (Flight_ID, Source_Airport_ID, Destination_Airport_ID, Departure_Date_Time, Arrival_Date_Time, Airplane_Type)
            VALUES (FlightData(i).Flight_ID, FlightData(i).Source_Airport_ID, FlightData(i).Destination_Airport_ID, FlightData(i).Departure_Date_Time, FlightData(i).Arrival_Date_Time, FlightData(i).Airplane_Type);
    END LOOP;
    CLOSE FlightCursor;

    -- Retrieve Seat data
    OPEN SeatCursor;
    LOOP
        FETCH SeatCursor BULK COLLECT INTO SeatData LIMIT 100;
        EXIT WHEN SeatData.COUNT = 0;

        -- Insert Seat data
        FORALL i IN 1..SeatData.COUNT
            INSERT INTO Seat_Details_backup (Seat_ID, Travel_Class_ID, Flight_ID)
            VALUES (SeatData(i).Seat_ID, SeatData(i).Travel_Class_ID, SeatData(i).Flight_ID);
    END LOOP;
    CLOSE SeatCursor;

    -- Retrieve Flight Service data
    OPEN ServiceCursor;
    LOOP
        FETCH ServiceCursor BULK COLLECT INTO ServiceData LIMIT 100;
        EXIT WHEN ServiceData.COUNT = 0;

        -- Insert Flight Service data
        FORALL i IN 1..ServiceData.COUNT
            INSERT INTO Flight_Service_backup (Service_ID, Service_Name)
            VALUES (ServiceData(i).Service_ID, ServiceData(i).Service_Name);
    END LOOP;
    CLOSE ServiceCursor;
    
    COMMIT;
END InsertPassengersFlightsSeatsServices;
