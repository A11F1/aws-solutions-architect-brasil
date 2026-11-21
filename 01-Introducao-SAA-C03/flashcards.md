# Flashcards — Introduction to SAA-C03

---
**Q:** Which domain has the highest weight on the SAA-C03?  
**A:** Design Resilient Architectures, at 30%.

---
**Q:** Which domain has the lowest formal weight?  
**A:** Design Cost-Optimized Architectures, at 18%.

---
**Q:** Does the SAA-C03 test more of the service catalog or architectural decisions?  
**A:** Architectural decisions with context and trade-offs.

---
**Q:** What most differentiates the SAA-C03 from the CLF-C02?  
**A:** The depth of technical choices and the justification between plausible alternatives.

---
**Q:** What does "Least operational overhead" usually point to?  
**A:** Managed services, if they meet the requirement.

---
**Q:** Does Multi-AZ replace Multi-Region?  
**A:** No. Multi-AZ handles AZ failures; Multi-Region handles regional disaster.

---
**Q:** CloudFront vs Global Accelerator in one sentence?  
**A:** CloudFront is a CDN with caching; Global Accelerator accelerates global traffic without being an edge cache.

---
**Q:** RDS vs DynamoDB in initial reasoning?  
**A:** RDS for relational/transactional; DynamoDB for key-value/document with horizontal scale.

---
**Q:** SQS vs SNS in one sentence?  
**A:** SQS decouples with a queue; SNS distributes messages to multiple subscribers.

---
**Q:** Secrets Manager vs Parameter Store?  
**A:** Secrets Manager is stronger for secrets with native rotation.

---
**Q:** What does CloudTrail record?  
**A:** API calls and audit events in the AWS account.

---
**Q:** Which block of services appears in a large part of the exam?  
**A:** EC2, ELB, ASG, S3, VPC, IAM, RDS, DynamoDB, Lambda, Route 53, and KMS.

---
**Q:** What does "Minimal changes to the application" indicate?  
**A:** Avoid alternatives that require major refactoring.

---
**Q:** What does near real-time analytics tend to point to?  
**A:** Streaming or event-driven processing, not late batch.

---
**Q:** Does the exam prefer more control or less operations?  
**A:** Less operations, as long as the functional and non-functional requirements are met.

---
**Q:** What does the exam measure beyond technical knowledge?  
**A:** The ability to read context and prioritize the right trade-offs.

---
**Q:** What is the recommended default region for this repository's labs?  
**A:** us-east-1.

---
**Q:** What does "cost-effective" mean in an AWS question?  
**A:** The best balance between total cost and the requirement, not just the lowest raw price.

---
**Q:** What is a common mistake when preparing for the SAA-C03?  
**A:** Studying overly rare services and neglecting the core blocks.

---
**Q:** How can you best review over 8 weeks?  
**A:** Alternate dense theory, questions, short review, and practice exams in the final stretch.

---
**Q:** What should you do when two alternatives seem correct?  
**A:** Re-read the main requirement, constraints, and qualifying words in the question statement.
