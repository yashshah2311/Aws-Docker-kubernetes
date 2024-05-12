# Aws-Docker-kubernetes

Deployment Phase

1. Connect to EC2 Instance: Use SSH to connect to your EC2 instance from your local machine. You'll perform the following steps on this instance.

2. Install Node.js: Update package repositories and install Node.js and npm on your EC2 instance.

        sudo apt update
        sudo apt install nodejs npm
3. Clone your repository: 

        git clone <repository-url>

4. Install Dependencies: Navigate to your application directory and install dependencies using npm or yarn.

       cd <your-app-directory>
       npm install

5. Configure Environment Variables: Set up environment variables for your application, including Firebase configuration.

        export FIREBASE_API_KEY="your-api-key"
        export FIREBASE_AUTH_DOMAIN="your-auth-domain"
        export FIREBASE_DATABASE_URL="your-database-url"
        export FIREBASE_PROJECT_ID="your-project-id"
        export FIREBASE_STORAGE_BUCKET="your-storage-bucket"
        export FIREBASE_MESSAGING_SENDER_ID="your-messaging-sender-id"
        export FIREBASE_APP_ID="your-app-id"

6. Run Your Application: Start your Node.js application using a process manager like PM2 to keep it running 
in the background.

       pm2 start <your-app-entry-file.js>

7. Test Your Application: Ensure your application is running correctly by accessing it through the public IP or DNS of your EC2 instance.


Database:

        const firebaseConfig = {
            apiKey: process.env.FIREBASE_API_KEY,
            authDomain: process.env.FIREBASE_AUTH_DOMAIN,
            databaseURL: process.env.FIREBASE_DATABASE_URL,
            projectId: process.env.FIREBASE_PROJECT_ID,
            storageBucket: process.env.FIREBASE_STORAGE_BUCKET,
            messagingSenderId: process.env.FIREBASE_MESSAGING_SENDER_ID,
            appId: process.env.FIREBASE_APP_ID
        };

#DNS 
1.Purchase a Domain Name: If you haven't already, purchase a domain name from a domain registrar like GoDaddy, Namecheap, or AWS Route 53.

2.Access DNS Management: Log in to your domain registrar's website and access the DNS management section for the domain you've purchased.

3.Create DNS Records: Add DNS records to point your domain to your EC2 instance's public IP address. You typically need to create an A record for the root domain (example.com) and possibly CNAME records for subdomains (www.example.com).

  For the root domain (example.com):

      Type: A
      Name: Leave blank or use @
      Value: Your EC2 instance's public IP address

  For subdomains (www.example.com):

      Type: CNAME
      Name: "www" or the subdomain name
      Value: Your domain name or alias (if applicable)

4. Set TTL (Time to Live): Set the TTL for your DNS records. This specifies how long DNS resolvers should cache the DNS information. A lower TTL allows for faster updates but may increase DNS query load.

5. Verify DNS Configuration: After configuring DNS records, verify the setup using DNS lookup tools like dig or online DNS lookup tools. Ensure that the domain resolves to the correct IP address.

6. SSL Certificate (Optional): If you want to enable HTTPS for your domain, obtain an SSL certificate. You can use a service like Let's Encrypt for free SSL certificates.

7. Configure Security Groups: Ensure that your EC2 instance's security group allows inbound traffic on port 80 (HTTP) and/or port 443 (HTTPS) for web traffic.

8. Test Your Domain: Once DNS changes propagate (which may take some time), test your domain by accessing it in a web browser. Ensure that your application loads correctly.

#AWS Route 53 (Optional):
If you're using AWS for domain registration or want to manage DNS directly through AWS Route 53:

1. Transfer Domain (Optional): If you've purchased your domain from a different registrar, you can transfer it to AWS Route 53 for easier management.

2. Create Hosted Zone: In AWS Route 53, create a hosted zone for your domain.

3. Add Record Sets: Add A and CNAME record sets as mentioned earlier, pointing to your EC2 instance's public IP address.

4. Set DNS Alias (Optional): Instead of using an IP address for A records, you can use an Alias record that points to your EC2 instance directly.

5. SSL Certificate with AWS Certificate Manager (Optional): You can also use AWS Certificate Manager to provision SSL/TLS certificates for your domain.

#EKS Implementation

1. Dockerfile:

        # Use a Node.js base image
        FROM node:14
        
        # Set the working directory in the container
        WORKDIR /usr/src/app
        
        # Copy package.json and package-lock.json (if available)
        COPY package*.json ./
        
        # Install dependencies
        RUN npm install
        
        # Copy the rest of the application code
        COPY . .
        
        # Expose the port your app runs on
        EXPOSE 3000
        
        # Command to run your app
        CMD ["node", "app.js"]

--------------------------------------------
Push Docker Image to Amazon ECR: 
  
    # Login to AWS ECR
    aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <aws-account-id>.dkr.ecr.<region>.amazonaws.com
    
    # Tag your Docker image
    docker tag your-image-name:tag <aws-account-id>.dkr.ecr.<region>.amazonaws.com/your-repository-name:tag
    
    # Push your Docker image to ECR
    docker push <aws-account-id>.dkr.ecr.<region>.amazonaws.com/your-repository-name:tag

3. Kubernetes Deployment YAML (deployment.yaml):

        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: your-app-deployment
        spec:
          replicas: 3
          selector:
            matchLabels:
              app: your-app
          template:
            metadata:
              labels:
                app: your-app
            spec:
              containers:
                - name: your-app
                  image: your-image-name:tag
                  ports:
                    - containerPort: 3000

4. Kubernetes Service YAML (service.yaml):
  
        apiVersion: v1
        kind: Service
        metadata:
          name: your-app-service
        spec:
          selector:
            app: your-app
          ports:
            - protocol: TCP
              port: 80
              targetPort: 3000
          type: LoadBalancer

   ------------------------------------------

         # deployment.yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: your-app-deployment
        spec:
          replicas: 3
          selector:
            matchLabels:
              app: your-app
          template:
            metadata:
              labels:
                app: your-app
            spec:
              containers:
                - name: your-app
                  image: <aws-account-id>.dkr.ecr.<region>.amazonaws.com/your-repository-name:tag
                  ports:
                    - containerPort: 3000
        ---
        # service.yaml
        apiVersion: v1
        kind: Service
        metadata:
          name: your-app-service
        spec:
          selector:
            app: your-app
          ports:
            - protocol: TCP
              port: 80
              targetPort: 3000
          type: LoadBalancer


6. Apply Kubernetes Resources:

        kubectl apply -f deployment.yaml
        kubectl apply -f service.yaml

5. (Optional) Set Up Ingress:
    
        apiVersion: networking.k8s.io/v1
        kind: Ingress
        metadata:
          name: your-app-ingress
        spec:
          rules:
            - host: your-domain.com
              http:
                paths:
                  - pathType: Prefix
                    path: "/"
                    backend:
                      service:
                        name: your-app-service
                        port:
                          number: 80
          
6. Apply the Ingress YAML file: 

        kubectl apply -f ingress.yaml
