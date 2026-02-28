# Nyaya Saathi — Requirements Document

## Introduction

Nyaya Saathi (Justice Companion) is an AI-powered legal rights awareness, case strategy, and grievance drafting assistant designed to bridge the justice gap for marginalized communities in India. The system enables rural women, SC/ST communities, daily-wage workers, migrant laborers, and trafficking survivors to understand their legal rights, assess case strength, compare legal options, and take action — in their own language, without needing a lawyer or expensive legal consultations.

The platform addresses a critical problem: while 80 million+ Indians are eligible for free legal aid, only 15 million access it. Target users often don't know their legal rights, cannot navigate 1,800+ central and state laws, and find legal language impenetrable. Nyaya Saathi provides conversational legal guidance in Hindi and English (with future regional language support), using Amazon Bedrock's AI capabilities with Retrieval-Augmented Generation (RAG) to match real-world situations to relevant legal acts and generate actionable documents.

The system uses a React/Next.js frontend hosted on AWS Amplify, a serverless backend with AWS Lambda functions, Amazon Bedrock for AI reasoning, Amazon Polly for Hindi text-to-speech, Amazon Textract for document scanning, and Bedrock Guardrails for safety — totaling 14 AWS services.

### Target Track
- **Student Track — Problem Statement 3:** AI for Communities, Access & Public Impact

---

## Problem Statement

India has over 1,800 central and state laws, yet access to legal awareness remains critically low among marginalized communities. According to NALSA (National Legal Services Authority) data, approximately 80 million Indians are eligible for free legal aid, but only about 15 million actually access it. The gap exists because:

- **Legal language is impenetrable:** Acts and sections are written in complex English legal jargon that even educated urban Indians struggle with.
- **Lawyers are expensive:** A basic legal consultation costs ₹500–5,000, putting it out of reach for daily-wage workers earning ₹300–500/day.
- **Awareness is near-zero:** A rural woman facing domestic violence may not know that the Protection of Women from Domestic Violence Act (2005) gives her the right to reside in her marital home. A Dalit facing caste-based discrimination may not know the SC/ST Prevention of Atrocities Act protects them. A worker with unpaid wages may not know they can file an RTI to check their MGNREGA payment status.
- **Free legal aid is underutilized:** District Legal Services Authorities exist in every district but are severely underutilized due to lack of awareness and accessibility.
- **Filing complaints is daunting:** Even when people know their rights, drafting an FIR, RTI application, or formal complaint requires literacy and legal formatting knowledge they don't have.
- **Literacy barriers:** 25% of Indian women are illiterate — they cannot read text on a screen at all, making text-only solutions insufficient.

---

## Why AI Is Needed (Not Just Digitization)

A static FAQ or searchable database cannot solve this problem because:

- **Contextual reasoning required:** Matching a person's messy, emotional, real-world description ("My husband beats me and my in-laws took my gold jewelry") to specific legal provisions across 1,800+ laws requires natural language understanding and multi-hop reasoning — not keyword search.
- **Combinatorial complexity:** A single situation may invoke multiple acts simultaneously (DV Act + IPC Section 498A + Dowry Prohibition Act). An AI must reason across these to provide comprehensive guidance.
- **Legal strategy assessment:** Evaluating case strength, identifying evidence gaps, and comparing multiple legal paths (criminal FIR vs civil protection order vs DLSA mediation) requires reasoning that goes beyond information retrieval.
- **Document generation:** Auto-drafting a personalized complaint letter or RTI application requires text generation capabilities that go beyond template filling — every situation is unique.
- **Document understanding:** Analyzing an uploaded legal notice or court summons that a user cannot read requires OCR + reasoning capabilities.
- **Language barriers:** Translating legal concepts (not just words) between English legal text and Hindi/regional languages requires semantic understanding, not dictionary lookup.
- **Voice accessibility:** Reading legal rights aloud in natural Hindi requires neural text-to-speech, not robotic voice synthesis.

---

## Glossary

| Term | Definition |
|------|------------|
| **System** | The Nyaya Saathi platform |
| **User** | An individual seeking legal rights information or grievance drafting assistance |
| **Situation** | A user's description of their legal problem or concern in natural language |
| **Legal_Act** | Indian legislation (e.g., IPC/BNS, Domestic Violence Act, MGNREGA) |
| **Rights_Explanation** | Plain-language description of applicable legal rights at Class 8 reading level |
| **Grievance_Document** | Auto-generated complaint, FIR description, RTI application, or legal aid application |
| **DLSA** | District Legal Services Authority — provides free legal aid to eligible citizens |
| **RAG** | Retrieval-Augmented Generation — vector search over legal corpus to ground LLM responses |
| **Intent** | Classified user request type: `new_situation`, `follow_up`, `document_request`, `document_scan`, `legal_aid_search`, `quiz`, `out_of_scope` |
| **Knowledge_Base** | Amazon Bedrock Knowledge Base containing 15+ Indian legal act PDFs indexed via Amazon Titan Embeddings V2 |
| **LLM** | Large Language Model — Amazon Nova Lite or Claude 3 Sonnet via Amazon Bedrock |
| **Session** | A temporary conversation context that does not persist PII beyond the interaction |
| **Source_Attribution** | Specific Act name and Section number cited in every response for independent verification |
| **Confidence_Score** | AI's certainty level in rights matching — low confidence triggers explicit uncertainty disclosure and DLSA referral |
| **IPC** | Indian Penal Code — replaced by BNS (Bharatiya Nyaya Sanhita) in July 2024 |
| **BNS** | Bharatiya Nyaya Sanhita — replacement for IPC, effective July 2024 |
| **BNSS** | Bharatiya Nagarik Suraksha Sanhita — replacement for CrPC, effective July 2024 |
| **BSA** | Bharatiya Sakshya Adhiniyam — replacement for Indian Evidence Act, effective July 2024 |
| **TTS** | Text-to-Speech — Amazon Polly converts rights explanations to spoken Hindi audio |
| **Streaming** | Word-by-word response delivery via Lambda Function URLs, similar to ChatGPT-style typing effect |
| **Case_Strength** | AI-assessed score (0-100%) evaluating the strength of a user's potential legal case based on available facts and evidence |
| **Legal_Path** | A distinct legal option available to a user (e.g., criminal FIR, civil protection order, DLSA mediation), compared side-by-side |
| **Visual_Journey** | A Mermaid.js-rendered flowchart showing step-by-step legal process from problem to resolution |

---

## Requirements

### Requirement 1: Multilingual Conversational Intake

**User Story:** As a user with limited English proficiency, I want to describe my legal situation in Hindi, English, or Hinglish using text or voice, so that I can access legal guidance without language or literacy barriers.

#### Acceptance Criteria

1. WHEN a user submits text input in Hindi (e.g., "Mere pati mujhe marte hain aur sasural waalon ne mera zewar le liya"), THEN THE System SHALL process and understand the input accurately
2. WHEN a user submits text input in English (e.g., "My employer hasn't paid my wages for 3 months"), THEN THE System SHALL process and understand the input accurately
3. WHEN a user submits text input in Hinglish (mixed Hindi-English), THEN THE System SHALL process and understand the input without requiring the user to switch languages
4. WHEN the System processes user input, THEN THE System SHALL ask clarifying follow-up questions if needed (e.g., "Are you currently living in your marital home?" or "Is your employer a government contractor or private?")
5. WHEN the input field is displayed, THEN THE System SHALL support text input of up to 2,000 characters per message
6. WHEN the input field is displayed, THEN THE System SHALL provide clear instructions in both Hindi and English for how to describe their situation
7. WHEN a user clicks the microphone button, THEN THE System SHALL activate browser-native voice input (Web Speech API) for speech-to-text conversion (see Requirement 16)

---

### Requirement 2: Intent Classification

**User Story:** As the system, I want to classify user requests into specific intent categories, so that I can route them to appropriate processing workflows.

#### Acceptance Criteria

1. WHEN a user describes a new legal situation, THEN THE System SHALL classify the intent as `new_situation`
2. WHEN a user asks follow-up questions about a previous response, THEN THE System SHALL classify the intent as `follow_up`
3. WHEN a user requests document generation (RTI, FIR, complaint letter, legal aid application), THEN THE System SHALL classify the intent as `document_request`
4. WHEN a user uploads a photo or image of a document for analysis, THEN THE System SHALL classify the intent as `document_scan`
5. WHEN a user asks for legal aid center information, THEN THE System SHALL classify the intent as `legal_aid_search`
6. WHEN a user initiates the "Know Your Rights" quiz, THEN THE System SHALL classify the intent as `quiz`
7. WHEN a user request is outside legal assistance scope, THEN THE System SHALL classify the intent as `out_of_scope` and respond helpfully
8. WHEN intent classification confidence is below threshold, THEN THE System SHALL ask clarifying questions rather than making assumptions

---

### Requirement 3: Situation-to-Rights Mapping Using RAG

**User Story:** As a user describing my legal problem, I want the system to identify which laws apply to my situation, so that I can understand my legal rights.

#### Acceptance Criteria

1. WHEN a user describes a situation, THEN THE System SHALL query the Knowledge_Base using RAG to retrieve relevant legal provisions
2. WHEN querying the Knowledge_Base, THEN THE System SHALL use Amazon Titan Embeddings V2 to generate query embeddings for semantic matching
3. WHEN retrieving legal provisions, THEN THE System SHALL search across all 15+ Indian legal acts in the Knowledge_Base (see Requirement 6 for full list)
4. WHEN multiple legal acts are relevant to a single situation (e.g., DV Act + IPC Section 498A + Dowry Prohibition Act for a domestic violence case), THEN THE System SHALL return provisions from all applicable acts with cross-references
5. WHEN RAG retrieval returns results, THEN THE System SHALL include Source_Attribution with specific Act name and Section number for each identified right
6. WHEN criminal provisions are cited, THEN THE System SHALL display both old law (IPC/CrPC) and new law (BNS/BNSS) references side-by-side (see Requirement 23)
7. WHEN RAG retrieval Confidence_Score is low, THEN THE System SHALL explicitly indicate uncertainty and suggest DLSA consultation instead of guessing
8. WHEN rights identification is complete, THEN THE System SHALL return the response within 15 seconds (or begin streaming within 3 seconds)
9. WHEN RAG retrieval returns results, THEN THE System SHALL also return the raw retrieved legal text chunks for side-by-side display alongside the simplified explanation (see Requirement 21)

---

### Requirement 4: Plain-Language Rights Explanation

**User Story:** As a user with Class 8 education level, I want legal rights explained in simple Hindi or English, so that I can understand my rights without legal jargon.

#### Acceptance Criteria

1. WHEN generating a Rights_Explanation, THEN THE System SHALL use vocabulary appropriate for Class 8 reading level (simple sentence structure, common words)
2. WHEN generating a Rights_Explanation in Hindi, THEN THE System SHALL use simple Hindi words, avoiding English legal terms wherever possible
3. WHEN explaining legal provisions, THEN THE System SHALL provide concrete examples relevant to the user's specific situation
4. WHEN multiple rights apply, THEN THE System SHALL organize the explanation with clear section headings and numbered points
5. WHEN citing legal provisions, THEN THE System SHALL include Source_Attribution in parentheses after each right (e.g., "Protection of Women from Domestic Violence Act 2005, Section 12")
6. WHEN the explanation is complete, THEN THE System SHALL include the disclaimer: "यह जानकारी केवल मार्गदर्शन के लिए है, यह कानूनी सलाह नहीं है। कृपया किसी वकील या जिला विधिक सेवा प्राधिकरण से संपर्क करें।"
7. WHEN explaining each right, THEN THE System SHALL cover three aspects: what the right means in practical terms, what the user can do about it, and what protection it offers

> **Example:** Instead of "Section 17 of PWDVA confers right of residence," the system says: "कानून कहता है कि आपको अपने ससुराल के घर में रहने का पूरा अधिकार है। कोई भी आपको घर से निकाल नहीं सकता।" (The law says you have full right to live in your marital home. No one can throw you out.)

---

### Requirement 5: Auto-Generation of Grievance Documents

**User Story:** As a user who needs to file a complaint, I want the system to generate the required document for me as a downloadable PDF, so that I can print and submit it to authorities without hiring a lawyer.

#### Acceptance Criteria

1. WHEN a user requests an RTI application, THEN THE System SHALL generate a properly formatted RTI application per RTI Act requirements with proper addressing, subject line, and specific information sought
2. WHEN a user requests an FIR description, THEN THE System SHALL generate a structured narrative suitable for police complaint registration, including relevant IPC/BNS sections
3. WHEN a user requests a complaint letter, THEN THE System SHALL generate a formal complaint letter addressed to the appropriate authority (District Collector, Labor Commissioner, Women's Commission, etc.)
4. WHEN a user requests a legal aid application, THEN THE System SHALL generate a DLSA legal aid application form with eligibility justification
5. WHEN generating any Grievance_Document, THEN THE System SHALL include:
   - Proper formatting (address, date, subject line, body, signature block)
   - All relevant facts from the user's described situation
   - Applicable Legal_Acts and section numbers (both IPC and BNS where applicable)
   - Placeholder markers for personal details clearly marked as `[YOUR NAME]`, `[YOUR ADDRESS]`, `[YOUR AADHAAR NUMBER]`, etc.
6. WHEN a Grievance_Document is generated, THEN THE System SHALL provide it in the user's chosen language (Hindi or English)
7. WHEN a Grievance_Document is generated, THEN THE System SHALL provide step-by-step submission instructions (where to go, what to carry, what to expect)
8. WHEN a Grievance_Document is generated, THEN THE System SHALL include the disclaimer: "This is guidance, not legal advice"
9. WHEN a Grievance_Document is generated, THEN THE System SHALL allow the user to copy the text to clipboard or download it as a formatted PDF (see Requirement 20)

---

### Requirement 6: Legal Acts Coverage

**User Story:** As a user with a legal issue, I want the system to cover major Indian laws relevant to marginalized communities, so that I can get guidance on common legal problems.

#### Acceptance Criteria

1. WHEN the Knowledge_Base is initialized, THEN THE System SHALL include the following 15+ legal acts:

| # | Legal Act | Key Coverage Area |
|---|-----------|-------------------|
| 1 | Indian Penal Code (IPC) / Bharatiya Nyaya Sanhita (BNS) | Criminal offenses |
| 2 | Code of Criminal Procedure (CrPC) / Bharatiya Nagarik Suraksha Sanhita (BNSS) | Criminal procedure |
| 3 | Indian Evidence Act / Bharatiya Sakshya Adhiniyam (BSA) | Evidence law |
| 4 | Protection of Women from Domestic Violence Act, 2005 | Domestic violence |
| 5 | Dowry Prohibition Act, 1961 | Dowry harassment |
| 6 | SC/ST (Prevention of Atrocities) Act, 1989 | Caste-based discrimination |
| 7 | Right to Information Act, 2005 | Government transparency |
| 8 | Mahatma Gandhi NREGA (MGNREGA), 2005 | Rural employment guarantee |
| 9 | Consumer Protection Act, 2019 | Consumer rights |
| 10 | Motor Vehicles Act, 1988 | Traffic challan grievances |
| 11 | Payment of Wages Act, 1936 | Wage disputes |
| 12 | Minimum Wages Act, 1948 | Minimum wage enforcement |
| 13 | Maintenance and Welfare of Parents and Senior Citizens Act, 2007 | Elder care |
| 14 | Protection of Children from Sexual Offences (POCSO) Act, 2012 | Child sexual abuse |
| 15 | Right to Education Act, 2009 | Education access |
| 16 | Legal Services Authorities Act, 1987 | Free legal aid eligibility |
| 17 | Maternity Benefit Act, 1961 | Maternity protection |
| 18 | Prohibition of Child Marriage Act, 2006 | Child marriage |

2. WHEN all legal act PDFs are collected, THEN THE System SHALL store them in Amazon S3 with organized folder structure
3. WHEN the Knowledge_Base indexes new acts, THEN THE System SHALL use Amazon Titan Embeddings V2 for chunking and vector embedding
4. WHEN a user's situation spans multiple acts, THEN THE System SHALL cross-reference provisions across all applicable acts
5. WHEN criminal provisions are cited, THEN THE System SHALL include both old law (IPC/CrPC/Evidence Act) and new law (BNS/BNSS/BSA) references

---

### Requirement 7: Nearest Free Legal Aid Center Finder

**User Story:** As a user who needs in-person legal assistance, I want to find the nearest free legal aid center, so that I can access professional help without cost.

#### Acceptance Criteria

1. WHEN a user requests legal aid center information, THEN THE System SHALL query the DLSA directory
2. WHEN querying the DLSA directory, THEN THE System SHALL use the user's specified state and district
3. WHEN no location is provided, THEN THE System SHALL ask the user for their district or state
4. WHEN DLSA centers are found, THEN THE System SHALL return up to 3 nearest centers with: name, address, phone number, and operating hours
5. WHEN displaying DLSA information, THEN THE System SHALL explain who is eligible for free legal aid:
   - Women (all categories)
   - Members of SC/ST communities
   - Persons with disabilities
   - Victims of trafficking
   - Industrial workers
   - Persons in custody
   - Persons below the income threshold set by the state government
6. WHEN the DLSA directory is initialized, THEN THE System SHALL store the directory data in Amazon S3

---

### Requirement 8: Prompt Chaining Workflow

**User Story:** As the system, I want to process user requests through a multi-stage pipeline, so that I can provide comprehensive and accurate legal guidance through structured reasoning.

#### Acceptance Criteria

1. WHEN a `new_situation` intent is detected, THEN THE System SHALL execute Stage 1: Situation Analyzer (extract facts, identify legal domains)
2. WHEN Stage 1 completes, THEN THE System SHALL execute Stage 2: Rights Explainer (RAG retrieval + plain-language explanation + IPC→BNS mapping + case strength assessment + multi-path comparison + visual journey map)
3. WHEN Stage 2 completes, THEN THE System SHALL make available Stage 3: Document Drafter (when user requests a document)
4. WHEN a `document_request` intent is detected, THEN THE System SHALL execute the Document Drafter stage using context from previous stages
5. WHEN a `document_scan` intent is detected, THEN THE System SHALL execute the Document Analyzer stage using Amazon Textract + Bedrock
6. WHEN a `follow_up` intent is detected, THEN THE System SHALL use the current Session context to provide continuity
7. WHEN a `quiz` intent is detected, THEN THE System SHALL execute the Quiz Generator to produce legal awareness questions
8. WHEN any stage fails, THEN THE System SHALL provide a graceful error message and suggest DLSA consultation as fallback

---

### Requirement 9: Responsible AI and Safety Guardrails

**User Story:** As a system operator, I want the AI to refuse inappropriate requests and avoid harmful outputs using Amazon Bedrock Guardrails, so that the system is used ethically and responsibly.

#### Acceptance Criteria

1. WHEN a user requests guidance on illegal activities, THEN THE System SHALL refuse and explain that it cannot assist with illegal actions
2. WHEN a user asks for case outcome predictions or sentencing predictions, THEN THE System SHALL refuse and explain that outcomes cannot be predicted
3. WHEN generating responses, THEN THE System SHALL avoid bias in coverage across different communities (SC/ST, women, workers, minorities)
4. WHEN a user describes a situation requiring immediate safety intervention (e.g., life-threatening domestic violence, child abuse), THEN THE System SHALL provide emergency contact information:
   - Police: 100
   - Women Helpline: 181
   - Child Helpline: 1098
   - National Commission for Women: 7827-170-170
5. WHEN Confidence_Score is low for rights matching, THEN THE System SHALL explicitly state uncertainty and recommend DLSA consultation
6. WHEN any response is generated, THEN THE System SHALL include the prominent disclaimer in both Hindi and English: "यह जानकारी केवल मार्गदर्शन के लिए है, यह कानूनी सलाह नहीं है। कृपया किसी वकील या जिला विधिक सेवा प्राधिकरण से संपर्क करें।" / "This information is for guidance only, it is not legal advice. Please contact a lawyer or District Legal Services Authority."
7. WHEN a situation is too complex or ambiguous for confident guidance, THEN THE System SHALL acknowledge limitations and recommend consulting a lawyer in person
8. WHEN any Bedrock LLM invocation is made, THEN THE System SHALL apply Amazon Bedrock Guardrails with the following configuration:
   - **Denied topics:** Case outcome prediction, encouraging illegal activity, medical/financial advice
   - **Content filters:** Block hate speech, violence glorification, sexual content
   - **Sensitive info filters:** Block PII in responses (Aadhaar numbers, phone numbers)
9. WHEN Bedrock Guardrails block a response, THEN THE System SHALL display a user-friendly message explaining that the request cannot be processed, and suggest rephrasing or consulting DLSA

---

### Requirement 10: Session Management and Privacy

**User Story:** As a user sharing sensitive legal information, I want my personal data to be protected and not stored permanently, so that my privacy is maintained.

#### Acceptance Criteria

1. WHEN a user starts a conversation, THEN THE System SHALL create a temporary Session with a unique identifier
2. WHEN a Session is active, THEN THE System SHALL maintain conversation context for follow-up questions within the same session
3. WHEN a Session ends (browser close or timeout), THEN THE System SHALL delete all conversation data including user situation details
4. THE System SHALL NOT store personally identifiable information (PII) in persistent storage at any time
5. THE System SHALL NOT store user names, addresses, phone numbers, Aadhaar numbers, or case details beyond the active Session
6. WHEN communicating with backend services, THEN THE System SHALL use HTTPS encryption for all API calls
7. THE System SHALL NOT require login or registration for basic use — access is anonymous. This is a deliberate safety design choice: a woman using her abuser's phone cannot leave a login trail.
8. WHEN using AWS services, THEN THE System SHALL enforce IAM-based access control with least-privilege policies

---

### Requirement 11: Mobile-Responsive React Web Interface

**User Story:** As a user accessing the system from a budget smartphone on a 2G/3G connection, I want the interface to work well on small screens with limited bandwidth, so that I can use it on my device without frustration.

#### Acceptance Criteria

1. WHEN the web interface loads on a mobile device, THEN THE System SHALL display a responsive layout optimized for screens as small as 320px width
2. WHEN the web interface loads, THEN THE System SHALL use a React/Next.js frontend generated via v0/Bolt for a professional, production-quality appearance
3. WHEN the web interface displays text, THEN THE System SHALL use readable font sizes (minimum 16px body text) for users with limited vision
4. WHEN the web interface displays input fields/buttons, THEN THE System SHALL make them easily tappable on touchscreens (minimum 44px height)
5. WHEN the web interface displays content, THEN THE System SHALL use high-contrast text for outdoor/bright-light readability
6. WHEN the web interface renders rights cards, THEN THE System SHALL display expandable cards showing: Right name → Simple explanation → Act & Section → Action steps
7. WHEN the web interface renders a document, THEN THE System SHALL display it in a panel with Copy, Download PDF, and Listen (🔊) buttons
8. WHEN the web interface renders responses, THEN THE System SHALL display them with word-by-word streaming (ChatGPT-style typing effect) for a professional, real-time feel
9. WHEN the web interface renders a visual legal journey, THEN THE System SHALL render a Mermaid.js flowchart inline within the response

---

### Requirement 12: Performance and Response Time

**User Story:** As a user with limited patience and intermittent connectivity, I want the system to respond quickly, so that I can get answers without long waits.

#### Acceptance Criteria

1. WHEN a user submits a situation for rights identification, THEN THE System SHALL begin streaming the response within 3 seconds, and complete the full response within 15 seconds
2. WHEN a user requests document generation, THEN THE System SHALL return the generated document within 30 seconds
3. WHEN a user queries for legal aid centers, THEN THE System SHALL return results within 5 seconds
4. WHEN the Knowledge_Base is queried via RAG, THEN THE System SHALL return retrieval results within 10 seconds
5. WHEN Amazon Polly generates speech audio, THEN THE System SHALL return the audio URL within 5 seconds
6. WHEN Amazon Textract processes an uploaded document image, THEN THE System SHALL return the extracted text and analysis within 15 seconds
7. WHEN the System experiences high load, THEN THE System SHALL maintain response times within acceptable limits using AWS Lambda auto-scaling
8. WHEN the system is under normal load, THEN THE System SHALL support at least 10 simultaneous user sessions

---

### Requirement 13: Error Handling and Graceful Degradation

**User Story:** As a user encountering system errors, I want clear error messages in my language and alternative options, so that I can still get help even when something goes wrong.

#### Acceptance Criteria

1. WHEN the LLM service (Amazon Bedrock) is unavailable, THEN THE System SHALL display an error message in Hindi explaining the issue and suggesting to try again later
2. WHEN the Knowledge_Base query fails, THEN THE System SHALL suggest contacting the nearest DLSA for assistance (with contact info)
3. WHEN document generation fails, THEN THE System SHALL provide a template outline that the user can fill manually
4. WHEN any error occurs, THEN THE System SHALL log the error for debugging without exposing technical details to the user
5. WHEN the System cannot understand user input after 2 clarification attempts, THEN THE System SHALL provide DLSA contact information as a fallback
6. WHEN RAG retrieval returns low-confidence results, THEN THE System SHALL state "I'm not confident about this — please consult a legal aid center" rather than guessing
7. WHEN voice input fails, THEN THE System SHALL silently fall back to text input mode
8. WHEN streaming response delivery fails, THEN THE System SHALL fall back to displaying the full response at once (non-streaming)
9. WHEN Polly TTS fails, THEN THE System SHALL display the text response normally without audio playback
10. WHEN Textract fails to process an uploaded document, THEN THE System SHALL display a message asking the user to try a clearer photo or type the document text manually

---

### Requirement 14: Source Attribution and Transparency

**User Story:** As a user receiving legal guidance, I want to know which specific laws and sections apply to my situation, so that I can verify the information independently and cite it when needed.

#### Acceptance Criteria

1. WHEN the System provides a Rights_Explanation, THEN THE System SHALL cite the specific Legal_Act name for each identified right
2. WHEN the System provides a Rights_Explanation, THEN THE System SHALL cite the specific section number for each identified right
3. WHEN the System generates a Grievance_Document, THEN THE System SHALL include relevant Legal_Act names and section numbers in the document body
4. WHEN multiple sources support a single right, THEN THE System SHALL list all applicable sources
5. WHEN the System cites a source, THEN THE System SHALL format it clearly and consistently (e.g., "Protection of Women from Domestic Violence Act 2005, Section 12")
6. WHEN the System retrieves information via RAG, THEN THE System SHALL only present information that has been grounded in the retrieved legal text (no hallucinated sections)
7. WHEN criminal provisions are cited, THEN THE System SHALL display both old (IPC/CrPC) and new (BNS/BNSS) section numbers with a visual comparison card (see Requirement 23)

---

### Requirement 15: Backend Architecture and AWS Integration

**User Story:** As a system architect, I want the backend to use AWS serverless services with a React frontend, so that the system is scalable, cost-effective, and professionally presented.

#### Acceptance Criteria

1. WHEN a user request is received from the React/Next.js frontend, THEN THE System SHALL route it through Amazon API Gateway (REST API) for non-streaming endpoints, or Lambda Function URLs for the streaming chat endpoint
2. WHEN API Gateway or Lambda Function URL receives a request, THEN THE System SHALL invoke the appropriate AWS Lambda function (Python 3.12 runtime)
3. WHEN Lambda functions need AI capabilities, THEN THE System SHALL invoke Amazon Bedrock (Nova Lite for intake/classification, Claude 3 Sonnet for rights explanation and document generation)
4. WHEN Lambda functions invoke Bedrock, THEN THE System SHALL apply Amazon Bedrock Guardrails for content safety filtering
5. WHEN Lambda functions need RAG capabilities, THEN THE System SHALL query Amazon Bedrock Knowledge Bases backed by Amazon OpenSearch Serverless as the vector store
6. WHEN Knowledge Bases perform vector search, THEN THE System SHALL use Amazon Titan Embeddings V2 for embedding generation
7. WHEN the System needs to store legal corpus PDFs and DLSA directory data, THEN THE System SHALL use Amazon S3
8. WHEN the System needs to generate Hindi speech audio, THEN THE System SHALL use Amazon Polly with the Kajal neural voice (hi-IN)
9. WHEN the System needs to extract text from uploaded document images, THEN THE System SHALL use Amazon Textract
10. WHEN the React/Next.js frontend is deployed, THEN THE System SHALL host it on AWS Amplify with CI/CD from the GitHub repository
11. WHEN Lambda functions execute, THEN THE System SHALL complete within AWS Lambda timeout limits (configured at 120 seconds for Bedrock calls)
12. WHEN the System is monitored, THEN THE System SHALL use Amazon CloudWatch for logging, metrics, and dashboards

---

### Requirement 16: Voice Input via Web Speech API

**User Story:** As a user who finds typing difficult, I want to speak my legal problem in Hindi, so that I can get help without needing to type.

#### Acceptance Criteria

1. WHEN the user clicks the microphone button, THEN THE System SHALL activate the browser's Web Speech API for speech-to-text conversion
2. WHEN voice input is active, THEN THE System SHALL support Hindi (hi-IN) language recognition
3. WHEN voice input is active, THEN THE System SHALL support English (en-IN) language recognition
4. WHEN the Web Speech API successfully transcribes speech, THEN THE System SHALL populate the text input field with the transcribed text
5. WHEN the Web Speech API is not supported by the browser or fails, THEN THE System SHALL gracefully fall back to text-only input without error
6. WHEN voice input is in progress, THEN THE System SHALL display a visual indicator (pulsing microphone icon) showing active recording

---

### Requirement 17: Amazon Polly Hindi Text-to-Speech

**User Story:** As an illiterate user who cannot read text on a screen, I want the system to read my legal rights aloud in Hindi, so that I can understand them through listening.

#### Acceptance Criteria

1. WHEN the user clicks the "🔊 सुनें" (Listen) button on any Rights_Explanation, THEN THE System SHALL generate Hindi speech audio using Amazon Polly
2. WHEN generating speech, THEN THE System SHALL use the Kajal neural voice (hi-IN) for natural-sounding Hindi
3. WHEN generating speech, THEN THE System SHALL use the neural engine for highest quality audio
4. WHEN speech audio is generated, THEN THE System SHALL play it directly in the browser via an HTML5 audio element
5. WHEN speech audio is playing, THEN THE System SHALL display playback controls (play, pause, stop)
6. WHEN the user's language is set to English, THEN THE System SHALL generate English speech audio using an appropriate Polly voice
7. WHEN Polly fails to generate audio, THEN THE System SHALL silently hide the Listen button without displaying an error

---

### Requirement 18: Streaming Responses

**User Story:** As a user waiting for AI-generated legal guidance, I want to see the response appear word-by-word in real time, so that I feel the system is actively working and I can start reading immediately.

#### Acceptance Criteria

1. WHEN the System generates a rights explanation via Bedrock, THEN THE System SHALL stream the response word-by-word using Bedrock's `converse_stream` API
2. WHEN streaming, THEN THE System SHALL deliver response chunks via Lambda Function URLs (not API Gateway REST, which does not support streaming)
3. WHEN the frontend receives streamed chunks, THEN THE System SHALL render them progressively (ChatGPT-style typing effect)
4. WHEN streaming fails mid-response, THEN THE System SHALL display the complete response at once as a fallback (invisible to the user)
5. WHEN a streaming response is complete, THEN THE System SHALL render the full structured UI elements (rights cards, flowcharts, comparison cards) from the completed response

---

### Requirement 19: Document Scanning via Amazon Textract

**User Story:** As a user who received a legal notice or court summons I cannot read, I want to take a photo of it and have the system explain what it says, so that I know whether I need to act and by when.

#### Acceptance Criteria

1. WHEN a user uploads an image of a document, THEN THE System SHALL extract text from the image using Amazon Textract's `detect_document_text` API
2. WHEN text is extracted, THEN THE System SHALL send the extracted text to Amazon Bedrock for legal analysis
3. WHEN the AI analyzes the extracted document, THEN THE System SHALL explain:
   - What type of document it is (court notice, legal notice, eviction order, challan, etc.)
   - Whether it is from a legitimate authority
   - What it is asking the person to do
   - What the deadline to respond is (if any)
   - Whether it is legally binding or an intimidation tactic
   - Step-by-step instructions on what to do next
   - Which legal provisions are cited and what they mean in simple language
4. WHEN the document is in Hindi, THEN THE System SHALL respond in simple Hindi (Class 8 level)
5. WHEN the AI cannot determine the document's authenticity, THEN THE System SHALL clearly state so and recommend consulting a lawyer or DLSA
6. WHEN the upload is initiated on a mobile device, THEN THE System SHALL offer the camera as an input source (via `capture="environment"` attribute)
7. WHEN Textract fails to extract text, THEN THE System SHALL ask the user to try a clearer photo or type out the document text manually

---

### Requirement 20: PDF Document Export

**User Story:** As a user who needs to submit a legal document, I want to download my generated complaint letter or RTI application as a properly formatted PDF, so that I can print it at a Common Service Centre and submit it.

#### Acceptance Criteria

1. WHEN a Grievance_Document has been generated, THEN THE System SHALL provide a "📄 Download PDF" button
2. WHEN the user clicks Download PDF, THEN THE System SHALL generate a formatted PDF using `@react-pdf/renderer`
3. WHEN the PDF is generated, THEN THE System SHALL include:
   - Document header (e.g., "शिकायत पत्र / Complaint Letter")
   - Proper addressing (authority name and designation)
   - Date placeholder
   - Document body with all facts and legal citations
   - Signature block with placeholder
   - Disclaimer footer
4. WHEN the PDF is generated, THEN THE System SHALL use a font that supports Devanagari script for Hindi documents
5. WHEN the PDF is generated, THEN THE System SHALL trigger an automatic download in the user's browser

---

### Requirement 21: Side-by-Side Legal Text Display

**User Story:** As a user who wants to verify the AI's explanation, I want to see the original legal text alongside the simplified explanation, so that I can confirm the information is grounded in actual law.

#### Acceptance Criteria

1. WHEN a Rights_Explanation includes a legal citation, THEN THE System SHALL provide a "Show Source" toggle on each rights card
2. WHEN the user clicks "Show Source", THEN THE System SHALL display a split panel: raw legal text (from RAG retrieval) on the left, simplified explanation on the right
3. WHEN displaying the raw legal text, THEN THE System SHALL show the exact text chunk retrieved from the Knowledge Base
4. WHEN displaying the side-by-side view, THEN THE System SHALL visually differentiate the two panels (different background colors, labels: "📜 Original Legal Text" and "💡 Nyaya Saathi Explanation")

---

### Requirement 22: Visual Legal Journey Map

**User Story:** As a user who needs to take legal action, I want to see a step-by-step visual flowchart of my legal journey, so that I know exactly what to do first, what comes next, and how long each step takes.

#### Acceptance Criteria

1. WHEN the System generates a Rights_Explanation for a `new_situation`, THEN THE System SHALL also generate a Visual_Journey as a Mermaid.js flowchart
2. WHEN the Visual_Journey is generated, THEN THE System SHALL include 4-6 steps showing the legal process from filing to resolution
3. WHEN each step is shown, THEN THE System SHALL include the applicable Act/Section, expected timeline, and any critical deadlines
4. WHEN the Visual_Journey is rendered, THEN THE System SHALL use the Mermaid.js library to render an interactive, styled flowchart in the React frontend
5. WHEN the user's language is Hindi, THEN THE System SHALL generate the flowchart labels in Hindi
6. WHEN the flowchart includes urgent actions, THEN THE System SHALL highlight them visually (e.g., emergency steps in red)

---

### Requirement 23: IPC → BNS Dual Legal Citation

**User Story:** As a user or legal professional referencing legal provisions, I want to see both the old (IPC/CrPC) and new (BNS/BNSS) section numbers, so that I can reference the correct law whether dealing with old court records or filing new cases.

#### Acceptance Criteria

1. WHEN the System cites criminal provisions, THEN THE System SHALL display both the old law reference (IPC/CrPC/Evidence Act) and the new law reference (BNS/BNSS/BSA)
2. WHEN displaying dual citations, THEN THE System SHALL render a visual comparison card with:
   - Old law badge (amber color): "पुराना कानून / Old Law" + section reference
   - New law badge (green color): "नया कानून / New Law" + section reference
   - Brief explanation of what the provision covers
3. WHEN the old and new sections have substantive differences, THEN THE System SHALL note the key changes
4. WHEN only non-criminal provisions apply (e.g., RTI Act, MGNREGA), THEN THE System SHALL cite normally without dual-citation format (these acts have not been replaced)

---

### Requirement 24: Case Strength Assessment

**User Story:** As a user considering legal action, I want to know how strong my case is and what evidence I should collect, so that I can prepare strategically before approaching authorities.

#### Acceptance Criteria

1. WHEN the System completes a Rights_Explanation for a `new_situation`, THEN THE System SHALL also generate a Case_Strength assessment
2. WHEN generating Case_Strength, THEN THE System SHALL evaluate factors including: clarity of the violation described, specificity of facts (dates, locations, actors), evidence mentioned or implied, number of applicable legal provisions
3. WHEN displaying Case_Strength, THEN THE System SHALL show a score from 0-100% with a visual progress bar and a label (Weak/Moderate/Strong/Very Strong)
4. WHEN displaying Case_Strength, THEN THE System SHALL list what the user already HAS in their favor (facts already shared)
5. WHEN displaying Case_Strength, THEN THE System SHALL list MISSING evidence and generate a prioritized evidence checklist (max 7 items) with an estimated impact percentage for each item
6. WHEN evidence collection would improve the case, THEN THE System SHALL explain how much the Case_Strength score would improve (e.g., "Collecting medical records +10%, witness statement +5%")
7. WHEN Case_Strength is below 50%, THEN THE System SHALL recommend consulting DLSA for professional guidance before taking action

---

### Requirement 25: Multi-Path Legal Comparison

**User Story:** As a user facing a legal problem, I want to see all my legal options compared side-by-side with timelines and difficulty levels, so that I can choose the best path forward.

#### Acceptance Criteria

1. WHEN the System completes a Rights_Explanation for a `new_situation`, THEN THE System SHALL identify 2-4 distinct Legal_Paths the user can take
2. WHEN displaying Legal_Paths, THEN THE System SHALL show each path with:
   - Path name (e.g., "Criminal FIR", "Civil Protection Order", "DLSA Mediation")
   - Legal basis (Act + Section)
   - Expected timeline (days/weeks/months/years)
   - Cost to user (usually free for marginalized communities)
   - Difficulty level (1-5 with visual indicator)
   - Likely outcome
   - Whether it can be combined with other paths
3. WHEN multiple paths are available, THEN THE System SHALL mark one as "⭐ RECOMMENDED" based on speed of relief, ease of execution, and effectiveness
4. WHEN paths can be pursued simultaneously, THEN THE System SHALL note this (e.g., "You can pursue Path A + B simultaneously")
5. WHEN displaying paths, THEN THE System SHALL render them as visually distinct comparison cards

---

### Requirement 26: Legal Rights Quiz

**User Story:** As a user who can't articulate my problem, I want to discover my legal rights through an interactive quiz, so that I can learn about protections I didn't know existed.

#### Acceptance Criteria

1. WHEN a user initiates the "Know Your Rights" quiz, THEN THE System SHALL generate 5 True/False or Yes/No legal awareness questions using Amazon Bedrock
2. WHEN generating quiz questions, THEN THE System SHALL focus on common legal misconceptions among marginalized communities (women's rights, worker rights, consumer rights, RTI, free legal aid)
3. WHEN the user answers a question incorrectly, THEN THE System SHALL display the correct answer with the applicable Act and Section
4. WHEN the user answers a question, THEN THE System SHALL provide a "🔍 Tell Me More" button that navigates to the main chatbot with that topic pre-loaded
5. WHEN the user answers a question, THEN THE System SHALL provide a "📝 Draft Complaint" button that navigates to document generation for that topic
6. WHEN the quiz is displayed, THEN THE System SHALL show questions in both Hindi and English based on the user's language preference
7. WHEN the quiz is complete, THEN THE System SHALL display a summary score and recommend exploring specific topics where the user scored poorly

---

## Additional Non-Functional Requirements

### Language Support
- **Primary:** Hindi and English
- **Mixed-language:** Hinglish (mixed Hindi-English) input handling without requiring language switching
- **Voice input:** Hindi and English via browser Web Speech API
- **Voice output:** Hindi and English via Amazon Polly (Kajal neural voice for Hindi)
- **Interface:** All UI labels, system messages, error messages, and disclaimers available in both Hindi and English
- **Extensibility:** Architecture designed to support additional Indian languages (Tamil, Telugu, Marathi, Bengali, Gujarati, Kannada) via Amazon Translate + Bedrock in future phases

### Cost Efficiency
- Uses Amazon Bedrock **pay-per-use** pricing — no dedicated infrastructure
- Optimized prompt engineering to minimize token usage per request
- Knowledge Base uses Amazon S3 (pennies per GB) for document storage
- AWS Lambda billed per invocation — zero cost when idle
- AWS Amplify hosting — pay-per-build + CDN
- Estimated prototype cost: Under ₹5/month for hackathon demo usage

### Reliability
- System uptime target: **99% during hackathon demo**
- Graceful error handling: If Bedrock API fails, system shows a helpful error message in Hindi (not a crash)
- Streaming fallback: If streaming fails, full response appears at once
- Voice fallback: If Web Speech API fails, text input remains available; if Polly fails, text display remains available
- All AWS services selected for high availability within `ap-south-1` (Mumbai) region

### Security
- **No authentication by design:** Anonymous access is a safety feature — a woman using her abuser's phone cannot leave a login trail
- Zero PII storage — sessions are temporary, no database
- All API calls over HTTPS/TLS 1.2+
- Amazon Bedrock Guardrails enforce content safety
- IAM least-privilege access control

---

## User Personas

### Persona 1: Sunita — Rural Woman Facing Domestic Violence
- **Age:** 28 | **Location:** Village in Uttar Pradesh | **Education:** Class 8 | **Language:** Hindi
- **Context:** Sunita's husband beats her regularly. Her in-laws took her gold jewelry (stridhan). She doesn't know she has legal protections or how to get help. She has never spoken to a lawyer.
- **Needs:** Understand her rights under DV Act and Dowry Prohibition Act, get help filing a complaint, find free legal aid at the nearest DLSA, hear her rights read aloud (she struggles to read)
- **Tech Access:** Husband's old Android smartphone, intermittent 2G/3G connectivity
- **Success Scenario:** Sunita speaks in Hindi via voice input → Nyaya Saathi explains her rights under PWDVA Section 17 and IPC 498A / BNS 85 with dual-citation cards → shows a visual flowchart of her legal journey → assesses her case strength at 75% and lists evidence to collect → compares 3 legal paths (criminal FIR vs protection order vs DLSA mediation) → generates a complaint letter PDF → reads it aloud in Hindi via Amazon Polly.

### Persona 2: Ramesh — Daily-Wage Worker with Unpaid Wages
- **Age:** 35 | **Location:** Semi-urban town in Jharkhand | **Education:** Class 10 | **Language:** Hindi
- **Context:** Ramesh worked under MGNREGA for 45 days but received payment for only 20 days. He doesn't know he can file an RTI to check payment records or complain to the block office.
- **Needs:** Understand MGNREGA entitlements, draft an RTI application to check his payment records, know where and how to complain
- **Tech Access:** Basic Android smartphone, Common Service Centre (CSC) access in nearby town
- **Success Scenario:** Ramesh types his problem → Nyaya Saathi explains his right to full payment under MGNREGA Section 3 → generates a ready-to-submit RTI application as a downloadable PDF → provides step-by-step submission instructions → shows 2 legal paths with recommendation.

### Persona 3: Kavita — NGO Fieldworker
- **Age:** 32 | **Location:** Operates across rural Rajasthan | **Education:** Graduate | **Language:** Hindi + English
- **Context:** Kavita works with an NGO helping trafficking survivors and child marriage victims. She needs to quickly identify applicable laws and draft complaints for multiple people every week.
- **Needs:** Quick legal reference tool across multiple acts, efficient document drafting for varied scenarios, reliable source citations for court filings, ability to scan and understand documents received by beneficiaries
- **Tech Access:** Smartphone with 4G, laptop at office
- **Success Scenario:** Kavita uses Textract to scan a court notice received by a beneficiary → the system explains it's a summons, not an eviction → she uses the quiz to identify additional rights the beneficiary may have → generates tailored FIR descriptions with both IPC and BNS sections — cutting her legal research time from hours to minutes.

---

## Success Metrics

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| **Comprehensiveness** | System correctly identifies applicable legal provisions for ≥80% of common scenarios | Test with 20+ scenarios across DV, wage theft, caste discrimination, consumer complaints, property disputes |
| **Usability** | Non-technical user receives actionable output within 3 messages | User testing with target personas |
| **Document Quality** | Auto-generated documents require <20% manual editing before submission | Expert review by legal aid professionals |
| **Response Time** | Streaming begins <3s, rights identification <15s, document generation <30s | Automated performance testing |
| **User Satisfaction** | Target users report that explanations are understandable and helpful | Feedback survey during demo |
| **Voice Accessibility** | Polly TTS correctly reads Hindi rights explanations with natural pronunciation | Audio quality testing with Hindi speakers |
| **Document Scanning** | Textract correctly extracts text from 80%+ of uploaded legal documents | Testing with varied document quality |
| **AWS Service Count** | System uses 14 AWS services correctly | Architecture audit |

---

## Scope Boundaries (What This Project Is NOT)

- **Not a replacement for lawyers:** This is a legal awareness, strategy, and guidance tool, not legal representation. It does not appear in court or negotiate on behalf of users.
- **Not a case management system:** Does not track ongoing court cases, hearing dates, or legal proceedings.
- **Not a legal database search engine:** Users don't search by Act/Section — they describe situations in plain language and the AI identifies applicable laws.
- **Not real-time legal advice:** System provides general legal information based on central legislation, not situation-specific legal opinions accounting for all jurisdictional variations.
- **Not a replacement for DLSA:** When situations are complex, the system actively refers users to District Legal Services Authorities for in-person professional assistance.
- **Not a WhatsApp bot:** WhatsApp integration is Phase 2. The current system is a web application.
- **Not an analytics dashboard:** We do not provide admin panels or usage analytics. CloudWatch handles operational monitoring.

---

## Future Roadmap (Post-Hackathon)

| Phase | Enhancement | Description |
|-------|-------------|-------------|
| **Phase 1** | Voice Input via Transcribe | Upgrade from browser Web Speech API to Amazon Transcribe for more accurate, server-side Hindi ASR |
| **Phase 2** | Regional Languages | Expand to Tamil, Telugu, Marathi, Bengali, Gujarati, Kannada using Amazon Translate + Bedrock multilingual capabilities |
| **Phase 3** | State-Specific Laws | Add state-specific acts and rules (Maharashtra Rent Control, UP Revenue Code, etc.) beyond current central legislation focus |
| **Phase 4** | WhatsApp Integration | Reach users on the messaging platform they already use daily via Amazon Pinpoint or Twilio — no app installation required |
| **Phase 5** | CSC Integration | Deploy as a tool for Common Service Centre (CSC) operators who assist villagers with government services |
| **Phase 6** | NGO Dashboard | Analytics dashboard for NGOs to track common legal issues in their operational areas and generate impact reports |
| **Phase 7** | Offline Mode | Cached common scenarios and rights information for zero-connectivity use in remote areas |
