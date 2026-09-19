An enterprise-style bank reconciliation automation built with UiPath.
 
This repository contains the **Dispatcher** component of a two-process automation architecture.
 
The solution uses:
 
- UiPath Studio
- UiPath Orchestrator Queues
- Dispatcher / Performer architecture
- REFramework for transaction processing
- Excel-based transaction input
- Queue-based workload distribution
- Exception handling and retry support
- Audit-friendly transaction processing
 
---
 
## Architecture
 
The automation is split into two independent UiPath processes.
 
```text
Bank Statement / Input File
            |
            v
+---------------------------+
| Bank Reconciliation       |
| Dispatcher                |
|                           |
| - Read input file         |
| - Validate records        |
| - Prepare queue data      |
| - Add queue items         |
+-------------+-------------+
              |
              v
      UiPath Orchestrator
        BR_RECON_QUEUE
              |
              v
+---------------------------+
| Bank Reconciliation       |
| Performer                 |
|                           |
| REFramework               |
| - Get Queue Item          |
| - Process transaction     |
| - Match / reconcile       |
| - Handle exceptions       |
| - Retry system failures   |
| - Set transaction status  |
+-------------+-------------+
              |
              v
     Reconciliation Output
Dispatcher
This repository contains the Dispatcher.
Its responsibility is to collect transactions from the source file, validate them and create transaction items inside the UiPath Orchestrator queue.
The Dispatcher does not perform the full reconciliation itself.
Transaction-level processing is delegated to the separate REFramework Performer.
Dispatcher Workflow
START
  |
  v
Load Bank Statement
  |
  v
Validate Input Data
  |
  v
Transform Transaction Data
  |
  v
For Each Transaction
  |
  v
Create Queue Item
  |
  v
BR_RECON_QUEUE
  |
  v
END
Orchestrator Queue
The solution uses an Orchestrator queue called:
BR_RECON_QUEUE
Each valid bank transaction is submitted as an individual queue item.
This enables the Performer to process transactions independently.
Using an Orchestrator Queue provides:
Transaction isolation
Retry capability
Centralized monitoring
Queue status tracking
Scalability
Better exception handling
Auditability
Dispatcher / Performer Pattern
The solution follows the common UiPath Dispatcher/Performer architecture.
Dispatcher
Responsible for:
Reading source transactions
Validating required information
Transforming data
Creating queue items
Preventing invalid data from entering the processing stage
Performer
Responsible for:
Retrieving transactions from Orchestrator
Performing reconciliation logic
Identifying matched and unmatched transactions
Handling business exceptions
Handling system exceptions
Retrying recoverable failures
Updating queue transaction status
Producing processing results
Performer Repository
The Performer is maintained as a separate UiPath project and uses the Robotic Enterprise Framework (REFramework).
Repository:
uipath-bank-reconciliation-performer
GitHub link will be added here once the Performer repository is published.
Why Two Processes?
Separating the automation into Dispatcher and Performer components provides several advantages.
The Dispatcher focuses only on preparing work.
The Performer focuses only on executing individual transactions.
This allows the automation to:
Process large transaction volumes
Resume processing after failures
Retry failed system transactions
Separate data ingestion from business processing
Scale to multiple robots
Monitor individual transactions through Orchestrator
Recover without restarting the entire batch
Project Structure
Example structure:
BankReconciliation_Dispatcher/
│
├── Main.xaml
├── project.json
├── project.uiproj
├── entry-points.json
├── .gitignore
├── README.md
│
└── Data/
    └── Sample_Bank_Statement.xlsx
Additional workflows may be moved into dedicated folders as the project grows.
Example:
Workflows/
├── ReadBankStatement.xaml
├── ValidateTransactions.xaml
├── PrepareQueueData.xaml
└── AddQueueItems.xaml
Sample Data
Only fictional or sanitized transaction data should be included in this repository.
Real banking information, customer information, credentials, account numbers or production files must never be committed to source control.
Example test file:
Sample_Bank_Statement.xlsx
Security
Sensitive information must not be stored directly inside workflows or committed to GitHub.
Examples include:
Passwords
API keys
Customer banking information
Production account numbers
Access tokens
Orchestrator credentials
Production secrets should instead be managed using mechanisms such as:
UiPath Orchestrator Assets
Credential Assets
External secret-management systems
Environment-specific configuration
Requirements
To run the project on another computer:
UiPath Studio Desktop
Git
Access to a UiPath Orchestrator environment
Required UiPath activity packages
An Orchestrator Queue named BR_RECON_QUEUE
Sample or approved input data
Running on Another PC
Clone the repository:
git clone https://github.com/YOUR_USERNAME/uipath-bank-reconciliation-dispatcher.git
Open the cloned project using UiPath Studio.
UiPath Studio will restore the dependencies defined by the project.
Local Studio-generated metadata and cache files are intentionally excluded from Git source control and will be recreated automatically.
Connect the Studio/Robot environment to the appropriate UiPath Orchestrator tenant.
Make sure the following queue exists:
BR_RECON_QUEUE
Then run:
Main.xaml
Exception Strategy
The Dispatcher should prevent invalid records from entering the transaction queue.
Examples of validation failures include:
Missing transaction reference
Missing transaction amount
Invalid date
Invalid account information
Duplicate transaction
Invalid transaction format
Transaction-processing errors are primarily handled by the Performer through REFramework.
The Performer differentiates between:
Business Exception
and:
System Exception
System exceptions may be retried according to the configured Orchestrator/REFramework retry strategy.
Disaster Recovery Strategy
The queue-based architecture helps make the automation recoverable.
If the Performer stops unexpectedly:
Successfully processed queue items remain completed.
Failed transactions can be identified.
Recoverable system exceptions can be retried.
New Performer sessions can continue processing remaining queue items.
The complete batch does not need to restart.
This reduces the risk of duplicate processing and improves operational resilience.
Future Enhancements
Planned improvements include:
Advanced duplicate detection
Automated reconciliation rules
Configurable matching tolerances
API-based transaction ingestion
Database integration
Automated reconciliation reports
Email notifications
Orchestrator monitoring
Transaction dashboards
Audit logging
Enhanced disaster recovery
Automated testing
Technologies
UiPath Studio
UiPath Orchestrator
REFramework
Orchestrator Queues
Microsoft Excel
Git
GitHub
Related Project
Bank Reconciliation Performer
The second component of this solution uses UiPath REFramework to consume transactions from BR_RECON_QUEUE.
It demonstrates:
REFramework state machine architecture
Queue transaction processing
Business exceptions
System exceptions
Automatic retry
Transaction status management
Logging
Recovery
Reconciliation processing
The Performer repository will be linked here once published.
Purpose
This project was developed as a portfolio implementation of enterprise UiPath automation architecture, with particular focus on:
Financial-process automation
Queue-based transaction processing
Reliability
Recoverability
Maintainability
Separation of responsibilities
Enterprise RPA development practices
 
Then make one small change before committing: replace:
 
```tex
