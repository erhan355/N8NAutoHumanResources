# Automation workflow using n8n that processes CVs from email attachments, extracts key information using AI, and organizes the data into Google Sheets.
![image](https://github.com/user-attachments/assets/d4c53955-d627-414f-822f-20ba81f508af)


https://github.com/user-attachments/assets/8903f47c-3c1a-473f-b069-dca70babb943

# Before you begin building this workflow, make sure you have the following set up:

* n8n Instance: You'll need access to n8n, either self-hosted or cloud-based.  
* Google Account: Required for Gmail, Google Drive, and Google Sheets integration.  
* OpenAI API Key: To access AI capabilities for processing CV content.  
* Google OAuth credentials with appropriate permissions for Gmail, Drive, and Sheets  
OpenAI API credentials  

## 1. Email Trigger and File Handling
The workflow begins when a new email with a CV attachment arrives in Gmail.
The Gmail Trigger node is configured to:

Monitor for new messages with attachments

Filter for unread emails only

Accept emails from a specific sender.

Download attachments with the prefix "Cv"

This ensures that only relevant CV files from trusted sources enter our workflow.

![image](https://github.com/user-attachments/assets/ba23bcad-f973-49eb-aa77-78ce78754cb9)


## 2. Google Drive Integration
After the email trigger activates, the attached CV is uploaded to Google Drive for storage and processing. We use two Google Drive nodes in succession:

First Google Drive node: Uploads the file to a specified folder

Second Google Drive node: Downloads the file for processing

![image](https://github.com/user-attachments/assets/fc06e9c8-5ac0-4375-9714-b306423daac2)
![image](https://github.com/user-attachments/assets/740f6556-dce8-46cf-8a2f-70946fc31a9f)

## 3. Extract From File
This node extracts the text content from the PDF CV file, making it readable for the next steps in our workflow.
![image](https://github.com/user-attachments/assets/5780d4da-7e4c-46f1-8a7e-8bf34d9d59bb)

## 4. OPEN AI
Our workflow uses OpenAI's capabilities to analyze the CV content. This node sends the extracted CV text to OpenAI and requests structured analysis.

![image](https://github.com/user-attachments/assets/388ede12-3826-485c-b8ca-a750b9285318)

Prompt : 
```
You are the world's most accurate and efficient CV summarizer known for producing concise and informative summaries that capture all essential details.

Your task is to compare the candidate's education and skill set with the provided job description, evaluate it with the given score set, and summarize the CV in the output format.

Instructions:
1. Extract all key information exactly as presented in the CV
2. Compare candidate qualifications with the job requirements
3. Evaluate the candidate on a 1-10 scale
4. Present findings in the specified format

Scoring Scale:
1-3: Weak candidate
4-6: Moderate candidate
7-8: Strong candidate
9-10: Exceptional candidate

Output Format:

Educational Qualifications
[Degree] [Institution] [Year]

Job History
[Job Title] [Company] [Date]

Skills
[Skill 1] [Skill 2] [Skill 3] [etc.]

Evaluation Score: [1-10]

Evaluation Rationale:
[Provide a brief, factual explanation of why this score was assigned]
[List specific matches and gaps between the CV and job requirements]
[Do not include subjective assessments or assumptions]

Important Notes:
- Extract information exactly as presented in the CV
- Do not add interpretations or assumptions about the CV content
- Use only formal, professional language
- Provide only the requested information in the specified format

Example Input
Job Description:
Senior Software Developer
Requirements:
- Bachelor's degree in Computer Science or related field
- 5+ years of experience in software development
- Proficient in Python, JavaScript, and SQL
- Experience with cloud platforms (AWS, Azure)
- Strong problem-solving and communication skills

CV:
John Smith
Education:
- Master of Science in Computer Science, Stanford University, 2018
- Bachelor of Engineering in Information Technology, MIT, 2016

Experience:
- Software Engineer, Google, Jan 2019 - Present
- Junior Developer, Microsoft, Aug 2016 - Dec 2018

Skills:
- Programming: Python, Java, C++, JavaScript
- Databases: MySQL, PostgreSQL, MongoDB
- Tools: Git, Docker, Jenkins
- Cloud: AWS, Google Cloud
- Languages: English (native), Spanish (intermediate)

Example Output:
Educational Qualifications
Master of Science in Computer Science, Stanford University, 2018
Bachelor of Engineering in Information Technology, MIT, 2016

Job History
Software Engineer, Google, Jan 2019 - Present
Junior Developer, Microsoft, Aug 2016 - Dec 2018

Skills
Python, Java, C++, JavaScript, MySQL, PostgreSQL, MongoDB, Git, Docker, Jenkins, AWS, Google Cloud, English, Spanish

Evaluation Score: 8

Evaluation Rationale:
Score of 8 (Strong candidate) based on:
1. Education exceeds requirements with Master's degree from Stanford
2. Experience matches requirements with 5+ years at top tech companies
3. Skills align with job needs (Python, JavaScript, AWS)
4. Missing specific SQL experience mentioned in requirements

Job Description :
We are looking for a skilled .NET Fullstack Developer with a strong background in projects to contribute to the design, development, and maintenance of our payment service software applications. You will work closely with cross-functional teams to deliver high-quality solutions that meet the unique needs of our payment systems client.



QUALIFICATIONS

Bachelor's degree in Computer Science, Software Engineering, or a related field,
Senior Full Stack: NET core 3.1 or .NET 6 (+8yrs exp), React( 6yrs exp.)+ Cloud Preference GCP, Test Automation
Excellent programming skills in Microsoft .Net with MVC, C#, WebAPI, EF and SQL knowledge
Eager to learn new technologies,
Good team player, result oriented attitude and analytical mind,
Strong communicational and interpersonal skills.
High energy and drive
Experience in relational database design and development including Oracle, MySQL


RESPONSIBILITIES

Collaborate with business analysts, project managers, and other stakeholders to understand and define project requirements.
Design, develop, and maintain .NET-based applications for payment projects, ensuring scalability, security, and performance.
Participate in the full software development life cycle, including planning, coding, testing, and deployment.
Conduct code reviews and provide constructive feedback to ensure code quality and adherence to coding standards.
Troubleshoot, debug, and resolve software defects and issues in a timely manner.
Stay current with industry trends and best practices to continuously improve development processes and technologies.
```

![image](https://github.com/user-attachments/assets/808891ce-ef87-4a83-8a56-a26d15e513cd)
Edit fields code
```
message.content => {{ $json.choices[0].message.content }}
```

The Code node runs custom JavaScript to extract specific sections from the CV text:

![image](https://github.com/user-attachments/assets/57336ee7-445e-4a3c-b400-889f5a7ebda4)

Code
```
// Extract relevant sections from the input text
const inputData = $input.item.json.message.content;

// Simple extraction function using indexOf and substring
const extractSection = (text, startSection, endSection) => {
  const startIdx = text.indexOf(startSection);
  if (startIdx === -1) return null;
  
  const startContentIdx = startIdx + startSection.length;
  let endIdx = text.length;
  
  if (endSection) {
    const nextSectionIdx = text.indexOf(endSection, startContentIdx);
    if (nextSectionIdx !== -1) {
      endIdx = nextSectionIdx;
    }
  }
  
  return text.substring(startContentIdx, endIdx).trim();
};

// Extract key sections from the CV
const educationalQualifications = extractSection(
  inputData, 
  "Educational Qualifications", 
  "Job History"
);

const jobHistory = extractSection(
  inputData, 
  "Job History", 
  "Skills"
);

// Check if we have "Skills" or "Skill Set" in the text
const skillSectionName = inputData.indexOf("Skill Set") !== -1 ? "Skill Set" : "Skills";
let skillSet = extractSection(
  inputData, 
  skillSectionName,
  "Evaluation Score"  // Looking directly for the score section
);

if (!skillSet) {
  // Try with "Candidate Evaluation" as the end marker
  skillSet = extractSection(
    inputData, 
    skillSectionName,
    "Candidate Evaluation"
  );
}

// Extract evaluation info
const scoreSection = extractSection(
  inputData,
  "Evaluation Score:",
  "Evaluation Rationale:"
);

const score = scoreSection ? scoreSection.trim() : null;

const justification = extractSection(
  inputData,
  "Evaluation Rationale:",
  null  // Assuming this is the last section
);

// Return structured output as a flat JSON object
return {
  educationalQualifications,
  jobHistory,
  skillSet,
  score,
  justification
};
```

## 4. Information Extractor
The Information Extractor node uses a JSON Schema to define what data to extract from the CV:
![image](https://github.com/user-attachments/assets/39700b83-d0bf-49ee-a14f-a1fb6cce8ca7)
Json : 
```
{
  "type": "object",
  "properties": {
    "candidate_name": {
      "type": "string"
    },
    "email_address": {
      "type": "string",
      "format": "email"
    },
    "contact_number": {
      "type": "string",
      "pattern": "^\\+\\d{1,3}\\(\\d{3}\\)-\\d{7}$"
    }
  }
}
```
![image](https://github.com/user-attachments/assets/ef31c520-7db9-4f65-ae37-7fa28df467fc)

## 5. Merge
The Merge node combines data from multiple sources:

Mode: Combine
Combine By: Position
Number of Inputs: 2

This creates a unified dataset containing both the basic candidate information and the detailed section extracts.

![image](https://github.com/user-attachments/assets/6f01aaed-1cc7-4ace-b5a4-41493f53a979)

## 6. Edit After Merge
This node finalizes our data structure before sending it to Google Sheets:

![image](https://github.com/user-attachments/assets/b7334161-8655-40cb-a1ca-30702a4831f1)

```
{
  "candidateName": "{{ $json.output.candidate_name }}",
  "educationalQualifications": "{{ $json.educationalQualifications ? $json.educationalQualifications.replace(/\n/g, '\\n') : '' }}",
  "jobHistory": "{{ $json.jobHistory ? $json.jobHistory.replace(/\n/g, '\\n') : '' }}",
  "skillSet": "{{ $json.skillSet }}",
  "score": "{{ $json.score }}",
  "justification": "{{ $json.justification ? $json.justification.replace(/\n/g, '\\n') : '' }}"
}
```

## 7. Google Sheets Integration

The final node in our workflow sends the processed data to Google Sheets:
![image](https://github.com/user-attachments/assets/2065d5d0-03b2-4390-9d15-d1f4e4c836db)

Operation: Append Row
Document: "Adaylar"
Sheet: "Sheet1"
Mapping Column Mode: Map Each Column Manually

Each CV processed through the workflow adds a new row to our spreadsheet with these mapped fields:

candidateName
educationalQualifications
jobHistory
skillSet
score
```
{{ $json.candidateName }}
{{ $json.educationalQualifications }}
{{ $json.jobHistory }}
{{ $json.skillSet }}
{{ $json.score }}
{{ $json.justification }}
```
