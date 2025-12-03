# Student Marksheet Dispatcher

**Part of the Automated Marksheet Generation System**

A UiPath automation project that reads student data from CSV files and populates an Orchestrator queue for processing. This is the **Dispatcher** component of a comprehensive Dispatcher-Performer architecture that automates end-to-end student marksheet generation, grade calculation, PDF creation, and email delivery.

## 📋 Overview

This Dispatcher is the **entry point** of the automated marksheet generation pipeline. It handles the initial data ingestion:

1. **Reads** student academic data from CSV file (`Sample_Student_Data.csv`)
2. **Validates** the presence and integrity of the input file
3. **Creates** queue items in UiPath Orchestrator's `MarksheetQueue`
4. **References** each transaction using student Roll Number as unique identifier
5. **Logs** processing status and results for audit trails

### 🔗 System Architecture

This Dispatcher works in tandem with the **Performer** component (REFramework-based) to deliver a complete solution:

```
┌─────────────────────────────────────────────────────────────┐
│                  COMPLETE WORKFLOW                           │
└─────────────────────────────────────────────────────────────┘

   DISPATCHER (This Project)           ORCHESTRATOR              PERFORMER (REFramework)
┌─────────────────────┐            ┌───────────────┐       ┌──────────────────────────┐
│  Read CSV           │   Add      │ MarksheetQueue│ Get   │ Main.xaml                │
│  Sample_Student_    │  ───────►  │               │ ◄───  │ ├── Initialize           │
│  Data.csv           │  Items     │ [Queue Items] │ Trans │ ├── GetTransactionData   │
│                     │            │               │       │ ├── Process Transaction  │
│ Parse & Validate    │            │ Reference:    │       │ │   ├── Validate Data    │
│ 17 Columns          │            │ RollNumber    │       │ │   ├── Calculate GPA    │
│                     │            │               │       │ │   ├── Generate PDF     │
│ Populate Queue      │            │ Status:       │       │ │   └── Verify Output    │
│ (All Students)      │            │ New/Success/  │       │ ├── SetTransactionStatus │
│                     │            │ Failed        │       │ └── End Process          │
└─────────────────────┘            └───────────────┘       │     └── Send Emails      │
                                                            └──────────────────────────┘
                                                                        │
                                                                        ▼
                                                            ┌──────────────────────────┐
                                                            │ OUTPUT                   │
                                                            │ ├── PDF Marksheets       │
                                                            │ └── Email Notifications  │
                                                            └──────────────────────────┘
```

**What Happens After Dispatch:**
- Performer retrieves queue items one by one
- Calculates grades (A-F scale) and GPA (10-point scale)
- Generates professional PDF marksheets using Python + ReportLab
- Updates queue status (Success/Failed)
- Sends personalized emails to students or bulk emails to admin
- Handles errors with REFramework retry logic

## 🏗️ Project Structure

```
Final Project Dispatcher/
│
├── AddTestQueueItem.xaml          # Main workflow file
├── project.json                   # Project configuration and dependencies
├── project.uiproj                 # UiPath project file
│
├── Data/
│   └── Input/
│       └── Sample_Student_Data.csv    # Student data input file (17 columns)
│
├── .entities/                     # UiPath entities metadata
├── .git/                          # Git version control
├── .local/                        # Local runtime files
├── .objects/                      # Compiled workflow objects
├── .project/                      # Project settings
├── .settings/                     # UiPath Studio settings
├── .templates/                    # Workflow templates
└── .tmh/                          # Telemetry and metadata
```

## 📊 Input Data Format

The CSV file (`Sample_Student_Data.csv`) contains **17 columns**:

| Column | Description | Type |
|--------|-------------|------|
| Name | Student's full name | String |
| RollNumber | Unique student identifier | String |
| DOB | Date of birth | String |
| Semester | Academic semester | String |
| Year | Academic year | String |
| Department | Department name | String |
| Subject1 - Subject5 | Subject names | String |
| Mark1 - Mark5 | Corresponding marks | Integer |
| Email | Student email address | String |

### Sample Data

The project includes **10 student records** with Computer Science students' academic information for Fall 2024 semester.

## 🔧 Technical Details

### Dependencies

- **UiPath.Excel.Activities**: Version 3.3.1
- **UiPath.System.Activities**: Version 25.10.3
- **Target Framework**: Windows
- **Studio Version**: 26.0.181.0

### Workflow Logic (`AddTestQueueItem.xaml`)

1. **File Validation**
   - Checks if CSV file exists at path: `Data\Input\Sample_Student_Data.csv`
   - Throws exception if file is not found
   - Logs confirmation when file exists

2. **Data Reading**
   - Uses `Read CSV` activity to load data into DataTable
   - Logs total number of rows read
   
3. **Queue Item Creation**
   - Iterates through each row in the DataTable
   - Creates Orchestrator queue item with:
     - **Queue Name**: `MarksheetQueue`
     - **Reference**: Student's `RollNumber` (unique identifier)
     - **Priority**: Normal
     - **Folder Path**: `220701283@rajalakshmi.edu.in's workspace`
   
4. **Queue Item Information**
   - All 16 data fields are passed as specific transaction items:
     - String fields: Name, RollNumber, DOB, Semester, Year, Department, Subject1-5
     - Integer fields: Mark1-5 (converted using `CInt()`)

5. **Completion Logging**
   - Logs success message: "Test queue item added successfully!"

### Variables

- `csvPath`: File path to input CSV (hardcoded)
- `queueName`: Name of Orchestrator queue ("MarksheetQueue")
- `dtStudents`: DataTable to store CSV data
- `currentRow`: Loop counter for row iteration
- `csvExists`: Boolean flag for file validation

## 🚀 Usage Instructions

### Prerequisites

1. **UiPath Orchestrator Access**
   - Ensure you're connected to UiPath Orchestrator
   - Have appropriate permissions to add queue items

2. **Queue Setup**
   - Create a queue named `MarksheetQueue` in Orchestrator
   - Navigate to: Orchestrator → Queues → Add Queue

3. **Input Data**
   - Prepare CSV file with the required 17-column format
   - Place file at: `Data/Input/Sample_Student_Data.csv`

### Running the Dispatcher

**Option 1: From UiPath Studio**
```
1. Open the project in UiPath Studio
2. Press F5 or click "Run"
3. Monitor execution in the Output panel
```

**Option 2: From UiPath Assistant**
```
1. Publish the project to Orchestrator
2. Open UiPath Assistant
3. Locate "Final Project Dispatcher"
4. Click "Run"
```

### Verification

1. **Check Execution Logs**
   - Verify log messages:
     - "CSV Exists"
     - "Rows read: [count]"
     - "Test queue item added successfully!"

2. **Verify Queue Items in Orchestrator**
   ```
   Orchestrator → Queues → MarksheetQueue → View Items
   ```
   - Confirm all student records appear as queue items
   - Check Reference field shows RollNumber values
   - Verify Status is "New"

## 🔗 Integration with Performer (REFramework)

This Dispatcher is **Part 1** of a two-component system. Understanding the complete workflow helps contextualize the dispatcher's role.

### Complete End-to-End Flow

**Phase 1: Dispatcher (This Project)**
1. Parse CSV file with 10 student records
2. Create 10 queue items in `MarksheetQueue`
3. Each queue item contains all 16 student data fields
4. Dispatcher completes and exits

**Phase 2: Performer (Separate REFramework Project)**
1. **Initialize** - Load configuration, verify Python, create Output folder
2. **Get Transaction** - Retrieve queue items one by one from `MarksheetQueue`
3. **Process Transaction** - For each student:
   - Validate marks (0-100 range)
   - Calculate letter grades (A-F based on marks)
   - Calculate GPA (10-point scale)
   - Execute Python script to generate PDF marksheet
   - Verify PDF created successfully
4. **Set Transaction Status** - Update queue status (Success/Failed)
5. **End Process** - Send email notifications with PDFs attached

### Performer Features

The downstream Performer project includes:

#### 🎯 Grade Calculation Engine
- **Letter Grades:** A (90-100), B (80-89), C (70-79), D (60-69), E (50-59), F (<50)
- **GPA Calculation:** Overall GPA on 10-point scale
- **Data Validation:** Business rule exceptions for invalid marks

#### 📄 Professional PDF Generation
- **Technology:** Python 3.14 + ReportLab library
- **Layout:** Institutional branding with student details, marks table, grades, GPA
- **Features:** Grade scale reference, signatures, timestamps
- **Output Location:** `Data/Output/Marksheet_[RollNumber].pdf`

#### 📧 Advanced Email Automation
- **Dual Modes:** 
  - Individual emails to each student (`rollnumber@rajalakshmi.edu.in`)
  - Bulk admin email with all marksheets
- **Configuration:** Interactive checkbox UI at runtime
- **Storage:** Email preferences saved as Orchestrator Assets
- **Content:** Personalized subject lines, grade summaries, PDF attachments

#### 🔐 Robust Error Handling
- **Business Exceptions:** Invalid data (marks out of range, missing fields)
- **Application Exceptions:** Python errors, file system issues
- **System Exceptions:** Network failures, SMTP errors
- **Retry Logic:** Automatic retry for transient failures
- **Comprehensive Logging:** Every step logged to Orchestrator

### Performance Metrics

| Metric | Value |
|--------|-------|
| **Processing Time** | ~5-8 seconds per student |
| **Throughput** | 10 students in ~60 seconds |
| **Success Rate** | 100% with valid data |
| **PDF Size** | ~15-20 KB each |
| **Scalability** | 1000 students in ~100 minutes |

### Technology Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Dispatcher** | UiPath Workflow | Queue population |
| **Performer** | UiPath REFramework | Transaction processing |
| **Queue** | Orchestrator `MarksheetQueue` | Transaction coordination |
| **PDF Engine** | Python + ReportLab | Professional document generation |
| **Email** | SMTP (Gmail) | Notification delivery |
| **Configuration** | Excel + Orchestrator Assets | Centralized settings |

### Related Documentation

For complete system details, refer to the **Performer Project README** which covers:
- REFramework implementation details
- Python script configuration
- Email setup and troubleshooting
- Configuration file structure
- Testing procedures
- Future enhancements



## 📝 Configuration

### Modifying the CSV Path

To change the input file location, update the `csvPath` variable in `AddTestQueueItem.xaml`:

```xml
<Variable x:TypeArguments="x:String" Name="csvPath">
  <Variable.Default>
    <Literal x:TypeArguments="x:String">YOUR_NEW_PATH_HERE</Literal>
  </Variable.Default>
</Variable>
```

### Changing Queue Name

To use a different queue, modify the `queueName` variable:

```xml
<Variable x:TypeArguments="x:String" Default="YourQueueName" Name="queueName" />
```

## 🐛 Troubleshooting

### Dispatcher-Specific Issues

**Issue**: "CSV not found" exception
- **Solution**: Verify the CSV file exists at `Data\Input\Sample_Student_Data.csv`
- Check file path is correct (relative to project root)
- Ensure file has not been moved or renamed

**Issue**: Queue items not appearing in Orchestrator
- **Solution**: 
  - Confirm queue `MarksheetQueue` exists in Orchestrator
  - Check Orchestrator connection in Studio (Project → Settings → Orchestrator)
  - Verify folder path matches: `220701283@rajalakshmi.edu.in's workspace`
  - Check user permissions for queue creation

**Issue**: Data type conversion errors
- **Solution**: Ensure Mark1-5 columns contain valid integer values
- Check CSV format matches expected structure (all 17 columns present)
- Verify no empty cells or non-numeric characters in mark columns
- Open CSV in text editor to check for encoding issues

**Issue**: Duplicate queue items created
- **Solution**: 
  - Check if dispatcher ran multiple times accidentally
  - Clear queue in Orchestrator before re-running
  - Verify CSV doesn't have duplicate RollNumber values

### Downstream Performer Issues

These issues occur in the Performer project but originate from Dispatcher data:

**Issue**: Performer fails with "Invalid marks" business exception
- **Root Cause**: CSV contains marks outside 0-100 range
- **Solution**: Validate CSV data before running Dispatcher
- **Prevention**: Add validation logic to Dispatcher (future enhancement)

**Issue**: Performer fails with "Missing field" error
- **Root Cause**: CSV missing required columns (Email, Subject names, etc.)
- **Solution**: Ensure CSV has all 17 columns with proper headers
- **Check**: Open `Sample_Student_Data.csv` and verify column structure

**Issue**: Queue items stuck in "In Progress" status
- **Root Cause**: Performer crashed mid-processing
- **Solution**: 
  - Manually set queue items back to "New" in Orchestrator
  - Re-run Performer
  - Check Performer logs for crash cause

**Issue**: PDFs not generated even though queue shows "Successful"
- **Possible Causes**: 
  - Python not installed on Performer machine
  - ReportLab library missing (`pip install reportlab`)
  - Output folder permissions issue
- **Solution**: See Performer README troubleshooting section

**Issue**: Email not sent but marksheets generated
- **Root Cause**: SMTP configuration issue in Performer
- **Impact**: Non-critical - PDFs are still created
- **Solution**: Check Performer email configuration
  - Gmail users: Use App Password, not regular password
  - Enable 2-Factor Authentication
  - See Performer README for detailed SMTP setup

### System Integration Issues

**Issue**: Orchestrator Robot Error (#1230)
- **Cause**: Unattended robot not configured (license constraints)
- **Solution**: Run Dispatcher from UiPath Studio (F5) or UiPath Assistant
- **Note**: This affects both Dispatcher and Performer

**Issue**: Queue reference mismatch
- **Symptom**: Performer can't find queue items created by Dispatcher
- **Solution**: 
  - Verify both projects use same queue name: `MarksheetQueue`
  - Check folder path matches in both projects
  - Ensure both connected to same Orchestrator tenant

**Issue**: End-to-end test failing
- **Debugging Steps:**
  1. Run Dispatcher → Check queue populated
  2. Verify queue items in Orchestrator → Check data fields
  3. Run Performer → Monitor logs in Output panel
  4. Check for PDFs in `Data/Output/` folder
  5. Review email inbox for notifications
  6. Check queue status (should be "Successful")

### Data Quality Checklist

Before running Dispatcher, verify CSV contains:
- ✅ All 17 columns with exact header names
- ✅ No empty cells in required fields
- ✅ Marks between 0-100 (inclusive)
- ✅ Valid email format (`rollnumber@rajalakshmi.edu.in`)
- ✅ Unique RollNumber for each student
- ✅ Proper date format for DOB
- ✅ No special characters that break CSV parsing

## � Quick Start Guide

### Running the Complete System

**Step 1: Setup Prerequisites**
```
1. Install UiPath Studio (v26.x or later)
2. Create MarksheetQueue in Orchestrator
3. Connect Studio to Orchestrator
4. Install Python 3.14 and ReportLab library (for Performer)
```

**Step 2: Run Dispatcher**
```
1. Open this project in UiPath Studio
2. Verify CSV exists: Data/Input/Sample_Student_Data.csv
3. Press F5 to run
4. Verify success message: "Test queue item added successfully!"
5. Check Orchestrator → MarksheetQueue shows 10 items
```

**Step 3: Run Performer**
```
1. Open Performer project (REFramework)
2. Configure email settings (optional)
3. Press F5 to run
4. Wait for processing (~60 seconds for 10 students)
5. Verify PDFs created in Data/Output/
6. Check emails sent (if configured)
```

**Step 4: Verify Results**
- Check Orchestrator queue: All items show "Successful"
- Check Output folder: 10 PDF files created
- Open sample PDF: Verify formatting and data accuracy
- Check email inbox: Verify delivery (if enabled)

### Single-Student Test

For quick testing:
```
1. Manually add 1 queue item in Orchestrator UI:
   - Queue: MarksheetQueue
   - Reference: 220701265
   - Add all 16 fields manually
2. Run Performer only
3. Verify single PDF generated
```

## 🧪 Testing & Validation

### Testing Workflow

**Unit Test: Dispatcher Only**
```powershell
# Terminal commands
cd "F:\UiPath\Final Project Dispatcher"
# Run from Studio (F5) and verify:
# - Output: "Rows read: 10"
# - Orchestrator: 10 queue items created
```

**Integration Test: Full Pipeline**
1. **Clear Queue:** Remove old items from Orchestrator
2. **Run Dispatcher:** Populate queue with fresh data
3. **Run Performer:** Process all items
4. **Validate Output:**
   - Queue status: All "Successful"
   - PDFs: 10 files in Data/Output/
   - File size: ~15-20 KB each
   - Email: Received (if configured)

**Error Testing**

Test invalid data handling:
```
1. Create test CSV with marks = 150 (invalid)
2. Run Dispatcher → Queue populated
3. Run Performer → Business exception logged
4. Check queue: Item marked "Failed"
5. Verify error message in Orchestrator logs
```

### Data Validation Test Cases

| Test Case | Expected Result |
|-----------|----------------|
| Valid data (marks 0-100) | Success, PDF generated |
| Marks > 100 | Business exception, queue item Failed |
| Marks < 0 | Business exception, queue item Failed |
| Missing email field | Business exception |
| Duplicate roll numbers | Multiple queue items (warning) |
| Empty CSV | Dispatcher completes, 0 queue items |

## 🔮 Future Enhancements

### Planned Improvements for Dispatcher

**1. Data Validation Layer**
- Add pre-validation before queue item creation
- Check marks range (0-100) in Dispatcher
- Validate email format
- Detect duplicate roll numbers
- **Benefit:** Prevent invalid items from entering queue

**2. Multi-File Support**
- Process multiple CSV files in batch
- Watch folder pattern for auto-processing
- Support for Excel (.xlsx) input files
- **Benefit:** Handle multiple departments/semesters

**3. Database Integration**
- Read from SQL Server / MySQL instead of CSV
- Real-time sync with student information system
- Support for incremental updates
- **Benefit:** Eliminate manual CSV exports

**4. Enhanced Logging**
- Detailed field-level validation logs
- Statistics: Total students, departments, semesters
- Duplicate detection report
- **Benefit:** Better audit trails

**5. Dynamic Queue Configuration**
- Read queue name from config file
- Support for multiple queues (by department/semester)
- Priority-based queue assignment
- **Benefit:** More flexible deployment

**6. Error Recovery**
- Automatic retry for transient Orchestrator connection issues
- Checkpoint/resume for large CSV files
- Partial success reporting
- **Benefit:** More robust operation

### System-Wide Enhancements

(See Performer README for additional enhancements including):
- Digital signatures on PDF marksheets
- QR code verification system
- Multi-language support
- Power BI analytics dashboard
- Parallel processing with multiple robots
- Watermarking and security features

## �📄 Project Metadata

- **Project Name**: Final Project Dispatcher
- **Project ID**: a4e9271e-d16c-49b7-bb2e-4b8547ef8d15
- **Version**: 1.0.5
- **Expression Language**: Visual Basic
- **Output Type**: Process
- **Profile**: Development

## 🤝 Contributing

When modifying this Dispatcher project:

**Data Integrity:**
1. Maintain the CSV column structure (17 columns)
2. Keep column names exactly as specified
3. Preserve data types (String vs Integer)

**Code Quality:**
4. Keep error handling for file validation
5. Preserve logging statements for debugging
6. Add comments for complex logic

**Testing:**
7. Test with sample data before production
8. Verify queue items created correctly
9. Test with Performer to ensure end-to-end compatibility

**Coordination:**
10. If changing queue structure, update Performer accordingly
11. Document any changes to data format
12. Maintain backward compatibility when possible

---

**Note**: This is a student project for academic purposes. Ensure proper testing and validation before deploying to production environments.


