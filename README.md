# ReportManager
is a lightweight program designed to, at regular time intervals, pull data from SELECTable objects, aggregate the data into a csv-like CLOB and save it in preparation for export of your choice.

## Overview of the database structure
1. [01_create_report_definition](DB_structure/01_create_report_definition.txt)
    - RPMR_RDEF stores data about defined reports
      - A report definition is only checked for a new version during execution planning and the highest version is chosen
      - DATA_SOURCE_OWNER defaults to the owner of the manager package if left as NULL
      - PLAN_FUNCTION contains names of functions which return the next execution time based on the previous execution; see example
2. [02_create_report_execution](DB_structure/02_create_report_execution.txt)
    - RPMR_RX stores data about individual executions of defined reports, past and future
      - EXECUTION_START_PL is the lower bound for when an execution may start
3. [03_create_output_csv_table](DB_structure/03_create_output_csv_table.txt)
    - RPMR_RCLOB stores output clobs associated with report executions
4. [04_create_report_manager_package](DB_structure/04_create_report_manager_package.txt)
    - REPORT_PACKAGE contains the following procedures
      - LOG_START
        - Logs the successful start of a report execution
      - LOG_END
        - Logs the end of a report execution
      - PLAN_NEXT
        - Invokes the PLAN_FUNCTION associated with a given report through report execution in order to plan the next execution
        - If no function is specified, no next execution is planned
      - PREPARE_DATA
        - Retrieves data from the SELECTable object specified in the relevant report definition, builds a csv-like CLOB from the data and stores it in RPMR_RCLOB
      - RUN_MANAGER_LOOP - the only procedure to invoke
        - Loops through all relevant report executions and invokes PREPARE_DATA on each one
      - The package code contains dbms_output debug lines
   
## Example
1. [e01_random_data_view](Example_project/e01_random_data_view.txt)
      - Creates a dummy view to pull data from; this is meant to be the actual data source for reporting
2. [e02_dummy_plan_function](Example_project/e02_dummy_plan_function.txt)
      - Creates a function which will cause subsequent executions of the report to be planned 10 minutes into the future
3. [e03_create_report](Example_project/e03_create_report.txt)
      - Creates a report definition and the first execution of the random data view report
4. [e04_run_report](Example_project/e04_run_report.txt)
      - Runs the report manager loop

## Remarks
- The choice of automation solution is up to the developer, all it has to do is invoke RUN_MANAGER_LOOP at a regular time interval compatible with intervals specified through planning functions.
- The choice of means of export is up to the developer. The original intent was to include an smtp solution, but I came to the conclusion that coverage of all possibilities within that direction would not be worth the effort.

## License

MIT License

Copyright (c) 2024 GateDigger

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
