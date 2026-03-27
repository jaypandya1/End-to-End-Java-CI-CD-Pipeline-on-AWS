# End-to-End Java CI/CD Pipeline on AWS

This repository contains a fully automated CI/CD pipeline for a Java application (vprofile), utilizing a modern DevOps stack. The pipeline automates the journey from code commit to a production-ready deployment on **Amazon ECS (Elastic Container Service)**.



## 🛠 Tech Stack
* **CI/CD:** Jenkins (Pipeline as Code)
* **Build Tool:** Maven
* **Code Quality:** SonarQube & Checkstyle
* **Containerization:** Docker (Multistage builds)
* **Artifact Registry:** Amazon ECR
* **Orchestration:** Amazon ECS (Fargate/EC2)
* **Notifications:** Slack

graph LR
    Dev([Developer]) -- Git Push --> GitHub((GitHub))
    GitHub -- Trigger --> Jenkins((Jenkins))
    
    subgraph CI/CD Pipeline
        Fetch[Fetch Code<br/>Git] --> UT[Unit Test<br/>Maven]
        UT --> CS[Checkstyle<br/>Maven]
        CS --> CA[Code Analysis<br/>SonarQube]
        CA -- Quality Gate --> DB[Docker Build<br/>Artifacts]
    end
    
    Jenkins -. Orchestrates .-> Fetch
    DB -- Push Image --> ECR[(Amazon ECR)]
    ECR -- Deploy --> ECS[Amazon ECS]

    classDef tool fill:#f9f9f9,stroke:#333,stroke-width:2px;
    class GitHub,Jenkins,ECR tool;


## 🏗 Pipeline Stages
1.  **Fetch Code:** Pulls the latest source code from GitHub.
2.  **Build & Test:** Compiles the Java application and runs Unit Tests using Maven.
3.  **Static Analysis:** * **Checkstyle:** Validates coding standards.
    * **SonarQube:** Performs deep security and quality scans.
4.  **Quality Gate:** Pauses the pipeline to ensure the code meets the defined "Pass" criteria.
5.  **Dockerization:** Builds a lightweight production image using a multistage Dockerfile.
6.  **Push to ECR:** Tags and pushes the image to Amazon Elastic Container Registry.
7.  **Cleanup:** Removes local images to save Jenkins node disk space.
8.  **Deploy:** Triggers a rolling update on the ECS Cluster to deploy the new image.

   <img width="1440" height="900" alt="Screenshot 2026-03-27 at 11 55 33 AM" src="https://github.com/user-attachments/assets/42a0009f-8912-44b1-96b6-76a4b5bd5e54" />


## 🚀 How to Use
1.  **Jenkins Setup:** * Install plugins: `Pipeline`, `Docker Pipeline`, `Amazon Web Services SDK`, `SonarQube Scanner`, `Slack Notification`.
    * Configure Tools: Add `MAVEN3.9`, `JDK17`, and `sonar6.2` in Global Tool Configuration.
2.  **Credentials:**
    * `awscreds`: AWS IAM User credentials with ECR and ECS permissions.
    * `sonarserver`: SonarQube authentication token.
3.  **Infrastructure:** Ensure your ECS Cluster (`vprofile`) and Service are pre-configured in the `us-east-1` region.

## 📊 Monitoring
* **SonarQube:** View detailed reports at your SonarQube Dashboard.
<img width="2880" height="1800" alt="image" src="https://github.com/user-attachments/assets/f060af5d-ce12-490d-9459-2ae3700b985c" />

* **Slack:** Real-time build status updates are sent to the `#help-jenkins` channel.
  <img width="1440" height="900" alt="Screenshot 2026-03-27 at 11 53 42 AM" src="https://github.com/user-attachments/assets/f975f28b-e7fa-418a-a5b0-a719f3e14e27" />
