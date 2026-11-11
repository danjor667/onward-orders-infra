# Decisions

Each decision, with its reason in business terms and what was rejected.

| # | Decision | Reason | Rejected alternative |
|---|---|---|---|
| 1 | Releases shift 10% of traffic for 5 minutes, then the rest (`ECSCanary10Percent5Minutes`) | A bad release reaches one customer in ten for a few minutes instead of everyone. Protects trust on peak days | All at once: faster, but every customer sees a bad release. Linear over 10+ minutes: safer, but slows weekly releases |
| 2 | The old version stays running 10 minutes after a release | Going back is a traffic switch that takes seconds, not a rebuild | Keeping it for hours: doubles compute cost for little extra safety |
| 3 | Automatic rollback on failed health checks or 5xx errors | Nobody has to be watching for a release to be reversed | Manual rollback only: depends on someone noticing |
| 4 | Two tasks minimum, one per Availability Zone; six maximum | Two is the smallest number that survives the loss of a zone. Six caps the bill on busy days | One task: cheaper, but any failure is an outage. Three zones: more resilient, but 50% more endpoint cost |
| 5 | Scale on average CPU at 60% | Leaves headroom to absorb a spike while new tasks start. CPU follows the service through blue/green swaps | Requests per target: tied to one target group, which changes on every release |
| 6 | VPC endpoints for ECR, CloudWatch Logs and S3; no NAT gateway | The application has no route to the internet at all, which is what Security asked for. Also cheaper than two NAT gateways (see `cost.md`) | NAT gateway: gives the application a path to the internet |
| 7 | GitHub Actions signs in with OIDC, limited to the `main` branch of one repository | No access key exists to leak or rotate. A fork or another branch cannot release | Access keys stored as GitHub secrets: long-lived credentials |
| 8 | The pipeline is started by the ECR "image pushed" event | The image is the release. Nothing deploys unless a tested image exists | Polling the repository: slower and costs API calls. Webhook from GitHub: needs a stored secret |
| 9 | The task definition for releases lives in the app repository (`deploy/taskdef.json`) | A change to memory or environment variables is released with blue/green and can be rolled back like code | Keeping it only in CloudFormation: ECS rejects such updates on a blue/green service |
| 10 | Three stacks (network, registry, service) instead of one | Each file answers one question. A network change cannot accidentally touch the pipeline | One large template: one file to read, but every change risks everything. Nested stacks: GitSync cannot package them |
| 11 | Image tags are the Git commit and cannot be overwritten; only `latest` moves | Any running version can be traced to the exact commit, which helps audits | Mutable tags: the same tag could mean two different images |
| 12 | HTTP only on the load balancer | The lab provides no domain name, and a certificate needs one | See limits below |
| 13 | Orders in a DynamoDB table, billed per request | Every task sees the same orders, and they survive releases. It costs nothing when idle and needs no capacity planning | Orders in each container's memory: free, but customers lose their order when another task answers. RDS: a server to size and pay for all month |
| 14 | The application reaches DynamoDB through a gateway endpoint | Keeps the "no path to the internet" rule, and gateway endpoints are free | An interface endpoint: works, but costs per hour per zone |
| 15 | Staff passcode generated at first start and kept in the orders table | No secret in any repository, and every task shares the same one | Secrets Manager: the production answer, but needs another paid interface endpoint here. A passcode in the task definition: a credential in Git |
| 16 | Region: `eu-central-1` (to confirm) | It is the Region configured in the AWS CLI used to build this. **Confirm or change, and write the justification here**: the Client Profile asks for the Region to be chosen and justified | |

## Limits

- **No HTTPS.** Production needs a domain, an ACM certificate and a 443 listener. Customer data should not travel over plain HTTP.
- **The Operations pages use one shared passcode**, sent over plain HTTP. Production needs HTTPS and real staff sign-in.
- **The operations dashboard reads the whole orders table.** Fine at this size; a busy system would add an index by status or date.
- **Idle cost is not zero.** The load balancer and the three interface endpoints are billed while they exist. An idle environment costs nothing only if its stacks are deleted, which is why everything can be rebuilt from the repositories.
- **The GitSync role is broad.** It can create any `onward-*` IAM role and policy. A permissions boundary would narrow it further.
- **The connection approval is manual**, once per account.
