# Cost

Prices below are approximate on-demand list prices and differ by Region.
Monthly figures assume 730 hours.

**Official estimate (public link, no AWS account needed):**
[Onward Logistics - Lab 2 Release Pipeline (eu-central-1)](https://calculator.aws/#/estimate?id=ae373aefeaf478df91977c610014d4254553f5b1)
on the AWS Pricing Calculator: **USD 96.34 per month**, saved on 11 November 2026.
The link expires after one year.

| Line in the public estimate | USD / month |
|---|---|
| AWS Fargate: 2 tasks all month, 0.25 vCPU and 0.5 GB each | 20.72 |
| AWS Fargate: extra tasks for busy days and blue/green releases (140 task-hours) | 1.99 |
| Elastic Load Balancing: 1 Application Load Balancer, low traffic | 19.79 |
| Amazon VPC: 3 interface endpoints (ecr.api, ecr.dkr, logs) in 2 zones, 10 GB processed | 52.66 |
| Amazon DynamoDB: on-demand, 120,000 writes, 300,000 reads, 1 GB | 0.42 |
| Amazon ECR: about 2 GB of release images | 0.20 |
| AWS CodePipeline (V2): about 20 releases, 380 action-minutes (first 100 free) | 0.56 |
| **Total** | **96.34** |

**What private connectivity costs:** USD 52.66 a month, 55% of the bill. It is fixed: the
interface endpoints are billed every hour in each zone whether or not anything is released.
The S3 and DynamoDB gateway endpoints are free. Not included above: CloudWatch Logs and the
rollback alarm (cents at this volume), and CodeDeploy, EventBridge and ECS, which have no
charge for this setup.

## What runs all the time

| Item | Quantity | Approx. unit price | Approx. per month |
|---|---|---|---|
| Interface VPC endpoints (ECR API, ECR Docker, Logs) | 3 endpoints × 2 zones | $0.01 per hour each, plus about $0.01 per GB | $45 to $55 |
| S3 and DynamoDB gateway endpoints | 2 | free | $0 |
| DynamoDB orders table | on-demand | about $1.25 per million writes, $0.25 per million reads | cents at this volume; $0 when idle |
| Application Load Balancer | 1 | about $0.025 per hour, plus usage | $18 to $25 |
| Fargate tasks (0.25 vCPU, 0.5 GB) | 2 minimum | about $0.012 to $0.015 per hour each | $18 to $22 |
| CodePipeline, CodeDeploy, ECR, S3, logs | low volume | | a few dollars |

## What private connectivity costs

The three interface endpoints are the price of keeping the application off
the internet: roughly **$45 to $55 a month**, fixed, whether or not anyone
uses the system.

The alternative, a NAT gateway in each zone, costs roughly $65 to $75 a month
for the two gateways plus about $0.045 to $0.05 per GB, and gives the
application a route to the internet. Endpoints are cheaper here *and* meet
the security requirement. This reverses if the application ever needs many
more AWS services: each new interface endpoint adds about $15 to $18 a month.

## What changes with load

- Each extra task on a busy day costs about one and a half cents an hour and is removed when load drops. The maximum of six tasks caps this.
- During a release both versions run for about 15 minutes (5 minutes of traffic shift plus the 10 minute rollback window).

## Idle environments

An environment that exists costs about $85 to $100 a month even with no
traffic. To pay nothing for an idle `dev` or `test` environment, delete its
stacks and rebuild from the repositories when needed (see the README).

## Building the estimate

In the [AWS Pricing Calculator](https://calculator.aws/#/), choose *Create
estimate*, set the Region, and add these services.

| Service to add | What to enter | Price list |
|---|---|---|
| AWS Fargate | Linux, x86, 0.25 vCPU and 0.5 GB per task. 2 tasks always on; add up to 4 more for the busy hours you expect | [Fargate pricing](https://aws.amazon.com/fargate/pricing/) |
| Elastic Load Balancing | 1 Application Load Balancer, with an estimate of requests and data per hour | [Elastic Load Balancing pricing](https://aws.amazon.com/elasticloadbalancing/pricing/) |
| AWS PrivateLink (VPC endpoints) | 3 interface endpoints × 2 Availability Zones, a few GB of data per month. The S3 gateway endpoint is free | [PrivateLink pricing](https://aws.amazon.com/privatelink/pricing/) |
| Amazon ECR | Storage for about 20 images | [ECR pricing](https://aws.amazon.com/ecr/pricing/) |
| AWS CodePipeline | 1 V2 pipeline, with the number of releases per month | [CodePipeline pricing](https://aws.amazon.com/codepipeline/pricing/) |
| Amazon CloudWatch | Log ingestion and 14 days of storage | [CloudWatch pricing](https://aws.amazon.com/cloudwatch/pricing/) |
| Amazon DynamoDB | On-demand capacity, with the number of orders per month (each order is a handful of writes and reads) | [DynamoDB pricing](https://aws.amazon.com/dynamodb/pricing/on-demand/) |

To show Finance the alternative that was rejected, add a second group with
2 NAT gateways instead of the endpoints ([VPC pricing](https://aws.amazon.com/vpc/pricing/)).
Then choose *Share* and paste the public link at the top of this page.

## Keeping spend visible

- **Budget alert.** The Client Profile asks for budget alerts to stay active.
  [AWS Budgets](https://aws.amazon.com/aws-cost-management/aws-budgets/)
  emails you when actual or forecast spend passes a limit; budgets that only
  send alerts are free. Set the monthly limit a little above the
  estimate.
- **Tags.** Every resource carries `Project`, `Owner` and `Environment`.
  Activate them as cost allocation tags in Billing, then filter
  [Cost Explorer](https://aws.amazon.com/aws-cost-management/aws-cost-explorer/)
  by `Project = onward` to split the bill by environment and compare it
  with the estimate.
