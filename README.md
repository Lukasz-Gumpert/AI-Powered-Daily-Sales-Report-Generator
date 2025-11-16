<p>
  <img src="AI-Powered-Daily-Sales-Report-Generator\assets\image-1.png" width="150" alt='make'/>
  <img src="AI-Powered-Daily-Sales-Report-Generator\assets\image-2.png" width="150" alt='gemini'/>
</p>


# AI-Powered Daily Sales Report Generator
This project implements an end-to-end business intelligence (BI) solution that transforms raw sales data from a simulated CRM (Google Sheet) into a contextual, summarized daily sales report using Google Gemini AI. The final report is then formatted and delivered directly to stakeholders via email (Gmail), demonstrating proficiency in data integration, AI utilization, natural language generation, and workflow automation.

### Project is divided into two conceptual parts:

- **Scenario 1: Sales Data Generator:**
A Make.com scenario responsible for populating the Google Sheet, simulating a dynamic CRM/ERP system with fresh, realistic sales transactional data (including calculated profit and cost metrics).

- **Scenario 2: AI Sales Intelligence & Notification:** The core workflow that consumes the generated sales data, uses Google Gemini AI for in-depth analysis and report generation, and automatically formats and distributes the final summary via Gmail.

### 1. Sales Data Generator

<p>
<img src="AI-Powered-Daily-Sales-Report-Generator\assets\scenario-1\image-1.png" width=100% alt='sales_data_generator'/>
</p>

### Introduction: ##
This foundational scenario serves as the data preparation layer for the entire project. Its primary function is to simulate a continuous flow of sales data by populating the Google Sheet (sales_data). By generating realistic transactional records, we ensure that the subsequent AI analysis (Scenario 2) operates on dynamic, fresh, and complex data, making the final report generation meaningful.

The implementation of this scenario demonstrates proficiency in data structuring, using variables, and performing core mathematical transformations crucial for any business system.

### Module Breakdown:

### 1. Parse JSON

This initial module ingests a static JSON payload containing all necessary lookup data to generate a realistic transaction record. The data includes: products (price, cost, category), product_list, region, country, salesperson, order_status, and channel.
<p align="center">
<img src="AI-Powered-Daily-Sales-Report-Generator\assets\scenario-1\module-1.PNG"  alt='json_parse_module'/>
</p>

### 2. Setting repeat_num variable and initializing Repeater

This module defines a variable named **repeat_num**, and sets its value (e.g., to 100). This value determines the exact number of sales records the generator will create in a single run.

The Repeater module uses the value of **repeat_num** set in the previous step to initiate a loop. All subsequent modules (data selection, calculation, and insertion) will execute repeatedly (e.g., 100 times) to generate the desired volume of sales data.

<p align="center">
<img src="AI-Powered-Daily-Sales-Report-Generator\assets\scenario-1\module-2.PNG" width=450 alt='variable_module'/>
<img src="AI-Powered-Daily-Sales-Report-Generator\assets\scenario-1\module-3.PNG" width=450 alt='repeater_module'/>
</p>

### 3. Data Calculation & Variable Setting

This sequence of modules, executed within the Repeater loop, is responsible for transforming the randomly selected product name into a complete, financially calculated sales record.

<p align="center">
<img src="AI-Powered-Daily-Sales-Report-Generator\assets\scenario-1\module-4.PNG" alt='variable_modules'/>
</p>

* **current_item_name:** uses the array of product names (from the parsed JSON) and a randomization function to select one product for the transaction. Demonstrates fetching a single, random element from a structured array.
```
get(1.product_list; floor(random * 15) + 1)
```
* **quantity:** sets a random integer between 1 and 5 as the number of units sold. Simulating realistic transaction volumes
```
{{floor(random * 5) + 1}}
```
* **discount_rate:** sets a base discount rate between 0-3%. Introducing financial variability essential for AI analysis.
```
floor(random * 4) / 100
```
* **current_data:** uses the selected **current_item_name** to retrieve the corresponding price, cost, and category data structure from the initial parsed JSON.
```
get(1.products; 4.current_item_name)
```
* **current_price:** extracts the unit price from **current_data** and converts it using parseNumber(). The use of parseNumber() is critical because the final data will be ingested by the AI model, which often processes data initially formatted as a text string (e.g. from CSV or Google Sheets). This function ensures the value is correctly interpreted as a numerical figure, preventing errors related to locale-specific decimal formats (dots vs. commas) in subsequent calculations.
```
parseNumber(get(5.current_data; "price")) * parseNumber(4.quantity)
```
* **total_cost:** calculates the total cost of goods sold. 
```
parseNumber(get(5.current_data; "cost")) * parseNumber(4.quantity)
```
* **final_price:** calculates the selling price after discount. parseNumber() is again used to ensure the final result is correctly formatted for output.
```
parseNumber(7.current_price) * (1 - parseNumber(4.discount_rate))
```
* **profit:** Key Performance Indicator (KPI) Derivation: This is the most crucial derived metric, directly used by the Google Gemini AI in the second scenario for performance reporting. parseNumber() ensures the profit figure is numerically accurate for the Google Sheet output.
```
parseNumber(9.final_price) - parseNumber(7.total_cost)
```

### 4. Data Insertion and Formatting (Google Sheets: Add Rows)
This final, critical module takes all the generated and calculated variables from the previous steps (Randomization, Lookup, and Calculation) and commits them to the target Google Sheet. 

This module maps every variable and calculated value to the correct column in the spreadsheet. It applies final transformations like generating unique IDs and ensuring numerical accuracy for all financial fields.

<p align="center">
<img src="AI-Powered-Daily-Sales-Report-Generator\assets\scenario-1\module-5.PNG" alt='google_sheet_module'/>
</p>

* **Unique ID Generation (Column A):** Uses the concatenation of a static prefix (DN-), a formatted date (YYYYMMDD), and a randomly generated number to create a unique Order ID for tracking.

```
{{"DN-" + formatDate(now; "YYYYMMDD") + "-" + substring(toString(floor(random * 1000000) + 1000000); -6)}}
```

* **order_date (Column B):** ensuring that all generated transactions for a given scenario run have the correct, current date, making the data suitable for the "daily" filtering and reporting in Scenario 2.

```
{{formatDate(now; \ + "YYYY-MM-DD\")}}
```

* **Numerical Fields (Column G, H, I, J, K, L, M):** Data Integrity for AI: Guarantees that the numerical data is clean and accurate for subsequent ingestion by the Google Gemini AI model, preventing parsing errors.

```
toString(formatNumber(5.current_data.price; 2; "."; emptystring))

toString(formatNumber(7.current_price; 2; "."; emptystring))

toString(formatNumber(4.discount_rate; 2; "."; emptystring))

toString(formatNumber(9.final_price; 2; "."; emptystring))

toString(formatNumber(5.current_data.cost; 2; "."; emptystring))

toString(formatNumber(7.total_cost; 2; "."; emptystring))

toString(formatNumber(10.profit; 2; "."; emptystring))
```
* **Categorical Fields (Column C, N, O, P, Q, R):** Random List Selections: Populates fields like Region, Salesperson, and Order_Status using the established randomization logic

```
1.region

get(1.Salesperson; floor(random * 3) + 1)

get(1.country; floor(random * 5) + 1)

get(1.order_status; floor(random * 3) + 1)

get(1.customer_segment; floor(random * 2) + 1)

```

### 5. Output: Generated Sales Data Sample:

The successful execution of the Sales Data Generator populates the linked Google Sheet with high-quality, simulated transactional records. Crucially, the data is stored using strict numerical formatting (e.g., parseNumber(), formatNumber()) and date formatting to ensure data integrity.

This meticulous formatting allows the downstream workflow to seamlessly ingest the data in a CSV-like structure, preventing parsing errors often associated with locale-specific number formats (commas vs. dots) when feeding data to the AI model. This robust dataset, including all necessary calculated fields like Profit and Total Cost, serves as the clean input for the AI Reporting Scenario.

A sample of the generated data is available for download as a [CSV file](AI-Powered-Daily-Sales-Report-Generator\assets\scenario-1\daily_sales.csv)

<p align="center">
<img src="AI-Powered-Daily-Sales-Report-Generator\assets\scenario-1\module-6.PNG" alt='google_sheet_output'/>
</p>

## 2. AI Sales Intelligence & Notification

<p>
<img src="AI-Powered-Daily-Sales-Report-Generator\assets\scenario-2\image-1.PNG" width=100% alt='AI_scenario_preview'/>
</p>

This is the core component of the project, responsible for integrating Google Gemini AI to deliver actionable business intelligence. This scenario runs daily, consuming the clean transactional data generated by Scenario 1, analyzing it, and producing a contextual, human-readable report. The final stage formats and distributes this report directly to stakeholders via email.

This workflow demonstrates expertise in advanced data retrieval, prompt engineering, AI utilization, and integration with third-party communication services (Gmail).

### Module Breakdown:

### 1. Google Sheets: Search Rows

This module connects to the **sales_data** spreadsheet and retrieves the necessary records for the daily report. 

Temporal Filtering based on Text Column: This function is crucial. It ensures that the workflow only pulls records where the **order_text_date** (Column S) matches the current run date. 

The use of Column S, which is automatically populated by the Google Sheets formula guarantees that the date comparison is performed against a pure text value (YYYY-MM-DD). This prevents potential errors associated with comparing native date objects or serial numbers, ensuring highly reliable daily filtering for the automation.

```
=ARRAYFORMULA(IF(ISBLANK(B2:B),"",TEXT(B2:B, "YYYY-MM-DD")))
```

```
order_date_text == formatDate(now; 'YYYY-MM-DD')

```
<p align="center">
<img src="AI-Powered-Daily-Sales-Report-Generator\assets\scenario-2\image-2.PNG" alt='google_sheet_search_rows'/>
</p>

### 2. Tools: Text Aggregator

This module receives the multiple data bundles (one for each sales transaction) retrieved from the Google Sheets module and consolidates them into a single, large text string (output variable: sales_data). This prepares the dataset for efficient input into the Gemini AI model.

Using the line break **\n** as a separator ensures that the data is presented to the AI in a clean, row-by-row format, mimicking a CSV file. This is the most accurate and token-efficient method for providing large, tabular datasets to the Gemini LLM for analysis.

<p align="center">
<img src="AI-Powered-Daily-Sales-Report-Generator\assets\scenario-2\image-3.PNG" alt='tools_text_aggregator'/>
</p>

### 3. Tools: Set Variable

This module sets a single variable, **sales_data**, which will contain the entire block of processed data ready for the AI model. It combines the list of column headers with the serialized data from the Text Aggregator.

<p align="center">
<img src="AI-Powered-Daily-Sales-Report-Generator\assets\scenario-2\image-4.PNG" alt='tools_set_variable'/>
</p>

### 4. Google Gemini AI: Generate a response

This module sends the final, structured sales data (from the sales_data variable) to the Gemini model for processing, analysis, and report generation based on detailed instructions.

**Advanced Prompt Engineering & Meta-Prompting:** The comprehensive prompt used here was initially developed via Meta-Prompting: a technique learned during the Google 'Skills of Tomorrow AI' course. This involved asking the AI itself to refine the role, output format, and constraints for optimal analytical performance.

```
"Act as a Prompt Engineer. We need a daily sales report generated from raw CSV data. Design a professional prompt that assigns the role of 'Expert Financial and Sales Data Analyst' to the model.
Include explicit instructions to calculate Top Profit, identify the Bestselling Region, and enforce a strict Markdown format with double line breaks for email transmission."

```
**The meta-prompt strictly enforces three critical components:** 

1. Role Assignment: Expert Financial and Sales Data Analyst.

2. Strict Output Format: Well-structured Markdown.

3. Critical Constraints: Ensuring **n\\** separation for perfect email readability.

**Footer Insertion and final output control**

A manual, fixed segment was added to the end of the prompt to force the AI to append a specific, professional closing signature (footer) to the report. This demonstrates Strict Output Control, ensuring the AI's final output includes necessary legal or informational disclaimers, links, and branding, even though it wasn't part of the core analysis task.

**Result:**

The prompt is highly structured, ensuring the AI performs complex analysis and delivers a perfectly formatted report. Due to its length, key sections are summarized below, but the full prompt is available in the repository.


**Analysis Requirements (Excerpt)**

* **KPIs:** Total Revenue, Total Profit, Average Discount Rate.
* **Insights:** Top profitable product and highest-selling region.
* **Trend Analysis:** Identify daily trends in discounts or profit consistency.

---
➡️ **VIEW FULL PROMPT:**  **[`gemini_sales_report_prompt.txt`](AI-Powered-Daily-Sales-Report-Generator/prompts/gemini_sales_report_prompt.txt)**

➡️ **VIEW MANUALLY INJECTED FOOTER:** : **[`gemini_sales_report_prompt.txt`](AI-Powered-Daily-Sales-Report-Generator/prompts/gemini_sale_report_footer.txt)**

<p align="center">
<img src="AI-Powered-Daily-Sales-Report-Generator\assets\scenario-2\image-5.PNG" alt='google_gemini_generate_response'/>
</p>

### 5. Markdown to HTML

This essential module takes the analytical result generated by the Gemini AI—which is formatted in Markdown and converts it into standard HTML format.

The Sanitize option is explicitly disabled. This is critical because the AI was instructed to include Markdown formatting and the specific custom HTML required for the signature footer. Disabling sanitization ensures that all necessary structural elements and custom line breaks generated by the AI are preserved in the final HTML email body.

<p align="center">
<img src="AI-Powered-Daily-Sales-Report-Generator\assets\scenario-2\image-6.PNG" alt='markdown_to_html'/>
</p>

### 6. Report Distribution (Gmail: Send an email)

This final module handles the critical task of securely connecting to the designated mailbox and sending the analyzed and formatted report to the stakeholder.

Dynamic Subject Creation: The subject line is dynamically generated. It combines static branding text (make.com daily sales analysis) with the current date, ensuring the recipient immediately knows the report's content and its temporal relevance.

The email body is explicitly set to accept HTML content. This choice is mandatory, as it accepts the highly structured output from the preceding Markdown to HTML module, allowing the use of headings, tables, and bullet points for professional presentation.

<p align="center">
<img src="AI-Powered-Daily-Sales-Report-Generator\assets\scenario-2\image-7.PNG" alt='module-7'/>
</p>

## Final Results and Proof of Concept

This section demonstrates the successful end-to-end execution of the project, showcasing the Google Gemini AI's ability to analyze tabular data and generate a professional, formatted business report ready for distribution.

Both Scenario 1 (Data Generator) and Scenario 2 (AI Reporting) are configured to run on a fixed daily schedule. This ensures the mock CRM is updated daily and the final report is delivered to management at the same consistent time, a key element of enterprise-level automation reliability.

The image below shows the final email received by the stakeholder. The content, generated by the Gemini model, has been successfully converted from Markdown to HTML by the workflow, preserving all structural elements (headings, bold text, lists) and adhering to the strict formatting and signature rules defined in the prompt.

<p align="center">
<img src="AI-Powered-Daily-Sales-Report-Generator\assets\scenario-2\image-8.PNG" alt='final-result'/>
</p>

## Conclusion and Key Learnings

This project was more than just an automation exercise—it was a practical stress test of the knowledge gained from the Google AI course and a real-world confrontation with business data challenges. Here is what I consider my most important lessons and achievements:

The primary lesson for me was learning how to build automation that can be relied upon. By configuring the scenarios to always run at the same time: first the CRM data update, and shortly thereafter, the report generation. I gained the confidence that I had created a reliable system that delivers value on time, without manual intervention.

Prompt Engineering: I successfully moved beyond simple conversational AI. Thanks to techniques like Meta-Prompting (asking the AI for help in writing the prompt) and Strict Output Control, I was able to dictate not only what Gemini should analyze but also how the output must look. This was the critical skill that allowed me to perfectly integrate the AI result into the final email body.

System Integration: Breaking down barriers between various tools became standard practice. The project demanded the secure and efficient connection and data management between Google Sheets, Make.com, the Gemini AI model, and Gmail. This mastery is the foundation of any modern automation.


