# CSV-to-XML Mapping Training: Speaker Notes
## IBM Sterling B2B Integrator - Knowledge Transfer Document
### Estimated Duration: 15 minutes (approximately 2,000 words)

---

## SLIDE 1: Title Slide
**"IBM STERLING B2B INTEGRATOR - CSV to XML Mapping"**

### Speaker Notes:

Good morning, everyone! Welcome to this training session on CSV-to-XML mapping in IBM Sterling B2B Integrator. My name is [Your Name], and today I'll be walking you through the complete process of transforming comma-separated values from a CSV file into a well-structured XML document.

Before we dive in, let me set the context. CSV and XML are two completely different data formats. CSV is flat and delimited—think of it as rows and columns in a spreadsheet. XML, on the other hand, is hierarchical and structured—with nested elements, making it ideal for complex business documents, web services, and data interchange scenarios. Understanding how to map between these two formats is a critical skill for any IBM Sterling B2B Integrator developer.

In this session, we'll be working with a real-world example called CSV_XML_SSY.mxl. This map takes a simple CSV file with 5 fields—NAME, AMOUNT, INVOICE, DATE, and CITY—and transforms each row into an XML element structure. We'll handle multiple CSV records by creating a repeating XML group that can process up to 9,999 rows.

Our agenda today covers 10 key topics: the CSV input structure, the XML output format, schema hierarchy, XML concepts, building the map, field-to-field mapping logic, compiling and deploying, running the map, understanding execution flow, and finally, interpreting results.

By the end of this training, you'll understand not just what happens, but why it happens, and how to apply these principles to your own CSV-to-XML mappings.

Let's get started!

---

## SLIDE 2: The CSV Input File
**"1. The CSV Input File"**

### Speaker Notes:

Now, let's look at our source data—the CSV input file. CSV stands for Comma-Separated Values, and as the name suggests, it's a simple text format where fields are separated by commas and records are separated by line breaks.

Here's what our input file looks like:

On line 1, we have the header: NAME, AMOUNT, INVOICE, DATE, CITY. This is a common pattern in CSV files—the first row contains column names that describe the data in each field. Sterling's CSV parser recognizes this because we've enabled the "Include Column Names" flag in the map.

Then we have the data rows. Row 2 has shashank, 1000, INV001, 20260120, Vizianagaram. Row 3 is mahesh, 400, INV002, 20250506, Vizag. And row 4 is sai, 3200, INV003, 20260122, Srikakulam. That's 3 employee or customer records, each with 5 fields.

Notice several key characteristics:

First, **field delimiter**: the comma. Sterling uses commas to know where one field ends and the next begins. If your CSV uses pipes or semicolons instead, you can configure that in the map's input properties.

Second, **record delimiter**: the newline character—CR/LF in Windows or LF on Unix. Sterling uses newlines to know when one record ends and the next begins.

Third, **variable length fields**: each field can be any width. NAME can be "shashank" (8 characters) or "sai" (3 characters). There's no fixed column position like in positional files. This is a key difference from the positional format we saw in the reference training.

Fourth, **optional quote character**: if a field contains a comma, it can be wrapped in quotes to escape the delimiter. For example, if NAME was "Smith, John", it would be written as "Smith, John" with quotes to prevent Sterling from treating the internal comma as a field separator.

In the Sterling map, we model this CSV input as a variable-delimited record type. The INPUT group contains a repeating "DATA" record with a cardinality of 0 to 9999. That means we can process anywhere from zero records up to 9999 rows, and Sterling will loop through each one.

---

## SLIDE 3: The XML Output File
**"2. The XML Output File"**

### Speaker Notes:

Now let's look at our target—the XML output. XML stands for Extensible Markup Language, and it's a fundamentally different beast from CSV. Instead of flat, comma-delimited rows, XML is hierarchical and tag-based.

Here's the XML output we're generating:

We start with the XML declaration: `<?xml version="1.0" encoding="UTF-8"?>`. This tells any XML parser that we're using XML version 1.0 in UTF-8 character encoding.

Next is the root element: `<OUTPUT>`. In XML, there's always one root element that contains everything else. We can't have multiple root elements—that would be invalid XML. The name OUTPUT is arbitrary; you could call it TransactionData or Records, but in this case, we've named it OUTPUT.

Inside OUTPUT, we have repeating `<DATA>` elements. For each CSV row we process, we generate one DATA element. So our 3 CSV data rows produce 3 DATA elements.

Inside each DATA element, we have 5 child elements: NAME, AMOUNT, INVOICE, DATE, and CITY. Each of these holds text content—what we call PCDATA, or Plain Character Data. For example, the first NAME element contains the text "shashank"—that's PCDATA.

Let me show you the structure more clearly:

```
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
  ...
</OUTPUT>
```

Notice the indentation and nesting. This hierarchy is what makes XML powerful. The DATA element contains NAME, not the other way around. This nesting represents the business logic: we're grouping all fields belonging to one record inside a DATA element.

Also notice that every opening tag has a closing tag. `<NAME>` must be closed with `</NAME>`. This strict structure is what allows XML parsers to reliably read and validate the data, even if it's huge.

The beauty of this structure is that it scales. You could have hundreds or thousands of DATA elements, and the parser would still understand the structure perfectly. With CSV, if you had a file with thousands of rows, it would just be thousands of lines with no inherent grouping or context.

---

## SLIDE 4: XML Schema Hierarchy
**"3. XML Schema Hierarchy"**

### Speaker Notes:

Now we need to understand the XML schema—the blueprint that defines what's allowed in our XML output. Think of it like a template or contract: it specifies what elements can exist, how many times they can repeat, in what order, and what type of data they contain.

Let's look at our schema structure:

**Root Element: OUTPUT**
- Mandatory (M)
- Cardinality 1-1: exactly one OUTPUT element must exist

**DATA Element**
- Conditional (C)
- Cardinality 0-9999: can have zero DATA elements (if the CSV is empty), or up to 9999 DATA elements

**Child Elements under DATA: NAME, AMOUNT, INVOICE, DATE, CITY**
- All Conditional (C)
- Cardinality 0-1: each child element is optional and appears at most once per DATA record

Let me explain this notation because it's important. In Sterling, we use M/C to indicate if something is mandatory or conditional:
- M = Mandatory. The element must be present.
- C = Conditional. The element is optional—it may or may not appear.

Then we have cardinality, shown as two numbers:
- First number = Minimum occurrences
- Second number = Maximum occurrences

So "C 0-1" means: optional, and can appear zero or one time.
"C 0-9999" means: optional, and can appear zero to 9999 times.

If we had "M 1-1", it would mean: mandatory, exactly one time.

Why is DATA conditional instead of mandatory? Because if the CSV file has no data rows—only a header or empty file—we still need valid XML. An empty OUTPUT with zero DATA elements is still valid. If we made DATA mandatory, we'd have to fail if there were no records, which isn't what we want.

This schema is defined in an XML Schema Definition, or XSD file, which Sterling reads and uses to validate the output. As developers, we don't usually write the XSD ourselves; Sterling infers it from the map structure we create. But it's good to understand what it specifies.

---

## SLIDE 5: XML Concepts – Elements and PCDATA
**"4. XML Concepts: Elements & PCDATA"**

### Speaker Notes:

Now let's demystify XML by breaking down its core concepts, using our actual mapping as examples.

**XML Element:**
An element is a building block of XML. It has an opening tag, content, and a closing tag. For example:

```xml
<NAME>shashank</NAME>
```

Here, NAME is the element name (also called the tag name). The entire thing—opening tag, content, and closing tag—is called an element.

Elements can have different types of content. Some contain text, some contain other elements, and some contain both. In our case, all elements contain text.

**PCDATA (Plain Character Data):**
PCDATA is the actual text inside an element. In `<NAME>shashank</NAME>`, the PCDATA is "shashank". It's plain text—no XML tags, no formatting, just characters.

When Sterling processes our CSV row "shashank,1000,INV001,20260120,Vizianagaram", it extracts each value and wraps it in the appropriate XML element:

```
shashank      → <NAME>shashank</NAME>
1000           → <AMOUNT>1000</AMOUNT>
INV001         → <INVOICE>INV001</INVOICE>
20260120       → <DATE>20260120</DATE>
Vizianagaram   → <CITY>Vizianagaram</CITY>
```

Notice that the PCDATA doesn't have any special formatting. The date "20260120" remains as-is—it's just text. XML doesn't inherently understand dates or numbers; it treats all PCDATA as strings. If you want Sterling to interpret a field as an integer or date for validation or formatting, you specify that in the field definition, but in the XML output, it's still just text.

**Content Particles – Sequence:**
XML also has a concept of "particles," which define how child elements can be arranged. Our DATA element uses a "sequence" particle, which means all child elements must appear in the specified order:

NAME → AMOUNT → INVOICE → DATE → CITY

They don't all have to be present—they're optional—but if they are present, they must be in this sequence. This is enforced by the schema.

There are other particle types. A "choice" particle would mean "pick one of these elements." But we use sequence in our map, which is the most common.

**Complex Types and Simple Types:**
The DATA element is a complex type because it contains child elements. NAME, AMOUNT, INVOICE, DATE, and CITY are simple types because they contain only PCDATA, not child elements.

This distinction matters when we're validating data. A simple type element can only have PCDATA. A complex type element can have child elements. Sterling uses this distinction to ensure the output is valid.

---

## SLIDE 6: Building the Map – Records and Fields
**"5. Creating Records and Fields"**

### Speaker Notes:

Now let's talk about how to actually construct a map in Sterling. We're going to build the INPUT and OUTPUT structures that define our transformation.

**On the INPUT side:**
We have one repeating record called "DATA". The cardinality is C-9999, meaning it's conditional (optional) and can repeat up to 9999 times. This record has 5 fields: NAME, AMOUNT, INVOICE, DATE, and CITY. All are string type, with variable length (no fixed column positions, because CSV doesn't have those).

When Sterling reads the CSV, it sees the header row, identifies the column names, and then parses each data row into the DATA record. So the first CSV data row populates DATA occurrence #1, the second row populates DATA occurrence #2, and so on.

**On the OUTPUT side:**
We have a root element called OUTPUT with cardinality M-1: mandatory, exactly one. This is the XML root element. Inside OUTPUT, we have another repeating element called DATA with cardinality C-9999. For each input DATA occurrence, Sterling generates one output DATA element.

Inside each output DATA, we have 5 child elements: NAME, AMOUNT, INVOICE, DATE, CITY. All are optional (C) and appear at most once per DATA element (0-1 cardinality). All contain PCDATA strings.

Notice the naming similarity: both INPUT and OUTPUT have a "DATA" record/element. They're connected in the map. Sterling reads from INPUT DATA and writes to OUTPUT DATA, looping for each occurrence.

**How to Build It:**

Step 1: Create a Record. In the Sterling Map Editor, you right-click on INPUT (or OUTPUT) and select New → Sub Record. You name it and set whether it's mandatory or conditional. For input, you typically make it conditional because the file might be empty. For output, the root element is usually mandatory.

Step 2: Set Cardinality. When you create a record, you specify minimum and maximum usage. For detail records that repeat, you'd set Max to 999 or 9999. For single headers, you'd set it to 1.

Step 3: Add Fields. You right-click the record and select New → Field. You enter the field name, data type (String, Integer, etc.), and length. For CSV input, the length is the maximum width of that field in the CSV. For XML output elements, the length just describes the maximum PCDATA width.

Step 4: Set Positions (for positional output only). If you're generating a positional file like in the reference training, you'd specify the column position and padding for each field. For XML, we don't need positions—XML isn't positional. The order is defined by the schema sequence, not by column positions.

Step 5: Link Fields. This is where the magic happens. You drag a line from an INPUT field to an OUTPUT element. This creates the mapping connection. Sterling then knows: "Take the value from INPUT.DATA.NAME and put it in OUTPUT.DATA.NAME." The same applies to all other fields.

---

## SLIDE 7: Field-to-Field Mapping Logic
**"6. Field-to-Field Mapping Logic"**

### Speaker Notes:

Now let's look at the actual mapping—how fields are connected between CSV input and XML output.

In this map, we have a simple 1:1 mapping. Each input field is connected to exactly one output element, and vice versa.

```
CSV Input              →  XML Output
NAME (field)          →  <NAME> element
AMOUNT (field)        →  <AMOUNT> element
INVOICE (field)       →  <INVOICE> element
DATE (field)          →  <DATE> element
CITY (field)          →  <CITY> element
```

When Sterling processes a CSV row, it reads the five input fields and assigns their values to the five output elements. That's the basic mapping logic.

But there's more happening behind the scenes: **loop handling**.

The INPUT DATA record is set to loop C-9999. That means Sterling will read data rows from the CSV repeatedly, up to 9999 times or until the file ends. For each input DATA occurrence, it automatically loops the output DATA element.

So the flow is:
1. Read the first CSV data row → populate INPUT.DATA occurrence 1
2. Process the mapping → generate OUTPUT.DATA occurrence 1 with NAME, AMOUNT, etc.
3. Read the second CSV data row → populate INPUT.DATA occurrence 2
4. Process the mapping again → generate OUTPUT.DATA occurrence 2
5. Continue until no more CSV rows

In our example with 3 CSV data rows, this produces 3 OUTPUT DATA elements in the XML.

**Important detail: Simple vs. Complex Mapping**

Our example is a simple field-to-field mapping. But Sterling is capable of much more complex logic:
- **Concatenation**: Combine two input fields into one output field
- **Substring**: Extract part of an input field
- **Lookups**: Reference external tables to transform values
- **Conditions**: Use IF/THEN logic to map different fields based on conditions
- **Aggregation**: Sum or count values across records
- **Constants**: Insert fixed values that don't come from input

Our map doesn't use any of these advanced features—it's straightforward field copying. But knowing what's possible helps when you encounter more complex requirements.

---

## SLIDE 8: Saving and Compiling the Map
**"7. Saving and Compiling the Map"**

### Speaker Notes:

Once we've designed our map in the Sterling Map Editor, we need to save it and compile it for execution. Let me walk you through the complete process.

**Step 1: Save the Map**
We go to File → Save, or press Ctrl+S. This saves the map source file as CSV_XML_SSY.mxl. The .mxl extension stands for Map Exchange Language. This is an XML file containing our complete map definition—the records, fields, cardinality, properties, and all the connection lines.

We should keep this .mxl file in version control. It's our source code. If we need to modify the map later, we open the .mxl file, make changes, and save it again.

**Step 2: Validate**
We go to File → Compile, or press F8. The Map Editor performs a validation check. It verifies that:
- The INPUT and OUTPUT structures are well-formed
- Every connected field has matching source and target
- There are no orphaned fields (fields with no mapping)
- Cardinalities are correctly set
- No circular references or logical errors exist

If there are errors, they're reported in an output window. We have to fix them before proceeding.

**Step 3: Compile**
If validation passes, the compilation continues. Sterling translates the .mxl into a binary translation object called a .txo file. This .txo file is optimized for the Sterling B2B Integrator server. It contains the compiled map logic, field definitions, and all the mapping rules in a format the server can quickly load and execute.

The .txo file is generated in a "Compiled Maps" folder within the Map Editor installation directory.

**Step 4: Deploy**
To use the compiled map, we upload the .txo file to the Sterling B2B Integrator server. We typically use the Sterling web interface or administration tools to do this. Once uploaded, the map is available for execution on the server.

**File Artifacts:**
- .mxl: Editable map source file (keep in version control)
- .txo: Compiled translation object (uploaded to server, executed at runtime)
- .xml: Map validation report (generated during compilation)

The .xml report includes details like which fields are used or unused, any warnings or info messages, and a summary of the compilation process. A clean report indicates a successful compilation with no issues.

---

## SLIDE 9: Running the Map – Map Test Dialog
**"8. Running the Map – Map Test Dialog"**

### Speaker Notes:

Now that we've compiled the map, how do we test it? Sterling provides a Map Test feature that lets us run the map directly from the Map Editor without having to manually upload files to the server.

Here's how it works:

We go to File → Map Test. A dialog box appears with several fields to fill in.

**Server:Port**
The hostname or IP address of the Sterling B2B Integrator server, followed by a colon and the dashboard port number. For example, "192.168.128.157:52033". The dashboard port is typically 52033 or similar. This tells the Map Editor where to send the map execution request.

**Username**
The Sterling account username. This user account must have sufficient permissions to execute maps on the server. In our example, it's "EDINOV_YAMINI". The username is used for authentication.

**Password**
The account password. This is sent securely over HTTPS to the server. If authentication fails, the server rejects the request.

**Translation Object**
The path to our compiled .txo file. For example, "C:\...\Compiled Maps\CSV_XML_SSY.txo". The Map Editor needs to know which .txo to send to the server.

**Data File**
The path to our input CSV file. For example, "C:\Users\...\sample.txt". This is the actual CSV data we want to transform.

**Run Test Button**
Once we've filled in all these fields, we click Run Test. The Map Editor packages up the .txo and CSV input file and sends them to the Sterling server as an HTTP POST request.

**Behind the Scenes**

The Map Editor connects to the server using the credentials provided. It sends the .txo and data file. The server receives the request, authenticates the user, loads the .txo into its translation engine, reads the CSV input using the map definition, applies the transformation logic, and generates the XML output. It also produces a translation report documenting any errors or warnings.

The server returns the generated XML and report back to the Map Editor via HTTP response. If all goes well, we see the transformed XML in a results window. If there were errors, they appear in the report.

---

## SLIDE 10: How Sterling Executes the Map
**"9. How Sterling Executes the Map"**

### Speaker Notes:

Let me break down the complete execution flow from the moment we click "Run Test" to when we get results.

**Step 1: Map Editor**
We click Run Test. The Map Editor prepares to communicate with the Sterling server.

**Step 2: HTTP POST Request**
The Map Editor sends an HTTP POST request (or sometimes HTTPS for security) to the Sterling server. The request is multipart, meaning it contains multiple "parts":
- Part 1: The compiled .txo file (binary)
- Part 2: The input CSV file (text)
- Part 3: Request metadata (user credentials, execution parameters)

**Step 3: Sterling B2B Integrator Receives Request**
The server receives the multipart HTTP request on its dashboard port.

**Step 4: Authentication and Loading**
The server validates the username and password. If authentication succeeds, it loads the .txo translation object into the translation engine. The .txo is a compiled binary format that the server understands.

**Step 5: Processing**
The server now processes the input CSV:
1. It parses the CSV file according to the input record definition from the .txo
2. It recognizes the comma delimiter and newline record delimiter
3. It reads the header row to confirm column names
4. It enters the DATA loop and reads each data row
5. For each data row, it applies the field mappings
6. For each input DATA occurrence, it generates an output DATA element in XML
7. It wraps all DATA elements in the root OUTPUT element
8. It generates an XML document with proper declaration, formatting, and encoding

**Step 6: Validation and Report Generation**
As it processes, the server monitors for errors:
- Are all mandatory fields present?
- Is the data in the correct format?
- Are cardinality constraints met?
- Any other validation issues?

If there are no errors, the translation report is clean. If there are warnings or errors, they're recorded in the report.

**Step 7: HTTP Response**
The server sends back an HTTP response to the Map Editor:
- HTTP Status Code: 200 (OK) if successful, 4xx/5xx if there was an error
- Content-Type: multipart/mixed (multiple parts in the response)
- Part 1: The generated XML output (MapTestTxResult.xml)
- Part 2: The translation report (MapTestTxReport.xml)

**Step 8: Map Editor Displays Results**
The Map Editor receives the response and displays the results. We can view the generated XML in one tab and the translation report in another.

This entire process typically takes just a few seconds. It's a complete end-to-end transformation happening on the server.

---

## SLIDE 11: Results and Validation Report
**"10. Result & Validation Report"**

### Speaker Notes:

After execution, we receive two key outputs: the transformed XML and the validation report.

**MapTestTxResult.xml – The Transformed Output:**

Here's what we get:

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

Perfect! The output shows exactly what we expected. Each CSV row became a DATA element. Each field became a child element with PCDATA content. The nesting is correct. The XML is well-formed and valid.

If we had 100 CSV rows instead of 3, we'd have 100 DATA elements. If we had 0 rows (empty file), we'd just have an empty OUTPUT element with no DATA children.

Notice the XML declaration at the top: `<?xml version="1.0" encoding="UTF-8" standalone="yes"?>`. This tells any XML consumer that the document is XML 1.0 in UTF-8 encoding, and it's standalone (no external DTD dependencies).

Also notice the formatting: the XML is nicely indented with each element on its own line. Sterling has an output property to control this. Some systems prefer compact XML with no whitespace; others prefer readable indentation.

**MapTestTxReport.xml – The Validation Report:**

```xml
<TranslationReport>
<![CDATA[Translation report contains no data.]]>
</TranslationReport>
```

This report says "contains no data," which actually means "no errors." It's a clean translation. If there had been errors—like a mandatory field missing, or a data type mismatch—they'd appear here inside the CDATA block.

CDATA, by the way, stands for Character Data. It's a special XML section for preserving text exactly as-is, including newlines and special characters, without having to escape them.

**Interpreting the Report:**

A clean report is what we want. It means:
- All CSV rows were successfully parsed
- All field mappings were applied correctly
- No validation errors were encountered
- The output is valid and ready for use

If we saw error messages, they might be things like:
- "Mandatory field NAME is missing in record 2"
- "Field AMOUNT contains non-numeric data"
- "Record structure doesn't match expected format"

When we see errors, we'd examine the problematic input data and adjust either the input CSV or the map logic.

**End-to-End Summary:**

CSV input (3 rows) → Map Editor design (5 fields, 1:1 mapping, repeating DATA) → Compile to .txo → Map Test sends to Sterling server → Server parses CSV (reads header, loops through data rows) → Server applies map logic (copy each field to corresponding element) → Server generates XML (root OUTPUT with 3 DATA children) → Returns XML result and clean report.

This is the complete journey of your data from input to output, all orchestrated by the Sterling B2B Integrator map.

---

## TRAINING SUMMARY

### Key Takeaways:

1. **CSV vs. XML**: CSV is flat and delimited; XML is hierarchical and structured.

2. **Map Structure**: INPUT has a repeating record; OUTPUT has a repeating element. Sterling loops through both.

3. **Field Mapping**: Simple 1:1 connections pull data from CSV fields and push into XML elements.

4. **Loops and Cardinality**: Repeating groups (C-9999) handle multiple records. For each input occurrence, an output occurrence is generated.

5. **XML Basics**: Elements contain PCDATA (text). Elements can be nested in complex structures. Sequences define element order.

6. **Compilation and Deployment**: .mxl (source) → .txo (compiled) → server upload → execution.

7. **Testing**: Map Test dialog sends .txo + CSV to server, receives transformed XML + validation report.

8. **Execution Flow**: Server parses input → applies map logic → generates output → validates → returns result.

---

## ALLOCATE NARRATION TIME PER SLIDE:

- **Slide 1 (Title)**: 1 minute – Set context and introduce the training goal
- **Slide 2 (CSV Input)**: 2 minutes – Explain CSV structure, delimiters, variable-length fields
- **Slide 3 (XML Output)**: 2 minutes – Show XML structure, hierarchy, nesting, and root element
- **Slide 4 (Schema Hierarchy)**: 1.5 minutes – Explain cardinality notation, M/C, element relationships
- **Slide 5 (XML Concepts)**: 2 minutes – Define elements, PCDATA, sequences, complex/simple types using examples
- **Slide 6 (Building Map)**: 2 minutes – Step-by-step record and field creation; explain INPUT/OUTPUT structures
- **Slide 7 (Mapping Logic)**: 2 minutes – Show 1:1 field mapping, loop handling, data flow
- **Slide 8 (Compile)**: 1.5 minutes – Save → Validate → Compile → Deploy process
- **Slide 9 (Map Test)**: 1.5 minutes – Explain dialog fields, credentials, file paths, Run Test
- **Slide 10 (Execution)**: 2 minutes – HTTP flow, server processing, validation, response delivery
- **Slide 11 (Results)**: 1.5 minutes – Show actual XML output, explain report, interpret success/errors

**Total: ~18 minutes** (including pauses and interaction)

This narration is designed for an AI voice generation tool. Speak naturally, as if addressing a room of developers. Pause briefly between key points to allow the audience to absorb information. Emphasize technical terms when first introduced. Use the examples from the slides to ground abstract concepts in reality.

