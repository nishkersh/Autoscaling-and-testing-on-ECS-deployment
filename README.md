
# Auto Scaling and Testing an Amazon ECS Deployment


### Brief Overview of the Lab Objective and it's Relevence at Enterprise Level 

In this Lab mainly Two things are performed :

1. Auto Scaling involves dynamically adding or removing instances of our application based on traffic and load patterns. This ensures that the application can handle varying levels of user activity without overloading or underutilizing resources. It's important at the enterprise level because it optimizes resource usage, maintains consistent performance, and reduces costs by scaling resources up during peak periods and down during off-peak periods.In this lab I did **Metric Target Tracking Type Of Scaling**

2. Testing the deployment involves running automated tests on the new deployment before making it available to users. This testing process helps identify and fix any issues or bugs before they impact the users, enhancing the overall quality of the service. At the enterprise level, thorough testing prevents downtime, ensures a positive user experience, and maintains the organization's reputation. And in this lab I **used Siege command for Load testing**

3. In lab for container deployment Amazon Elastic Container Service (ECS) is used .It's a container orchestration service that allows us to deploy and manage Docker containers at scale. There are several methods of deploying containers using ECS. And in this lab I have used **Fargate Launch Type** . As Fargate is a **serverless compute engine** for containers. With Fargate, we don't need to manage the underlying infrastructure; Amazon ECS handles it for us. We simply define our containerized application, its resource requirements, and ECS takes care of the rest.

In combination, auto scaling and rigorous testing create a robust and reliable environment for deploying applications, addressing both scalability and quality assurance concerns at an enterprise level.

### Learning Objectives

- Configure auto scaling for an ECS service using the AWS command-line interface (CLI)
- Perform a load test on a web application running in ECS
- Observe the auto scaling process using the Amazon ECS console

### Standard Enviornment Provided In The Lab
- This lab has deployed a sample microservices application into an ECS cluster for you. The application consists of three services. One frontend service that is publicly accessible through a load balancer, and two private backend services that the frontend service fetches data from.

Diagram of the Deployed Resources

![](https://assets.cloudacademy.com/labs/uploads/ecs-autoscaling-lab/steps/2_reviewing-the-ecs-deployment/assets/ecs-demo.drawio.png)

---
![](https://drive.google.com/file/d/1ZfXKYTMqM12UmRH54Wnp1bymYc_DScS2/view?usp=drive_link)
---
Therefore through this lab I will be explaining how to use Application Auto Scaling to scale the frontend service in response to high CPU utilization.

-------
### STEPS
#### Here I will be discussing the steps that I performed in the lab after logging into the AWS console and configuring the Open code Editor with my Access Key ID, Secret Access Key, and default region. 
- **Scaling ECS services using metric target tracking**

In this lab step, I inspected the application in the Amazon ECS console

-----

>And I used the AWS command-line interface (CLI) to apply a target tracking scaling policy to the frontend application.And for this Target tracking method I used Amazon CloudWatch metrics to determine when scaling should occur.

1. For  listing available metrics for the frontend service:
 `aws cloudwatch list-metrics \`
 >`   --namespace "AWS/ECS" \`
 `   --dimensions "Name=ServiceName,Value=frontend-service-fxgjjv"`
`

2. I created a service-linked role through CLI and and got a response,which is a JSON representation of the newly created service-linked role:
`aws iam create-service-linked-role --aws-service-name ecs.application-autoscaling.amazonaws.com`

3. Then registered a service as a scalable target for Application Auto Scaling
  `aws application-autoscaling register-scalable-target \`
 >`   --service-namespace ecs \`
 `   --scalable-dimension ecs:service:DesiredCount \`
  >`  --resource-id service/lab-cluster/frontend-service-fxgjjv \`
`  --min-capacity 1 \`
  >  `--max-capacity 4`
4. Configured a file for a target tracking scaling policy
```bash
cat << EOF > scaling-config.json
{
    "TargetValue":20.0,
    "CustomizedMetricSpecification":{
        "MetricName":"CPUUtilization",
        "Namespace":"AWS/ECS",
        "Dimensions":[
            {
                "Name":"ServiceName",
                "Value":"frontend-service-fxgjjv"
            },
            {
                "Name":"ClusterName",
                "Value":"lab-cluster"
            }
        ],
        "Statistic":"Maximum",
        "Unit":"Percent"
    }
}
EOF
```

5. Created a scaling policy using command
```bash
aws application-autoscaling put-scaling-policy \
    --service-namespace ecs \
    --scalable-dimension ecs:service:DesiredCount \
    --resource-id service/lab-cluster/frontend-service-fxgjjv \
    --policy-name cpu20-target-tracking-scaling-policy --policy-type TargetTrackingScaling \
    --target-tracking-scaling-policy-configuration file://scaling-config.json
```
6. Then I checked  the Service configuration of the fronted service group and there we can see a new policy is being created






