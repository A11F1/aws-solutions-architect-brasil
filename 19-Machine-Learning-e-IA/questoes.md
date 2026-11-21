# Questions — Module 13: Machine Learning and Artificial Intelligence

> **SAA-C03 Domain**: Design High-Performing Architectures  
> **Difficulty**: Medium

---

**1.** An e-commerce company wants to display personalized recommended products to each user based on their purchase history, without having an ML team. Which AWS service should it use?

- A) Amazon SageMaker with a collaborative filtering algorithm
- B) Amazon Personalize (managed recommendation service)
- C) Amazon Forecast
- D) Amazon Comprehend

<details><summary>Answer</summary>

**B** — Amazon Personalize: fully managed personalized recommendations without ML expertise. You provide historical interaction data (views, purchases), the service trains the model and provides a real-time recommendation API. Same engine as Amazon.com.

</details>

---

**2.** A media company needs to automatically moderate images uploaded by users, identifying adult and violent content. Which AWS service should it use?

- A) Amazon Rekognition Content Moderation
- B) Amazon Comprehend for image analysis
- C) Amazon Textract to extract text from the images
- D) Amazon SageMaker with a custom classification model

<details><summary>Answer</summary>

**A** — Amazon Rekognition detects inappropriate content (nudity, violence, drugs) in images and videos with a single API call. It returns labels with confidence scores and hierarchical moderation categories. No need to train a custom model.

</details>

---

**3.** A call center receives 10,000 calls per day. The company wants to analyze customer sentiment and detect keywords in all calls. Which combination of AWS services should it use?

- A) Amazon Rekognition + Amazon Comprehend
- B) Amazon Transcribe Call Analytics + Amazon Comprehend
- C) Amazon Lex + Amazon Polly
- D) Amazon Connect + Amazon Kendra

<details><summary>Answer</summary>

**B** — Transcribe Call Analytics: STT specialized for call centers (identifies speaker, silences, interruptions, sentiment per speaker). The transcript is sent to Comprehend for entity, sentiment, and keyword analysis. A complete call analysis pipeline.

</details>

---

**4.** A financial company needs to forecast demand for a product over the next 30 days based on 3 years of historical sales data, including external variables (holidays, promotions). Which service should it use?

- A) Amazon Personalize
- B) Amazon Forecast
- C) Amazon SageMaker with Autopilot
- D) Amazon Lookout for Metrics

<details><summary>Answer</summary>

**B** — Amazon Forecast: managed time series forecasting. Accepts historical data and "related time series" (external variables such as holidays, promotions, temperature). Simpler than training a model in SageMaker for demand/inventory forecasting use cases.

</details>

---

**5.** A company wants to implement a chatbot on its website to answer frequently asked questions. The chatbot must understand natural language and integrate with back-end systems via Lambda. Which service should it use?

- A) Amazon Polly
- B) Amazon Lex v2 with Lambda fulfillment
- C) Amazon Comprehend
- D) Amazon Kendra

<details><summary>Answer</summary>

**B** — Amazon Lex v2: builds conversational chatbots (same technology as Alexa). You define Intents (actions), Slots (data to collect), Utterances (phrases). For each intent, a fulfillment Lambda runs the business logic (query a DB, create an order, etc.) and returns the response.

</details>

---

**6.** A healthcare company needs to automatically extract structured data (diagnoses, medications, dosages) from medical reports in PDF. Which service should it use?

- A) Amazon Rekognition
- B) Amazon Comprehend Medical + Amazon Textract
- C) Amazon Transcribe Medical
- D) Amazon Kendra

<details><summary>Answer</summary>

**B** — Textract: extracts text and structure (tables, forms) from PDFs/images. Comprehend Medical: analyzes the extracted medical text and identifies clinical entities (diagnoses, medications, dosages), conditions, and relationships between them. Pipeline: Textract → text → Comprehend Medical → structured data.

</details>

---

**7.** A logistics company has a vehicle fleet and wants to automatically detect when a driver shows signs of fatigue (closed eyes, yawning) using onboard cameras. Which service should it use?

- A) Amazon Rekognition Video with streaming face analysis
- B) Amazon Lookout for Vision
- C) Amazon SageMaker with a custom detection model
- D) Amazon Comprehend

<details><summary>Answer</summary>

**A** — Rekognition Video: analyzes real-time video streams (via Kinesis Video Streams). Face Analysis detects attributes such as eyes open/closed and emotional state. For real-time fatigue detection, Rekognition Video + SNS alerts is the simplest solution.

</details>

---

**8.** A Data Science team needs to train a classification model with 100 GB of data in S3, automatically optimizing the hyperparameters, without managing the training infrastructure. Which service and feature should it use?

- A) SageMaker Training Job with Hyperparameter Tuning Job
- B) Amazon Forecast with AutoML
- C) SageMaker Autopilot
- D) AWS Glue ML Transforms

<details><summary>Answer</summary>

**A** — SageMaker Training Job: trains the model on managed instances with your S3 data. Hyperparameter Tuning Job (HPO): runs multiple training jobs in parallel with different hyperparameter combinations using Bayesian Optimization. Autopilot (C) is more automatic but less controllable.

</details>

---

**9.** A company wants to use a large language model (LLM) to generate document summaries, without training the model from scratch and without managing GPUs. Which service should it use?

- A) Amazon SageMaker with a HuggingFace model
- B) Amazon Bedrock with Foundation Models via API
- C) Amazon Comprehend with sentiment analysis
- D) Amazon Kendra

<details><summary>Answer</summary>

**B** — Amazon Bedrock: access to Foundation Models (Claude, Titan, etc.) via a serverless API, without provisioning or managing GPU instances. Pay per token processed. For LLM summarization, it is the simplest and most cost-effective path compared to managing SageMaker endpoints with large models.

</details>

---

**10.** A company wants to implement an enterprise search system in which employees can ask questions in natural language and receive precise answers from internal documents (PDFs, Word, wikis). Which service should it use?

- A) Amazon OpenSearch Service
- B) Amazon Kendra with content connectors
- C) Amazon Comprehend Entity Recognition
- D) Amazon Lex

<details><summary>Answer</summary>

**B** — Amazon Kendra: enterprise search with NLP that answers natural language questions from corporate documents. Native connectors for SharePoint, S3, Confluence, ServiceNow, etc. Unlike OpenSearch (keyword/relevance search), Kendra understands the intent of the question.

</details>

---

**11.** A company wants to automatically detect anomalies in its business metrics (revenue, conversion, NPS) without configuring manual thresholds. Which service should it use?

- A) CloudWatch Anomaly Detection
- B) Amazon Lookout for Metrics
- C) Amazon Forecast
- D) Amazon Comprehend

<details><summary>Answer</summary>

**B** — Amazon Lookout for Metrics: detects anomalies in business metrics (not just infrastructure like CloudWatch) using ML. Connects to RDS, Redshift, S3, CloudWatch, SaaS (Salesforce, Marketo). Sends alerts with context (why the anomaly occurred, which dimensions were affected). CloudWatch Anomaly Detection (A) is for technical infrastructure metrics.

</details>

---

**12.** A developer wants to ensure that biased training data for ML (under-representation of gender or ethnicity) is identified before the model is trained. Which service should it use?

- A) Amazon Rekognition Face Analysis
- B) SageMaker Clarify for bias detection in the data
- C) Amazon Macie for sensitive data analysis
- D) Amazon GuardDuty

<details><summary>Answer</summary>

**B** — SageMaker Clarify: analyzes datasets and models to detect bias (pre-training and post-training). Measures bias metrics such as DPPL (Difference in Positive Proportions in Labels) and CI (Class Imbalance), and generates reports. It also offers explainability (SHAP values) to explain model predictions.

</details>

---

**13.** A company wants to automatically transcribe recorded video meetings, identifying who spoke each segment. Which service and feature should it use?

- A) Amazon Transcribe with Speaker Diarization enabled
- B) Amazon Rekognition to identify faces in the videos
- C) Amazon Connect for meeting recordings
- D) Amazon Comprehend to identify speakers

<details><summary>Answer</summary>

**A** — Amazon Transcribe with Speaker Diarization (ShowSpeakerLabels = true): automatically identifies different speakers in the audio and labels each transcribed segment with the speaker ID (Speaker_0, Speaker_1, etc.). Supports up to 10 speakers per transcription.

</details>

---

**14.** A company wants to automatically translate documents from English into 20 different languages, respecting industry-specific technical terminology. Which service and feature should it use?

- A) Amazon Polly with Multi-Language support
- B) Amazon Translate with Custom Terminology
- C) Amazon Comprehend with Detect Language
- D) Amazon Bedrock with Claude for contextual translation

<details><summary>Answer</summary>

**B** — Amazon Translate with Custom Terminology: you provide a glossary of terms that must be translated in a specific way (e.g., "load balancer" → always "balanceador de carga" in Portuguese, never "equilibrador"). Ensures consistency of technical terminology across all translations.

</details>

---

**15.** A company trains SageMaker models with training jobs that take 8 hours each. Costs are high. What is the most effective strategy to reduce training costs?

- A) Use smaller instances to reduce the hourly cost
- B) Enable SageMaker Managed Spot Training with checkpoints in S3
- C) Train locally in AWS Cloud9
- D) Use SageMaker Serverless Inference instead of training

<details><summary>Answer</summary>

**B** — Spot Training: SageMaker can use Spot Instances (up to 90% discount) for training jobs. Since Spot can be interrupted, checkpoints saved in S3 allow training to resume from where it stopped. For long 8-hour jobs, savings can be 70-90% with little additional complexity.

</details>
