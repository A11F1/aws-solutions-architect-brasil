# Flashcards — Module 21: Resources and Links

> **Format:** Q = Question | A = Answer  
> **Total:** 20 flashcards — resources, strategies, and study guidance

---

**Q:** What is the official AWS site for all certifications and where do you schedule the exam?  
**A:** **aws.amazon.com/certification/** — main certifications page. To schedule: "Schedule Exam" button → redirects to **Pearson VUE** (in-person test center or online proctored). Alternative: register through the SAA-C03 certification page itself. Remember: valid identification is required (government-issued ID or passport for online proctored).

---

**Q:** What are the 3 main paid courses recommended for the SAA-C03?  
**A:** **(1) Stephane Maarek (Udemy):** Ultimate AWS Certified Solutions Architect Associate — most popular, 30h+, up to date, exam-focused. **(2) Neal Davis (Digital Cloud Training):** AWS SAA-C03 — clear explanations, good for beginners. **(3) Adrian Cantrill:** superior technical depth, harder, anime style — for those who want to truly understand. All available on Udemy, often at a discount.

---

**Q:** What is the most important official AWS resource for real practice questions?  
**A:** **AWS Skill Builder** (skillbuilder.aws): **(1)** "AWS Certified Solutions Architect – Associate Official Practice Question Set" — 20 free official questions. **(2)** "Exam Prep Official Practice Exam: AWS SAA-C03" — 65 paid questions, with detailed feedback. Credentials: a free Skill Builder account. The official questions give the real difficulty level of the exam.

---

**Q:** What is Tutorials Dojo and why is it highly recommended?  
**A:** **tutorialsdojo.com** — practice exam platform created by Jon Bonso. Highlights: explanations for EVERY option of every question (why the right one is right + why each wrong one is wrong); cheat sheets per service; timed and review modes; high correlation with the real exam. Cost: ~$15-30 per practice exam. Widely cited as the best practice resource besides the official one.

---

**Q:** Which AWS whitepaper is most important for the SAA-C03 and where do you find it?  
**A:** **"AWS Well-Architected Framework"** — aws.amazon.com/architecture/well-architected. Describes the 6 pillars with design principles. Other essentials: **(1)** "Architecting for the Cloud: AWS Best Practices". **(2)** "AWS Security Best Practices". **(3)** "Disaster Recovery of Workloads on AWS". All free at aws.amazon.com/whitepapers.

---

**Q:** How do you use the AWS Architecture Center to study?  
**A:** **aws.amazon.com/architecture** — library of Reference Architectures by use case (web apps, serverless, data lake, microservices) and industry. To study: open diagrams of architectures relevant to the exam (3-tier web app, serverless API, data lake), and understand which services are combined and why. Official AWS reference architectures.

---

**Q:** What is the official tool for estimating the cost of an AWS architecture?  
**A:** **AWS Pricing Calculator** (calculator.aws) — free. Select services, configure parameters (Region, instance type, expected usage in hours/GB/requests), add them to the estimate, and share the estimate via URL. Unlike Cost Explorer (for costs already incurred). On the exam: "estimate costs before implementing" → Pricing Calculator.

---

**Q:** Where do you find the AWS service FAQs and how do you use them to study?  
**A:** On each AWS service page: `aws.amazon.com/[service]/faqs/`. E.g.: `aws.amazon.com/s3/faqs/`, `aws.amazon.com/ec2/faqs/`. FAQs explain limits, edge cases, and specific behaviors that appear on the exam. Reading priority: EC2, S3, RDS, IAM, VPC, Lambda, DynamoDB — the services with the greatest weight on the exam.

---

**Q:** What is the difference between Pearson VUE and PSI for scheduling the AWS exam?  
**A:** Both are exam providers approved by AWS. **Pearson VUE:** more popular, more test centers, online proctored option (OnVUE). **PSI:** alternative, fewer centers, also has online proctored. For online proctored: you need a webcam, microphone, a quiet environment with no other people, and no objects on the desk. The exam is available in several languages, including Portuguese (but some technical questions remain in English).

---

**Q:** What should you study in the final week before the SAA-C03 exam?  
**A:** Last week: **(1)** Review mistakes from practice exams (focus on what you got wrong, not what you already know). **(2)** Cheatsheets per service (numeric limits, comparisons). **(3)** Domain 3 (Security, 30%) — IAM conditions, encryption, network security. **(4)** A full practice exam (65q) to measure readiness. **(5)** Don't study anything new in the last 2 days — review + rest. Score 750+ on practice exams = ready for the exam.

---

**Q:** Which YouTube channel has the best free summaries of AWS services for the exam?  
**A:** **(1) Be a Better Dev** — practical and conceptual demos. **(2) Stephane Maarek** — course snippets. **(3) Digital Cloud Training (Neal Davis)** — detailed tutorials. **(4) AWS Events** — technical re:Invent talks (advanced). **(5) TechWorld with Nana** — complementary Docker/K8s. For Brazilian Portuguese content: search "SAA-C03 preparação" — a growing Brazilian community.

---

**Q:** What is the AWS Free Tier for in hands-on study?  
**A:** Practicing labs at no cost. Essential Always Free resources: Lambda 1M requests/month, DynamoDB 25GB, CloudFront 1TB/month, SNS 1M requests, API Gateway 1M requests/month. 12-month free: EC2 t2/t3.micro 750h/month, S3 5GB, RDS db.t2.micro 750h/month. **ALWAYS:** configure a Budget Alert in the Billing Console to warn you if cost > $1. Clean up after labs (stop/delete resources).

---

**Q:** How do you use AWS Skill Builder for free to prepare?  
**A:** Create a free account at skillbuilder.aws. Access: **(1)** "AWS Cloud Practitioner Essentials" (basic free course). **(2)** "Architecting on AWS" (concepts). **(3)** "AWS Certified Solutions Architect - Associate" learning path (free). **(4)** "Official Practice Question Set SAA-C03" (20 free questions). The paid subscription ($29/month) gives access to full practice exams and labs.

---

**Q:** What is the AWS Certification Discord and how do you benefit from it?  
**A:** Online communities where candidates share exam experiences, resources, and tips. AWS: join servers like "AWS Study Group", "r/AWSCertifications" (Reddit), "Discord AWS Brasil". Benefits: exam update tips, notifications when questions have changed, sharing of free resources, support from people who passed recently.

---

**Q:** How do you set up a local study environment for labs (AWS CLI + Terraform)?  
**A:** **(1)** Install AWS CLI v2: `winget install Amazon.AWSCLI` (Windows) or `brew install awscli` (Mac). **(2)** Configure: `aws configure` (access key + secret + Region + format). **(3)** Install Terraform: `winget install Hashicorp.Terraform`. **(4)** Verify: `aws sts get-caller-identity` returns account info. Use AWS Cloud9 as an alternative — an in-browser environment with no local setup.

---

**Q:** Which book is recommended to complement SAA-C03 study?  
**A:** **"AWS Certified Solutions Architect Study Guide" by Ben Piper and David Clinton** (Sybex/Wiley) — updated for SAA-C03, methodical approach, good complementary pre-exam read. **"AWS Cookbook" by John Culkin** — more hands-on, practical recipes. Books are complementary (not substitutes) to practice exams. Priority: practice exams > books.

---

**Q:** What is AWS "Exam Readiness" and how do you interpret it?  
**A:** AWS Skill Builder has "Exam Readiness: SAA-C03" courses that map the domains, explain question types, and teach elimination strategies. Also: a score of 800+ on Tutorials Dojo practice exams in timed mode is generally considered sufficient readiness. Below 720 (80% equivalent on practice exams): keep studying the weak points you identified.

---

**Q:** Where do you read about new AWS services and updates that may appear on the exam?  
**A:** **(1) AWS What's New:** aws.amazon.com/new — updates feed. **(2) AWS Blog:** aws.amazon.com/blogs/aws — technical posts. **(3) re:Invent announcements:** YouTube/AWS Events. **(4) AWS Certification changelog:** AWS announces exam version changes 6 months in advance. Focus: services that entered the official Exam Guide. Don't study out-of-scope services.

---

**Q:** How do you create a 6-week study plan for the SAA-C03?  
**A:** **Week 1:** Fundamental modules (EC2, S3, IAM, VPC). **Week 2:** Databases + advanced Networking (RDS, DynamoDB, Route 53, CloudFront). **Week 3:** Serverless + Decoupling (Lambda, API GW, SQS, SNS, EventBridge). **Week 4:** Analytics + Migration + Well-Architected. **Week 5:** Full practice exams (2-3 full exams) + review mistakes. **Week 6:** Focus on weak points + final practice exam + rest.

---

**Q:** How do you use the Tutorials Dojo cheat sheets for final review?  
**A:** Tutorialsdojo.com/aws-cheat-sheets/ — available per service, free. Format: comparison tables, important numeric limits, differences between similar services. For review: print or save offline the cheat sheets of the core services (EC2, S3, RDS/Aurora, VPC, IAM, Lambda, DynamoDB). Especially review the comparison tables that the exam frequently tests (e.g., SQS vs SNS vs EventBridge).

---

**Q:** After earning the SAA-C03 certification, how do you keep it valid?  
**A:** Validity: **3 years**. To recertify: **(1)** Retake the SAA-C03 before it expires. **(2)** Earn a higher-level certification (SAP-C02 Professional) — automatically recertifies the SAA. **(3)** Earning another Associate or specialty (does not recertify automatically). In your AWS Certification profile: you can download the digital certificate and Credly badge. Share it on LinkedIn.
