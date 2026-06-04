# CSV-to-XML Mapping Training: Comprehensive Guide
## IBM Sterling B2B Integrator - Knowledge Transfer & Developer Training
**Author:** Training Team  
**Date:** March 2026  
**Duration:** 15 minutes (approximately 4,500 words including reference material)  
**Target Audience:** New IBM Sterling B2B Integrator Developers

---

## EXECUTIVE OVERVIEW

This training document provides a complete knowledge transfer for transforming CSV (Comma-Separated Values) files into XML (Extensible Markup Language) format using IBM Sterling B2B Integrator's Map Editor. 

### What You'll Learn:
- Understanding CSV and XML formats
- Building map structures in Sterling
- Field-to-field transformation logic
- XML schema hierarchy and validation
- Running and testing maps
- Interpreting results and error reports

### Training Deliverables:
1. **CSV_to_XML_Mapping.pptx** - Interactive presentation with 11 slides
2. **SPEAKER_NOTES.md** - Detailed narration script for each slide (this document)
3. **Training Project Files**:
   - Sample CSV input file
   - Generated XML output
   - Compiled map file (.mxl)
   - Validation reports

---

## MODULE 1: FOUNDATIONS

### Understanding CSV Format

**What is CSV?**
CSV stands for Comma-Separated Values. It's a simple text-based format where:
- Data is organized in rows and columns
- Columns are separated by delimiters (typically commas)
- Rows are separated by line breaks (CR/LF on Windows, LF on Unix)
- Fields are variable-length (no fixed column positions)
- It's human-readable and easy to process programmatically

**CSV Example:**
```csv
NAME,AMOUNT,INVOICE,DATE,CITY
shashank,1000,INV001,20260120,Vizianagaram
mahesh,400,INV002,20250506,Vizag
sai,3200,INV003,20260122,Srikakulam
```

**Key Characteristics:**
1. **Header Row**: First row contains column names (optional but common)
2. **Data Rows**: Subsequent rows contain actual data
3. **Field Delimiter**: Comma (,) – can be customized to |, ;, etc.
4. **Record Delimiter**: Newline character (CR/LF or LF)
5. **Quoting**: Fields with embedded delimiters can be quoted: "Smith, John"
6. **Variable Length**: Each field can be any width

### Understanding XML Format

**What is XML?**
XML stands for Extensible Markup Language. It's a hierarchical, structured format where:
- Data is wrapped in tags (elements)
- Tags come in pairs: opening and closing
- Elements can be nested inside other elements
- Data inside tags is called PCDATA (Plain Character Data)
- Structure is defined by a schema
- It's both human-readable and machine-parseable

**XML Example:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<OUTPUT>
  <DATA>
    <NAME>shashank</NAME>
    <AMOUNT>1000</AMOUNT>
    <INVOICE>INV001</INVOICE>
    <DATE>20260120</DATE>
    <CITY>Vizianagaram</CITY>
  </DATA>
  <DATA>
    <NAME>mahesh</NAME>
    ...
  </DATA>
</OUTPUT>
```

**Key Characteristics:**
1. **XML Declaration**: `<?xml version="1.0"?>` at the beginning
2. **Root Element**: One single root element that contains everything: `<OUTPUT>`
3. **Nested Elements**: Child elements inside parent elements
4. **PCDATA Content**: Text inside elements
5. **Tags**: Elements have opening `<TAG>` and closing `</TAG>` tags
6. **Attributes** (optional): Key-value pairs in the opening tag: `<name="value">`
7. **Well-Formed**: Strict syntax requirements (must close all tags, proper nesting)

### CSV vs. XML: Key Differences

| Aspect | CSV | XML |
|--------|-----|-----|
| **Structure** | Flat (rows/columns) | Hierarchical (nested) |
| **Delimiters** | Commas | Tags/Elements |
| **Readability** | Simple, spreadsheet-like | Self-documenting with tags |
| **Nesting** | Not supported | Full support |
| **Schema** | Optional | Can be strictly defined (XSD) |
| **Validation** | Limited | Comprehensive (XSD, DTD) |
| **Attributes** | Not supported | Supported in elements |
| **Scalability** | Limited for complex data | Excellent for complex hierarchies |
| **Use Cases** | Data exchange, reports | Web services, configuration, complex docs |

---

## MODULE 2: XML CONCEPTS IN CONTEXT

### Elements

An **element** is the fundamental building block of XML. It consists of:
- **Opening tag**: `<NAME>`
- **Content**: `shashank`
- **Closing tag**: `</NAME>`

Together: `<NAME>shashank</NAME>`

In our mapping:
```xml
<NAME>shashank</NAME>      <!-- NAME is the element name/tag -->
<AMOUNT>1000</AMOUNT>      <!-- AMOUNT is the element name/tag -->
<INVOICE>INV001</INVOICE>  <!-- INVOICE is the element name/tag -->
```

### PCDATA (Plain Character Data)

**PCDATA** is the actual text content inside an element:
- In `<NAME>shashank</NAME>`, the PCDATA is `shashank`
- In `<AMOUNT>1000</AMOUNT>`, the PCDATA is `1000`
- PCDATA is just text—no XML formatting, no tags

When Sterling transforms CSV to XML, it takes the field value (CSV column content) and wraps it as PCDATA inside an XML element.

**Important**: XML doesn't understand data types. Whether AMOUNT contains "1000" or "abc", it's stored as text. The data type is a logical interpretation, not enforced by XML itself.

### Content Particles: Sequence, Choice, All

**Content Particles** define how child elements can be arranged:

#### Sequence
Elements must appear in a specific order. In our schema:
```
OUTPUT contains: DATA (sequence)
DATA contains: NAME → AMOUNT → INVOICE → DATE → CITY (in order)
```

If we wanted to generate XML with elements out of order, the schema would reject it.

#### Choice
Only one of the specified elements can appear. Example:
```xml
<Payment>
  <CreditCard>...</CreditCard>  <!-- OR -->
  <BankTransfer>...</BankTransfer>
</Payment>
```

We could have EITHER CreditCard OR BankTransfer, but not both.

#### All
Elements can appear in any order, but only once each. Less common than sequence.

**Our Map Uses Sequence:**
Our DATA element uses a sequence particle, meaning the fields must appear in order: NAME, AMOUNT, INVOICE, DATE, CITY.

### Complex Types vs. Simple Types

**Simple Type**: Contains only PCDATA, no child elements.
```xml
<NAME>shashank</NAME>  <!-- Simple type: only text inside -->
```

**Complex Type**: Contains child elements.
```xml
<DATA>                 <!-- Complex type: has children -->
  <NAME>shashank</NAME>
  <AMOUNT>1000</AMOUNT>
  ...
</DATA>
```

In our schema:
- **OUTPUT** = Complex type (contains DATA elements)
- **DATA** = Complex type (contains NAME, AMOUNT, INVOICE, DATE, CITY)
- **NAME, AMOUNT, INVOICE, DATE, CITY** = Simple types (only PCDATA)

### Attributes

XML elements can have **attributes**—key-value pairs in the opening tag:
```xml
<DATA id="1" status="active">
  <NAME>shashank</NAME>
  ...
</DATA>
```

Here, `id` and `status` are attributes. Our current map doesn't use attributes—it only uses element content. But you could add attributes if needed.

---

## MODULE 3: MAP STRUCTURE & DESIGN

### INPUT Structure (CSV)

In Sterling's Map Editor, the input CSV is modeled as:

```
INPUT
  └─ DATA (C-9999)
      ├─ NAME (String, 0-10)
      ├─ AMOUNT (String, 0-10)
      ├─ INVOICE (String, 0-10)
      ├─ DATE (String, 0-10)
      └─ CITY (String, 0-20)
```

**Explanation:**
- **INPUT**: Root group (mandatory, M-1)
- **DATA**: Repeating record (conditional, C-9999)
  - Conditional means the file can be empty
  - 0-9999 means it can repeat from 0 to 9999 times
- **NAME, AMOUNT, etc.**: Fields within the DATA record
  - All String type (text fields)
  - Max lengths: 10, 10, 10, 10, 20 characters

### OUTPUT Structure (XML)

In Sterling's Map Editor, the output XML is modeled as:

```
OUTPUT
  └─ OUTPUT (M-1)
      └─ DATA (C-9999)
          ├─ NAME (C-0-1) [PCDATA String]
          ├─ AMOUNT (C-0-1) [PCDATA String]
          ├─ INVOICE (C-0-1) [PCDATA String]
          ├─ DATE (C-0-1) [PCDATA String]
          └─ CITY (C-0-1) [PCDATA String]
```

**Explanation:**
- **OUTPUT**: Root XML element (mandatory, M-1) – must have exactly one
- **DATA**: Repeating element inside OUTPUT (conditional, C-9999)
  - For each input DATA occurrence, one output DATA element is generated
- **NAME, AMOUNT, etc.**: Child elements of DATA
  - All conditional (C), 0-1 cardinality (optional, at most one per DATA)
  - All contain PCDATA strings

### Cardinality Notation

In Sterling, cardinality is expressed as **M/C** with **min-max** numbers:

| Notation | Meaning |
|----------|---------|
| M-1 | Mandatory, exactly 1 (must exist, appears once) |
| C-1 | Conditional, at most 1 (optional, appears 0 or 1 times) |
| C-9999 | Conditional, up to 9999 (optional, repeats 0-9999 times) |
| M-999 | Mandatory, at least 1 but up to 999 times |

**M vs. C:**
- **M (Mandatory)**: The element must be present. If Sterling can't find it, it fails.
- **C (Conditional)**: The element is optional. It may or may not appear.

In our map, DATA is C-9999 because:
- C: The file might be empty (no data rows)
- 9999: It can repeat up to 9999 times to handle large files

---

## MODULE 4: FIELD MAPPING

### 1:1 Direct Mapping

Our map uses the simplest type of mapping: direct field-to-field copying.

```
INPUT CSV Field  →  Transformation  →  OUTPUT XML Element
NAME             →  Copy as-is     →  <NAME>value</NAME>
AMOUNT           →  Copy as-is     →  <AMOUNT>value</AMOUNT>
INVOICE          →  Copy as-is     →  <INVOICE>value</INVOICE>
DATE             →  Copy as-is     →  <DATE>value</DATE>
CITY             →  Copy as-is     →  <CITY>value</CITY>
```

In the Map Editor, we drag a connection line from each input field to the corresponding output element. Sterling then knows: "Copy the input field value into the output element."

### Repeating Group Handling

The key feature that makes this work for multiple records is **loop handling**:

**Processing Flow:**
1. Sterling reads the CSV header row
2. Sterling recognizes the DATA record is repeating (C-9999)
3. Sterling reads the first data row → populates DATA occurrence #1
4. For each input DATA occurrence, Sterling generates one output DATA element
5. Sterling reads the second data row → populates DATA occurrence #2 → generates DATA element #2
6. Sterling continues until the CSV file ends or reaches max loop count

**Example with 3 CSV Rows:**
```
CSV Input:                          Generated XML Output:
─────────────────────────────       ──────────────────────────────────
Row 1: shashank,1000,...     →      <DATA><NAME>shashank</NAME>...
Row 2: mahesh,400,...        →      <DATA><NAME>mahesh</NAME>...
Row 3: sai,3200,...          →      <DATA><NAME>sai</NAME>...
```

This loop-per-loop correspondence is automatic when you set up repeating groups properly.

### Advanced Transformations (Beyond Our Map)

Sterling is capable of more complex transformations, though our simple map doesn't use them:

1. **Concatenation**: Combine two fields
   - Input: FNAME="John", LNAME="Smith"
   - Output: FULLNAME="John Smith"

2. **Substring Extraction**: Get part of a field
   - Input: PHONE="1-555-123-4567"
   - Output: AREACODE="555"

3. **Lookups**: Reference external tables
   - Input: COUNTRY_CODE="US"
   - Output: COUNTRY_NAME="United States" (from lookup table)

4. **Conditional Logic**: Map different fields based on conditions
   - If TRANS_TYPE="CREDIT" → add to CREDIT element
   - If TRANS_TYPE="DEBIT" → add to DEBIT element

5. **Aggregation**: Sum or count values
   - Count: How many transactions per customer?
   - Sum: Total amount per customer?

6. **Constants**: Insert fixed values
   - MAPPING_VERSION="1.0" (same for every output record)
   - PROCESSING_DATE="2026-03-15" (current date)

Our map is foundational knowledge. These advanced techniques build on the same principles.

---

## MODULE 5: COMPILATION & DEPLOYMENT

### Compilation Process

**Step 1: Save the Map Source**
```
File → Save (or Ctrl+S)
Output: CSV_XML_SSY.mxl
```
The .mxl file is XML-formatted source code containing your complete map definition. Store it in version control (Git, SVN, etc.).

**Step 2: Validate**
```
File → Compile (or F8)
```
The Map Editor validates:
- Structure integrity (proper nesting, no orphans)
- Field connectivity (all mapped fields have sources)
- Cardinality consistency (no conflicting constraints)
- Logical correctness (no circular loops or contradictions)

If validation fails, error messages appear. Fix them and try again.

**Step 3: Compile to Translation Object**
```
Compilation succeeds → Generates .txo file
Location: Compiled Maps folder
Output: CSV_XML_SSY.txo (binary format)
```

The .txo is a compiled binary format optimized for the Sterling server. It contains:
- The complete map definition
- Field types and constraints
- Mapping rules and transformations
- Any extended rules or conditions

**Step 4: Deploy to Server**
```
Upload CSV_XML_SSY.txo to Sterling B2B Integrator server
Via: Web interface, SFTP, API, etc.
Status: Map is now available for use on the server
```

### File Artifacts

| File | Format | Purpose | Audience |
|------|--------|---------|----------|
| CSV_XML_SSY.mxl | XML (text) | Source code for the map | Developers |
| CSV_XML_SSY.txo | Binary | Compiled, server-executable object | Sterling server |
| MapTestTxReport.xml | XML (text) | Compilation/validation report | QA, Developers |

---

## MODULE 6: TESTING & EXECUTION

### Map Test Dialog

The Map Test feature (File → Map Test) lets you test your map by running it on the Sterling server directly from the Map Editor.

**Dialog Fields:**

1. **Server:Port**
   - Hostname/IP and dashboard port of Sterling B2B Integrator
   - Example: `192.168.128.157:52033`
   - Typically port 52033 for the dashboard

2. **Username & Password**
   - Sterling account credentials
   - Account must have map execution permissions
   - Password sent securely over HTTPS

3. **Translation Object Path**
   - Full path to the compiled .txo file
   - Example: `C:\...\Compiled Maps\CSV_XML_SSY.txo`

4. **Data File Path**
   - Full path to the input CSV file
   - Example: `C:\Users\...\sample.txt`

5. **Run Test Button**
   - Sends .txo and CSV to server for execution
   - Waits for result and displays output

### Server-Side Execution

When you click Run Test:

1. **Network Request**
   - Map Editor sends HTTP POST to server
   - Multipart request containing .txo (binary) and CSV (text)
   - User credentials sent for authentication

2. **Server Processing**
   - Authentication: Validates username/password
   - Loading: Loads .txo into translation engine
   - Parsing: Parses CSV according to input definition
   - Processing: Loops through records, applies mappings
   - Validation: Checks constraints, detects errors
   - Generation: Builds XML output

3. **Response**
   - HTTP 200 OK (success) or error status
   - Two files returned:
     - **MapTestTxResult.xml**: The transformed output
     - **MapTestTxReport.xml**: Validation report

4. **Map Editor Display**
   - Receives response
   - Displays XML output in results pane
   - Displays any errors/warnings in report pane

---

## MODULE 7: RESULTS & INTERPRETATION

### Expected Output

**Input CSV:**
```csv
NAME,AMOUNT,INVOICE,DATE,CITY
shashank,1000,INV001,20260120,Vizianagaram
mahesh,400,INV002,20250506,Vizag
sai,3200,INV003,20260122,Srikakulam
```

**Generated XML Output:**
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes" ?>
<OUTPUT>
  <DATA>
    <NAME>shashank</NAME>
    <AMOUNT>1000</AMOUNT>
    <INVOICE>INV001</INVOICE>
    <DATE>20260120</DATE>
    <CITY>Vizianagaram</CITY>
  </DATA>
  <DATA>
    <NAME>mahesh</NAME>
    <AMOUNT>400</AMOUNT>
    <INVOICE>INV002</INVOICE>
    <DATE>20250506</DATE>
    <CITY>Vizag</CITY>
  </DATA>
  <DATA>
    <NAME>sai</NAME>
    <AMOUNT>3200</AMOUNT>
    <INVOICE>INV003</INVOICE>
    <DATE>20260122</DATE>
    <CITY>Srikakulam</CITY>
  </DATA>
</OUTPUT>
```

**Observations:**
- XML declaration at top with encoding
- Single root element OUTPUT
- 3 DATA elements (one per CSV row)
- Each DATA element has 5 children: NAME, AMOUNT, INVOICE, DATE, CITY
- Each child element contains PCDATA (the field value)
- Proper nesting and indentation
- No errors or corruption

### Validation Report

**Clean Report (Success):**
```xml
<TranslationReport>
<![CDATA[Translation report contains no data.]]>
</TranslationReport>
```

"Contains no data" = no errors. Clean translation.

**Report with Errors (Example):**
```xml
<TranslationReport>
<![CDATA[
ERROR: Mandatory field NAME is missing in record 2
WARNING: Field AMOUNT contains non-numeric data "abc" in record 3
ERROR: Unexpected character in CITY field at record 1
]]>
</TranslationReport>
```

### Troubleshooting Guide

| Symptom | Possible Cause | Solution |
|---------|----------------|----------|
| Connection refused | Server is down or wrong port | Check server IP/port, ensure Sterling is running |
| Authentication failed | Wrong credentials | Verify username/password |
| No output generated | Empty input file | Ensure CSV has data rows |
| XML malformed | Mapping error | Check field mappings, cardinality |
| Mandatory field missing | Optional field not populated | Mark field as optional in schema or populate it |
| Data type mismatch | String contains invalid data | Validate input data, add transformation rules |

---

## MODULE 8: COMPLETE WORKFLOW SUMMARY

### End-to-End Data Flow

```
1. CSV Input File
   ↓ (created by source system)
   
2. Map Editor
   ↓ (design map: define input/output structures, create mappings)
   
3. Compile to .txo
   ↓ (save source as .mxl, compile to binary .txo)
   
4. Sterling B2B Integrator Server
   ↓ (uploaded .txo via web interface or API)
   
5. Map Test (or runtime execution)
   ↓ (send CSV + .txo to server)
   
6. Server Translation Engine
   ├─ Parse CSV (read header, recognize delimiter)
   ├─ Loop through records
   ├─ Apply mappings (copy fields to elements)
   ├─ Generate XML (nest elements, add tags)
   └─ Validate (check constraints, detect errors)
   ↓
   
7. Generate Output
   ├─ XML Result File (MapTestTxResult.xml)
   └─ Validation Report (MapTestTxReport.xml)
   ↓
   
8. Return to Map Editor
   ↓ (via HTTP response)
   
9. Results Display
   ├─ View transformed XML
   ├─ View validation report
   └─ Verify success or debug errors
```

### Key Learning Points

1. **CSV Structure**: Flat, delimited format with variable-length fields
2. **XML Structure**: Hierarchical, tagged format with nested elements and PCDATA
3. **Map Design**: INPUT groups with repeating records map to OUTPUT elements with repeating nesting
4. **1:1 Mapping**: Simple field-to-field copying is the foundation; advanced transformations build on it
5. **Loop Handling**: Repeating groups automatically handle multiple records with loop-per-loop correlation
6. **Compilation**: .mxl source → .txo binary for server execution
7. **Testing**: Map Test dialog sends data to server and returns results
8. **Validation**: Reports indicate success or list errors for debugging

---

## MODULE 9: QUICK REFERENCE

### Sterling Map Editor Keyboard Shortcuts

| Action | Shortcut |
|--------|----------|
| Save Map | Ctrl+S |
| Compile Map | F8 |
| Map Test | Ctrl+T |
| New Sub Record | Right-click → New → Sub Record |
| New Field | Right-click → New → Field |
| Undo | Ctrl+Z |
| Redo | Ctrl+Y |

### Common Sterling Properties

**For INPUT Records:**
- Delimiter: , (comma)
- Quote Character: " (double-quote)
- Include Column Names: Yes/No
- Quote Handling: Standard, Optimized, etc.

**For OUTPUT Elements:**
- Element Name: (matching tag name)
- PCDATA Handling: Trim, Include Whitespace, etc.
- Encoding: UTF-8, ISO-8859-1, etc.

### XML Syntax Rules

1. **Every tag must close**: `<NAME>data</NAME>` (not `<NAME>data`)
2. **Proper nesting**: `<A><B></B></A>` (not `<A><B></A></B>`)
3. **Root element required**: One root element containing everything
4. **Attribute quotes**: `attr="value"` (double or single quotes required)
5. **Special characters**: Use entities: `&lt;` for `<`, `&amp;` for `&`, `&quot;` for `"`
6. **Case-sensitive**: `<Name>` ≠ `<name>`

---

## MODULE 10: HANDS-ON EXERCISES (FOR LEARNERS)

### Exercise 1: Analyze the Mapping
Look at the CSV input and XML output provided. Trace each value from input to output.
- Where does "shashank" go?
- How many DATA elements are generated?
- What's the root element?

**Solution:** Each CSV row becomes a DATA element. "shashank" from the first NAME field becomes `<NAME>shashank</NAME>` in the first DATA element.

### Exercise 2: Modify the Map
Add a new field to the CSV and XML:
- CSV: Add "STATUS" field (6th column)
- XML: Add `<STATUS>` element to DATA
- Update the map to include this field

**Steps:**
1. Edit INPUT.DATA record: add STATUS field
2. Edit OUTPUT.DATA element: add STATUS child element
3. Draw mapping line from INPUT STATUS to OUTPUT STATUS
4. Recompile

### Exercise 3: Test with Different Data
Create a new CSV file with 5 records instead of 3, and test the map.
- Does it generate 5 DATA elements?
- Are all fields populated correctly?
- Does the report stay clean?

**Expected Result:** 5 DATA elements in output, no errors.

---

## CONCLUSION & NEXT STEPS

Congratulations! You now understand:
✓ How CSV and XML differ structurally  
✓ XML concepts: elements, PCDATA, sequences, complex/simple types  
✓ How to build input and output structures in Sterling  
✓ How field mappings connect CSV to XML  
✓ How loops handle repeating records  
✓ How to compile maps and deploy them  
✓ How to test maps and interpret results  

### Next Steps:

1. **Practice**: Build your own simple CSV-to-XML map
2. **Advance**: Learn complex transformations (concatenation, lookups, conditions)
3. **Optimize**: Learn performance tuning for large files
4. **Integrate**: Connect maps to business processes in Sterling
5. **Master**: Handle EDI formats, web services, database transformations

---

## APPENDIX: GLOSSARY

**Attribute**: Key-value pair in XML element opening tag  
**CDATA**: Character Data section in XML for preserving text as-is  
**Cardinality**: Min/max occurrence count for an element or record  
**Complex Type**: XML element containing child elements  
**CSV**: Comma-Separated Values format  
**Delimiter**: Character separating fields (comma, pipe, semicolon, etc.)  
**Element**: XML building block with opening tag, content, closing tag  
**Map Editor**: Sterling B2B Integrator's visual mapping tool  
**Mapping**: Connection between input field and output field  
**M/C**: Mandatory/Conditional notation in Sterling  
**Multipart Request**: HTTP request with multiple sections (e.g., file + metadata)  
**PCDATA**: Plain Character Data inside XML elements  
**Schema**: Structural definition of valid XML (XSD format)  
**Simple Type**: XML element containing only PCDATA  
**Translation Object**: Compiled .txo file executable on Sterling server  
**Transformation**: Process of converting input to output format  
**XML**: Extensible Markup Language hierarchical format  
**XSD**: XML Schema Definition file defining XML structure  

---

## REFERENCE FILES

- **CSV_to_XML_Mapping.pptx**: Training presentation (11 slides)
- **SPEAKER_NOTES.md**: Detailed narration script
- **sample.txt**: Input CSV file example
- **MapTestTxResult.xml**: Generated XML output example
- **MapTestTxReport.xml**: Validation report example
- **CSV_to_XML.PNG**: Mapping screenshot from Sterling Map Editor
- **CSV_XML_SSY.mxl**: Source map file

---

**Training Complete! You're ready to build CSV-to-XML maps in IBM Sterling B2B Integrator.**

For questions, refer to the speaker notes or consult the Sterling B2B Integrator documentation.

