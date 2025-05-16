
# Accrua Service Endpoint

This code implements a RESTful API endpoint for running the interest accrual and penalty accrual:

### Controller Layer

- Defines a POST endpoint at `/api/v2/EOD/run`
- Validates the request and handles validation errors
- Accepts EodRunReigger request body

### Service Layer

The service accept the request and send the request to apache camel:
- Apache camel recieve the request and process the request in the following order


1. **updateLoanWithLatestEod**: this method perform the following
   - Fecth loan  from loan table where status is disbussed and
   - Identify disbursed loans missing their EOD markers.
   - Determine for each either (a) the earliest upcoming repayment schedule or (b) the day before disbursement.
   - Update the loan record lm_last_successful_eod_date, lm_last_successful_eod_schedule_id.
     
3. **bulkUpdatePaymentStatusDue**: this method perform the following
   - fecth loan from loan and lm_loan_repayment_schedule table where lm_payment_status is NOT_DUE and lm_total_amount is 0
   - Update the Payment Status to DUE

4. **processAccrual**: this method perform the following
   - Log eg activity started
   - get fiscal peroid from  fiscal service and check if fiscal peroid is not null
   - fetch loan where is DISBURSED, EXPIRED and lm_loan_repayment_schedule where DUE, LATE, PARTIALLY_PAID
   - check if AccrualAmount is not null and LastSuccessfulEodScheduleId and  LastSuccessfulEodDate is not > greater than the CobDate
   - calculate accrual amount
   - save to detail to lm_eod_accrual_record,
   - update loan with LastSuccessfulEodDate
   - save to journal posting
   - update lm_loan_repayment_schedule with accrual amount

flowchart TD
    A[Log activity started]
    B[Get fiscal period from fiscal service]
    C{Fiscal period not null?}
    D[Fetch loans<br/>(DISBURSED, EXPIRED)]
    E[Fetch repayment schedules<br/>(DUE, LATE, PARTIALLY_PAID)]
    F{AccrualAmount not null<br/>AND LastSuccessfulEodScheduleId &<br/>LastSuccessfulEodDate ≤ COB date?}
    G[Calculate accrual amount]
    H[Save detail to lm_eod_accrual_record]
    I[Update loan with LastSuccessfulEodDate]
    J[Save to journal posting]
    K[Update lm_loan_repayment_schedule<br/>with accrual amount]
    L[End or error]

    A --> B
    B --> C
    C -- Yes --> D
    C -- No --> L
    D --> E
    E --> F
    F -- Yes --> G
    F -- No --> L
    G --> H
    H --> I
    I --> J
    J --> K
    K --> L


5. **runPenalAccrual**:
   - get fiscal peroid from  fiscal service and check if fiscal peroid is not null
   - fetch loan where is DISBURSED, EXPIRED , lm_loan_repayment_schedule where DUE, LATE, PARTIALLY_PAID and grace period is less than the cobDate
   - check if total accrual amount is not zero
   - check if AccrualAmount is not null and LastSuccessfulEodScheduleId and  LastSuccessfulEodDate is not > greater than the CobDate
   - save detials to lm_eod_penalty_record
   - save to journal posting
   - update loan with LastSuccessfulEodPenalDate
   - update lm_loan_repayment_schedule with penalty accrual amount


6. **closePaidOfLoan** 
   - fetch loan where status is Disbussed and lm_loan_repayment_schedule all schedule are PAID
   - Make api to call to close loan on account on customer acccount,
   - set the loan to close

7. **requeryJournalPostingRecord**:
   - Make Api call to journal posting to get status of transaction
   - updatate the recored to success 

8. **runLoanPerformanceScheduler**:
   - fetch loan with status as  DISBURSED,
   - fetch lm_loan_repayment_schedule with loan id
   - run performance classification based on the set data on lm_performance_configuration


10. **updateRepaymentScheduleStatusToLate**:
 
11. **updateLoanToExpired**:



12. **Exception Handling**: Catches exceptions, logs audit information for failures, and rethrows the exception

## Detailed Flow Diagram
