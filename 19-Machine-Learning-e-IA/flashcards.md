# Flashcards — Module 13: Machine Learning and AI

> **Format:** Q = Question | A = Answer  
> **Total:** 25 flashcards

---

**Q:** What are the 3 layers of the AWS AI/ML framework?  
**A:** **(1) AI Services (top):** ready-made APIs with no ML knowledge — Rekognition, Comprehend, Polly, Transcribe, Translate, Lex, Textract, Kendra, Bedrock. **(2) ML Services (middle):** SageMaker (custom build-train-deploy). **(3) Infrastructure (base):** EC2 GPU, Trainium, Inferentia for training/inference at scale.

---

**Q:** What is Amazon Bedrock and how does it differ from SageMaker?  
**A:** **Bedrock:** access to third-party Foundation Models (LLMs) via API (Anthropic Claude, Amazon Titan, Meta Llama, Mistral, Cohere, Stability AI) without training a model. **SageMaker:** platform to train, tune, and deploy your own ML models. Bedrock = use ready-made models; SageMaker = build custom models.

---

**Q:** What is RAG in the context of Amazon Bedrock?  
**A:** Retrieval Augmented Generation: a technique that connects LLMs to an external knowledge base. Flow: user asks → system retrieves relevant context from the knowledge base → sends context + question to the LLM → LLM generates an answer grounded in current data. Bedrock Knowledge Bases implements RAG with Amazon OpenSearch Serverless as the vector store.

---

**Q:** For which use case should you use Amazon Rekognition?  
**A:** **Image and video analysis:** object/scene/text detection, facial recognition (comparison, search in a collection), inappropriate content detection (moderation), PPE analysis, celebrity detection, streaming or batch video analysis. Requires no ML knowledge.

---

**Q:** What is the difference between Amazon Transcribe and Amazon Polly?  
**A:** **Transcribe:** Speech-to-Text (STT) — converts audio to text. Features: diarization (identify who is speaking), custom vocabulary, PII redaction, automatic language identification. **Polly:** Text-to-Speech (TTS) — converts text to audio. Features: neural voices (NTTS), SSML for pronunciation control, custom lexicons.

---

**Q:** What is Amazon Comprehend and what are its main analyses?  
**A:** NLP service for text analysis without ML code. Analyses: **Entities** (people, organizations, places), **Key Phrases**, **Sentiment** (positive/negative/neutral/mixed), **Language Detection**, **PII Detection/Redaction**, **Topic Modeling** (group documents by latent topics). Comprehend Medical: specialized version for clinical texts.

---

**Q:** What is Amazon Lex v2 and what is it used for?  
**A:** Managed service for building voice and text **chatbots and virtual assistants**. Uses ASR (Automatic Speech Recognition) + NLU (Natural Language Understanding). Intents = actions; Slots = parameters to collect. The foundation of Alexa. Use cases: IVR, customer service bots, help desk assistants. No ML knowledge required.

---

**Q:** What is the difference between Amazon Kendra and Amazon OpenSearch?  
**A:** **Kendra:** intelligent search for corporate documents using ML and NLP. Understands natural language, semantic search, answers questions directly. Input: PDFs, Word, SharePoint, Salesforce, S3. **OpenSearch:** technical search and analytics (logs, metrics, e-commerce search, full-text search). Kendra = intelligent for humans; OpenSearch = technical/full-text.

---

**Q:** For which scenario should you use Amazon SageMaker Ground Truth?  
**A:** **Data labeling** for training ML models. Flow: humans (crowdworkers via Amazon Mechanical Turk or a private workforce) annotate/label images, texts, videos. Active Learning: partially trained models automatically label high-confidence data, reducing human labeling cost by up to 70%.

---

**Q:** What is Amazon Textract and how does it differ from simple OCR?  
**A:** **Textract:** extracts text + **structure** from scanned documents. Goes beyond OCR: detects Tables (extracts headers and cells), Forms (extracts field:value from forms), Queries (answers specific questions about the document). Supports multi-page PDFs, IDs (passports, driver's licenses), medical documents.

---

**Q:** What is Amazon Fraud Detector?  
**A:** Managed fraud detection service using ML. Models pre-trained on AWS fraud data + the customer's historical data. Detection events: new account registrations, online payments, guest checkout. Returns a fraud score. No ML expertise required.

---

**Q:** What are the stages of the SageMaker ML pipeline?  
**A:** **(1) Data prep:** SageMaker Data Wrangler / Processing Jobs. **(2) Training:** Training Jobs (EC2 instances). **(3) Tuning:** Hyperparameter Optimization (HPO). **(4) Evaluation:** Experiments. **(5) Registry:** Model Registry. **(6) Deploy:** SageMaker Endpoints (online) / Batch Transform (offline). **(7) Monitor:** Model Monitor.

---

**Q:** What is Amazon SageMaker Feature Store?  
**A:** Centralized repository of features (input variables) for ML models. Allows: sharing features across teams and models, reusing computed features, consistency between training and inference (same calculation). **Online Store:** low latency for real-time inference. **Offline Store:** history via S3 for training.

---

**Q:** What is Amazon Forecast and what problem does it solve?  
**A:** Managed **time series forecasting** service using DeepAR+ (a proprietary Amazon algorithm). Input: historical time series data (sales, demand, traffic) + metadata (promotions, holidays). Output: future forecasts with confidence intervals. Use cases: demand planning, supply chain, capacity planning.

---

**Q:** What is Amazon Personalize and what type of recommendation is it used for?  
**A:** Managed **personalized recommendations** service (the same one used on Amazon.com/Prime Video). Input: user-item interaction data (clicks, purchases, plays), item/user metadata. Output: real-time recommendations by userId. Use cases: "other users also bought", "continue watching", "recommended products".

---

**Q:** What is Amazon Lookout for Metrics?  
**A:** Automatic **time series anomaly detection** using ML. Input: business metrics (revenue, DAUs, conversion rate) from S3, RDS, Redshift, Salesforce, etc. Automatically detects significant deviations. Sends alerts via SNS. No ML knowledge required. Different from CloudWatch Anomaly Detection (technical infrastructure metrics).

---

**Q:** What is Amazon SageMaker JumpStart?  
**A:** Hub of pre-trained models and ready-made ML solutions for one-click deployment. Contains: Foundation Models (Llama, Mistral, Falcon), vision models (ResNet, EfficientNet), NLP (BERT, RoBERTa), industry solutions (demand forecast, fraud). Speeds up the start of ML projects without starting from scratch.

---

**Q:** What is Amazon Bedrock Guardrails?  
**A:** Safety controls for applications with Foundation Models. Features: content filters (violence, hate speech, sexual), topic denial (block prohibited topics — e.g., competitors), PII redaction, word filters. Applied automatically to the user input AND the model response.

---

**Q:** For which scenario should you use Amazon Translate with Custom Terminology?  
**A:** When there are company/product-specific terms that must not be translated automatically (product names, corporate slang, acronyms). Custom Terminology: CSV with source→target pairs. E.g., "Amazon Aurora" → "Amazon Aurora" (do not translate), "order" → "pedido" (specific translation). Maintains terminological consistency across translations.

---

**Q:** What is Amazon SageMaker Canvas?  
**A:** **No-code** interface for ML. Business professionals without programming experience can train ML models (classification, regression, time series forecasting, image analysis) through a visual interface. Input: CSV uploads, S3. Uses AutoML under the hood. Shares models with data scientists in SageMaker Studio.

---

**Q:** What is Amazon Rekognition Content Moderation?  
**A:** API that detects inappropriate or adult content in images and videos. Returns hierarchical labels with a confidence score (e.g., "Explicit Nudity" > "Nudity", confidence: 99.5%). Use cases: UGC (User Generated Content) platforms, social networks, marketplaces. Companies define a confidence threshold for automatic moderation.

---

**Q:** What is Amazon Transcribe Call Analytics?  
**A:** Specialized version of Transcribe for call center call analysis. Beyond transcription: detects the **sentiment** of each conversation turn (customer vs. agent), identifies custom categories (complaint, cancellation), extracts entities (account numbers, dates), measures silence time and interruptions.

---

**Q:** When should you use Amazon Comprehend Medical?  
**A:** Extracts medical entities from clinical text: **diagnoses** (ICD-10 codes), **medications** (dosage, frequency, route of administration), **medical procedures**, **anatomy**, **vital signs**. Detects relationships (medication → dosage). HIPAA eligible. Use cases: NLP on electronic health records, medical coding, clinical research.

---

**Q:** What is Amazon Polly Neural Text-to-Speech (NTTS)?  
**A:** Very high-quality synthetic voices using deep learning — practically indistinguishable from a human voice. More natural than standard voices (concatenation synthesis). Supports **SSML** (Speech Synthesis Markup Language) to control pauses, emphasis, pronunciation. Use cases: IVR, audiobooks, accessibility, e-learning.

---

**Q:** What is the responsibility model in Amazon Bedrock (Security)?  
**A:** **AWS is responsible for:** infrastructure, the base model, the FM training data. **Customer is responsible for:** data sent to the model (prompts/context), fine-tuning data, the application and generated responses. Prompts and responses are **not used to train AWS models** (private customer data). Bedrock is HIPAA/SOC/ISO eligible.
