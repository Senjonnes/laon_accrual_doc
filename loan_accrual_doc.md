
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

5. **runPenalAccrual**:
   - get fiscal peroid from  fiscal service and check if fiscal peroid is not null
   - fetch loan where is DISBURSED, EXPIRED , lm_loan_repayment_schedule where DUE, LATE, PARTIALLY_PAID and grace period is less than the cobDate
   - check if total accrual amount is not zero
   - check if AccrualAmount is not null and LastSuccessfulEodScheduleId and  LastSuccessfulEodDate is not > greater than the CobDate
   - save detials to lm_eod_penalty_record
   - save to journal posting
   - update loan with LastSuccessfulEodPenalDate
   - update lm_loan_repayment_schedule with penalty accrual amount


6. **closePaidOfLoan** (Individual service only):
   - fetch loan where status is Disbussed and lm_loan_repayment_schedule all schedule are PAID
   - Make api to call to close loan on account on customer acccount,
   - set the loan to close

7. **requeryJournalPostingRecord**:
   - Make Api call to journal posting to get status of transaction
   - updatate the recored to success 

8. **runLoanPerformanceScheduler**:


9. **updateRepaymentScheduleStatusToLate**:
 
10. **updateLoanToExpired**:



12. **Exception Handling**: Catches exceptions, logs audit information for failures, and rethrows the exception

## Detailed Flow Diagram
