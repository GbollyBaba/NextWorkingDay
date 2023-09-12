# NextWorkingDay


POSTGRES DB SCRIPT
**********************
Note the Postgres database is  of version 16.

CREATE TABLE public_holidays (
    holiday_date date PRIMARY KEY
);

create some notable  holidays
insert into public_holidays values ('2023-12-25');
insert into public_holidays values ('2023-12-26');
insert into public_holidays values ('2023-11-12');
insert into public_holidays values ('2023-01-01');
insert into public_holidays values ('2024-01-01');

CREATE OR REPLACE FUNCTION get_next_working_day(input_date date DEFAULT NULL)
RETURNS TABLE (next_working_day date, next_working_day_name text)
AS $$
DECLARE
  next_working_date date;
BEGIN
  IF input_date IS NULL THEN
    input_date := current_date;
  END IF;

  SELECT
    CASE
      WHEN EXTRACT(ISODOW FROM input_date) = 5 THEN input_date + INTERVAL '3 days' -- If input_date is Friday, add 3 days (skip Saturday and Sunday)
      WHEN EXTRACT(ISODOW FROM input_date) = 6 THEN input_date + INTERVAL '2 days' -- If input_date is Saturday, add 2 days (skip Sunday)
      ELSE input_date + INTERVAL '1 day' -- For all other days, add 1 day (skip weekends)
    END
  INTO next_working_date;

  LOOP
    -- Check if the next working-day is a public holiday
    IF EXISTS (SELECT 1 FROM public_holidays WHERE holiday_date = next_working_date) THEN
      next_working_date := next_working_date + INTERVAL '1 day'; -- Skip public holidays
    ELSE
      EXIT; -- Exit the loop when a valid working day is found
    END IF;
  END LOOP;

  SELECT
    next_working_date,
    CASE
      WHEN EXTRACT(ISODOW FROM next_working_date) = 5 THEN 'Monday' -- If input_date is Friday, the next working day is Monday
      WHEN EXTRACT(ISODOW FROM next_working_date) = 6 THEN 'Monday' -- If input_date is Saturday, the next working day is Monday
      ELSE to_char(next_working_date, 'Day') -- For all other days, return the day name
    END
  INTO next_working_day, next_working_day_name;

  RETURN NEXT;
END;
$$ LANGUAGE plpgsql;

/*

TEST THE DATABASE LAYER LOGIC

-- Call the function with a specific date
SELECT * FROM get_next_working_day('2023-09-11');

SELECT * FROM get_next_working_day('2023-09-09');

-- Call the function without providing a date (uses the current date)
SELECT * FROM get_next_working_day();

*/


P12 KESTORE GENERATION for SSL
*************************************
---This   generate at the directory level where the pom.xml file is located
keytool -genkeypair -alias gbolly -keyalg RSA -keysize 4096 \
  -validity 3650 -dname "CN=localhost" -keypass gboladeshada -keystore keystore.p12 \
  -storeType PKCS12 -storepass gboladeshada
  
  
  The AUthorization is based on Basic Auth.
  
  Username :  user
  Password :  <password generated when the server starts up >
  
  
  
  
  API SERVICE CONTRACT DEFINITION 
  ********************************
  
  this can be viewed    using the postman
  I have attached the  exported postman collection.
  https://localhost:8443/v2/api-docs
  
  
  
  
  
