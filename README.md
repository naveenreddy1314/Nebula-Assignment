1.Can you describe a challenging incident in production you managed in AWS/Kubernetes, what tools you used for troubleshooting, and how you reduced MTTR?                                      

Answer:   
We had a production issue in our AWS EKS cluster where one of our core services started failing during peak traffic. I noticed pods were going into CrashLoopBackOff due to memory issues. I used CloudWatch, kubectl logs and Grafana dashboards to confirm a memory leak introduced in the latest deployment.
To fix it quickly, I rolled back the deployment using Helm, increased memory limits temporarily, and restored service within 25 minutes. Afterward, we added better HPA settings and pod disruption budgets to prevent similar issues. This reduced our MTTR for similar incidents by more than half.


2.Explain your experience in building/optimizing CI/CD pipelines (tools used, rollback strategy, and security/observability integrations).

Answer:    
I’ve built and optimized CI/CD pipelines using Jenkins, and Azure DevOps . For application builds, I use Docker to containerize services and push images to Container Registry. I integrate SonarQube cloud for static code analysis and enforce quality gates before deployment. Deployments are managed via Helm into Kubernetes clusters and manage infrastructure with Terraform. . I use Prometheus and Grafana to monitor deployment health and performance metrics, and I’ve tuned HPA settings based on custom metrics like latency and queue depth. For rollback, I rely on Helm’s versioned releases so I can revert instantly if a deployment fails and restore terraform state from S3. Secrets are managed using AWS Secrets Manager or Azure Key Vault. 


3.Share an example where you used Infrastructure as Code (Terraform/Ansible) to automate cloud infrastructure at scale. What problems did it solve?

Answer:    
I used Terraform to automate the provisioning of a multi-region AWS and Azure infrastructure that needed isolated environments for regions all modularized using reusable Terraform modules. I also used Ansible to configure EC2 instances post-launch, installing monitoring agents, setting up SSH hardening, and applying baseline security policies. With Terraform and Ansible, we reduced setup time to under 10 minutes per environment, eliminated drift across regions, and ensured consistent tagging and cost tracking. We also integrated the Terraform pipeline into CI/CD using Jenkins, with remote state stored in S3 and locking via DynamoDB.


4.Write a Python script using Boto3 to list all EC2 instances in a region and highlight those with unencrypted EBS volumes.

    import boto3

    def lambda_handler(event, context):
    region = event.get("region", "us-east-1")
    ec2 = boto3.client('ec2', region_name=region)

    print(f"\n Scanning EC2 instances in region: {region}\n")

    response = ec2.describe_instances()
    reservations = response['Reservations']

    results = []

    for reservation in reservations:
        for instance in reservation['Instances']:
            instance_id = instance['InstanceId']
            instance_info = {"InstanceId": instance_id, "UnencryptedVolumes": []}

            for mapping in instance.get('BlockDeviceMappings', []):
                ebs = mapping.get('Ebs')
                if ebs:
                    volume_id = ebs['VolumeId']
                    volume = ec2.describe_volumes(VolumeIds=[volume_id])['Volumes'][0]
                    encrypted = volume['Encrypted']

                    if not encrypted:
                        instance_info["UnencryptedVolumes"].append(volume_id)

            results.append(instance_info)

    return results




5.Build a simple Python Flask API that returns health status (200 OK with JSON payload { "status": "healthy" }).


    from flask import Flask, jsonify
    app = Flask(__name__)
    @app.route("/health", methods=["GET"])
    def health():
        return jsonify({"status": "healthy"}), 200
    if __name__ == "__main__":
        app.run(host="0.0.0.0", port=5000)


6.Write a Go program that connects to AWS S3 and lists all buckets with their creation date.



    package main
    import (
    	"context"
    	"fmt"
    	"github.com/aws/aws-lambda-go/lambda"
    	"github.com/aws/aws-sdk-go-v2/aws"
    	"github.com/aws/aws-sdk-go-v2/config"
    	"github.com/aws/aws-sdk-go-v2/service/s3"
    )
    
    type Response struct {
    	Buckets []BucketInfo `json:"buckets"`
    }
    
    type BucketInfo struct {
    	Name         string `json:"name"`
    	CreationDate string `json:"creation_date"`
    }
    
    func Handler(ctx context.Context) (Response, error) {
    	cfg, err := config.LoadDefaultConfig(ctx)
    	if err != nil {
    		return Response{}, fmt.Errorf("failed to load config: %w", err)
    	}
    
    	s3Client := s3.NewFromConfig(cfg)
    	output, err := s3Client.ListBuckets(ctx, &s3.ListBucketsInput{})
    	if err != nil {
    		return Response{}, fmt.Errorf("failed to list buckets: %w", err)
    	}
    
    	var buckets []BucketInfo
    	for _, b := range output.Buckets {
    		buckets = append(buckets, BucketInfo{
    			Name:         aws.ToString(b.Name),
    			CreationDate: aws.ToTime(b.CreationDate).String(),
    		})
    	}
    
    	return Response{Buckets: buckets}, nil
    }
    
    func main() {
    	lambda.Start(Handler)
    }
