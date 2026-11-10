# Cost

Prices below are approximate on-demand list prices and differ by Region.
**Check them in the AWS Pricing Calculator for the chosen Region before
presenting them.** Monthly figures assume 730 hours.

## What runs all the time

| Item | Quantity | Approx. unit price | Approx. per month |
|---|---|---|---|
| Interface VPC endpoints (ECR API, ECR Docker, Logs) | 3 endpoints × 2 zones | $0.01 per hour each, plus about $0.01 per GB | $45 to $55 |
| S3 gateway endpoint | 1 | free | $0 |
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

## Keeping spend visible

Every resource carries `Project`, `Owner` and `Environment` tags. Activate
them as cost allocation tags in Billing to split the bill by environment.
The Client Profile also asks for budget alerts to stay active; those were
set up at account level in week 1 and are not repeated here.
