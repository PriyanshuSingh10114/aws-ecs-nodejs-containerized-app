<h1>AWS EC2 to ECS Deployment using Docker, ECR, IAM, and CloudWatch</h1>
<p>This document explains a brief end-to-end process to deploy a containerized application on AWS using EC2, Docker, Amazon ECR, Amazon ECS, IAM, and CloudWatch.</p>
<h2>Step 1: Create EC2 Instance</h2> 
<p>Create an EC2 instance with the following configuration:</p> 
<p> 
  
    • OS: Ubuntu Linux
  
    • Instance Type: t2.micro 
    
    • Storage: 10 GB 
    
    • Security Group: - SSH (22) – TCP - HTTP (80) – TCP 
</p> 

<h2>Advanced Configuration (User Data Script)</h2> 

<pre>
  
            #!/bin/bash 
        
            sudo apt-get update -y 
        
            sudo apt install nginx -y 
        
            sudo systemctl start nginx 
        
            sudo systemctl enable nginx 
  
</pre>

<h2>Step 2: Connect to EC2 using SSH</h2>

<pre> ssh -i your-key.pem ubuntu@&lt;EC2_PUBLIC_IP&gt; </pre>

<h2>Step 3: Install AWS CLI v2</h2>

<pre> 
  
  sudo apt install unzip -y
  curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" unzip awscliv2.zip 
  sudo ./aws/install aws --version 
  
</pre>

<h2>Step 4: Install Docker</h2>

<pre> sudo apt install docker.io -y sudo systemctl start docker sudo systemctl enable docker </pre> <p>Provide Docker permission to the current user:</p> <pre> sudo chown $USER /var/run/docker.sock </pre>

<h2>Step 5: Clone Application Repository</h2> <pre> git clone git@github.com:&lt;username&gt;/&lt;repo-name&gt;.git cd &lt;repo-name&gt; </pre>

<h2>Step 6: Create IAM User (Minimum Permissions)</h2>

<p>Create an IAM user and attach only the following policies:</p> 

<p>
  
    • AmazonEC2ContainerRegistryFullAccess 
    
    • AmazonECSFullAccess 
    
    • CloudWatchFullAccess
  
</p> 

<p>Create access key and secret key for this user and configure AWS CLI:</p> <pre> aws configure </pre> <p>Region:</p> <pre> ap-south-1 </pre>

<h2>Step 7: Create ECR Repository</h2>

<p>Go to AWS Console → ECR → Create Repository.</p> <p>Copy the push commands shown in the ECR console.</p>

<h2>Step 8: Authenticate Docker to ECR</h2> <pre> aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin &lt;ACCOUNT_ID&gt;.dkr.ecr.ap-south-1.amazonaws.com </pre>

<h2>Step 9: Build and Push Docker Image</h2> 

<pre> 
  docker build -t app-image . 
  docker tag app-image:latest &lt;ECR_URI&gt;:latest docker push &lt;ECR_URI&gt;:latest 
</pre>

<h2>Step 10: Create ECS Cluster</h2> 
<p> 
  
    • Go to ECS → Clusters → Create Cluster 
    • Cluster name: app-cluster 
    • Select a method of obtaining compute capacity of infrastructure advanced 
    • Container Insights with enhanced observability in Monitoring

</p> 
<p>Wait for provisioning to complete.</p>

<h2>Step 11: Create ECS Task Definition</h2>
<p> 
  
      • Launch type: EC2 
      
      • Execution Role: ecsTaskExecutionRole 
      
      • Container image: ECR image URI 
      
      • Port mapping: 80 or application port 
      
      • Log driver: awslogs 
  
</p>

<h2>Step 12: Create ECS Service and Run Task</h2> 

<p> 
  • Select cluster 
  • Create service 
  • Desired tasks: 1 
  • Wait until task state becomes RUNNING 
</p>

<h2>Step 13: Verify Deployment</h2> 

<p>Open browser:</p> <pre> http://&lt;EC2_PUBLIC_IP&gt;:8000 </pre>

<h2>Step 14: View Logs in CloudWatch</h2>
<p> 
  
    • Go to CloudWatch → Log Groups 
    • View application and container logs 
  
</p>

<h2>Conclusion</h2> 
<p>This setup demonstrates a secure and minimal AWS container deployment using EC2, Docker, ECR, ECS, IAM, and CloudWatch with least-privilege access.</p>
