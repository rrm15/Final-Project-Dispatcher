# Student Marksheet Dispatcher

A UiPath automation project that reads student data from CSV files and populates an Orchestrator queue for processing. This is the **Dispatcher** component of a Dispatcher-Performer architecture for automated student marksheet generation.

## 📋 Overview

This project automates the first step in the marksheet generation workflow:
1. Reads student academic data from a CSV file
2. Validates the presence of the input file
3. Creates queue items in UiPath Orchestrator's `MarksheetQueue`
4. Uses student Roll Number as unique reference identifier
5. Logs processing status and results

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

| Column | Description | Type | Example |
|--------|-------------|------|---------|
| Name | Student's full name | String | Sharukeshwar P |
| RollNumber | Unique student identifier | String | 220701265 |
| DOB | Date of birth | String | 15-Jan-2004 |
| Semester | Academic semester | String | Fall 2024 |
| Year | Academic year | String | 2024 |
| Department | Department name | String | Computer Science |
| Subject1 - Subject5 | Subject names | String | Data Structures, Database Management, etc. |
| Mark1 - Mark5 | Corresponding marks | Integer | 95, 88, 82, 75, 92 |
| Email | Student email address | String | 220701265@rajalakshmi.edu.in |

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

## 🔗 Dispatcher-Performer Architecture

This project is the **Dispatcher** component. It works in conjunction with a **Performer** component (separate project) that:

1. Retrieves queue items from `MarksheetQueue`
2. Processes each student's data
3. Generates PDF marksheets
4. Updates queue item status (Success/Failed)
5. Sends email notifications (if configured)

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

### Common Issues

**Issue**: "CSV not found" exception
- **Solution**: Verify the CSV file exists at `Data\Input\Sample_Student_Data.csv`
- Check file path is correct (relative to project root)

**Issue**: Queue items not appearing in Orchestrator
- **Solution**: 
  - Confirm queue `MarksheetQueue` exists in Orchestrator
  - Check Orchestrator connection in Studio (Project → Settings → Orchestrator)
  - Verify folder path and permissions

**Issue**: Data type conversion errors
- **Solution**: Ensure Mark1-5 columns contain valid integer values
- Check CSV format matches expected structure (no missing columns)

## 📄 Project Metadata

- **Project Name**: Final Project Dispatcher
- **Project ID**: a4e9271e-d16c-49b7-bb2e-4b8547ef8d15
- **Version**: 1.0.5
- **Expression Language**: Visual Basic
- **Output Type**: Process
- **Profile**: Development

## 🤝 Contributing

When modifying this project:
1. Maintain the CSV column structure
2. Keep error handling for file validation
3. Preserve logging statements for debugging
4. Test with sample data before production use


