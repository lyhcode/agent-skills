# AWS CLI Command Reference

Detailed command reference for `aws` CLI v2. Source: https://docs.aws.amazon.com/cli/latest/

## Installation

### macOS
```bash
brew install awscli
# or
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
```

### Linux (x86_64)
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

### Linux (ARM)
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-aarch64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

### Windows
```cmd
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi
```

### Update
```bash
# Linux
sudo ./aws/install --update
# macOS/Windows: re-run the installer
```

## Configuration

### Interactive Setup
```bash
aws configure                        # Default profile
aws configure --profile staging      # Named profile
aws configure set region us-west-2 --profile staging
aws configure get region --profile staging
aws configure list                   # Show active config with sources
aws configure list-profiles          # List all profiles
aws configure export-credentials --profile dev --format env
```

### SSO Configuration

```ini
# ~/.aws/config (recommended modern approach)
[profile dev]
sso_session = my-sso
sso_account_id = 111122223333
sso_role_name = SampleRole
region = us-west-2

[sso-session my-sso]
sso_region = us-east-1
sso_start_url = https://my-sso-portal.awsapps.com/start
sso_registration_scopes = sso:account:access
```

```bash
aws sso login --profile dev
aws sso login --sso-session my-sso   # Login to SSO session directly
aws sso logout                       # Clear cached tokens
```

### Credential Precedence (highest to lowest)

1. Command line options (`--region`, `--profile`)
2. Environment variables (`AWS_ACCESS_KEY_ID`, etc.)
3. Assume role (via config)
4. Assume role with web identity
5. AWS IAM Identity Center (SSO)
6. Credentials file (`~/.aws/credentials`)
7. Custom process (`credential_process`)
8. Config file (`~/.aws/config`)
9. Container credentials (ECS task role)
10. EC2 instance profile (IMDS)

### All Environment Variables

| Variable | Purpose |
|----------|---------|
| `AWS_ACCESS_KEY_ID` | Access key |
| `AWS_SECRET_ACCESS_KEY` | Secret key |
| `AWS_SESSION_TOKEN` | Session token (temporary creds) |
| `AWS_DEFAULT_REGION` | Default region |
| `AWS_REGION` | SDK-compatible region (overrides DEFAULT_REGION) |
| `AWS_PROFILE` | Named profile |
| `AWS_DEFAULT_OUTPUT` | Output format |
| `AWS_CONFIG_FILE` | Custom config file path |
| `AWS_SHARED_CREDENTIALS_FILE` | Custom credentials file path |
| `AWS_ROLE_ARN` | IAM role ARN (web identity) |
| `AWS_WEB_IDENTITY_TOKEN_FILE` | Web identity token file path |
| `AWS_PAGER` | Pager program (empty to disable) |
| `AWS_CLI_AUTO_PROMPT` | Auto-prompt (`on` or `on-partial`) |
| `AWS_MAX_ATTEMPTS` | Max retry attempts |
| `AWS_RETRY_MODE` | `legacy`, `standard`, or `adaptive` |
| `AWS_EC2_METADATA_DISABLED` | Disable IMDS (`true`) |
| `AWS_ENDPOINT_URL` | Global custom endpoint |
| `AWS_ENDPOINT_URL_<SERVICE>` | Per-service custom endpoint |
| `AWS_CA_BUNDLE` | CA certificate bundle path |

## Output Formats

| Format | Description |
|--------|-------------|
| `json` (default) | Standard JSON, works with `jq` |
| `yaml` | YAML, good for CloudFormation |
| `yaml-stream` | Streaming YAML, starts before full download |
| `text` | Tab-delimited, for `grep`/`awk`/`sed` |
| `table` | ASCII table, human-readable |
| `off` | Suppress stdout, check `$?` for status |

## JMESPath Query Reference

### Syntax

| Pattern | Example |
|---------|---------|
| Select all | `'Items[*]'` |
| By index | `'Items[0]'` |
| Slice | `'Items[0:3]'`, `'Items[::-1]'` |
| Nested | `'Reservations[*].Instances[*].InstanceId'` |
| Flatten | `'Reservations[].Instances[].InstanceId'` |
| Filter | `'Volumes[?Size > `50`]'` |
| Comparators | `==`, `!=`, `<`, `<=`, `>`, `>=` |
| Pipe | `'Items[*].Name | [0]'` |
| Multiselect list | `'Items[].[Id, Name]'` |
| Multiselect hash | `'Items[].{ID: Id, Name: Name}'` |

### Functions

`sort_by()`, `reverse()`, `length()`, `not_null()`, `max()`, `min()`, `contains()`, `starts_with()`, `ends_with()`

### Examples

```bash
# Running instances as table
aws ec2 describe-instances \
    --filters "Name=instance-state-name,Values=running" \
    --query 'Reservations[*].Instances[*].{ID:InstanceId,Type:InstanceType,AZ:Placement.AvailabilityZone,Name:Tags[?Key==`Name`]|[0].Value}' \
    --output table

# Count volumes with high IOPS
aws ec2 describe-volumes --query 'length(Volumes[?Iops > `1000`])'

# 5 most recent AMIs
aws ec2 describe-images --owners self \
    --query 'reverse(sort_by(Images,&CreationDate))[:5].{ID:ImageId,Date:CreationDate}'
```

## Pagination

| Parameter | Purpose |
|-----------|---------|
| `--no-paginate` | First page only (single API call) |
| `--page-size <n>` | Items per API call (reduces timeouts) |
| `--max-items <n>` | Total items limit; returns `NextToken` |
| `--starting-token <token>` | Resume from previous `NextToken` |

## Parameter Input

### File Loading
- `file://path/to/file.json` — text content
- `fileb://path/to/file.bin` — raw binary content
- Relative to CWD; use `file:///absolute/path` for absolute

### Skeleton Workflow
```bash
aws ec2 run-instances --generate-cli-skeleton input > params.json
# Edit params.json
aws ec2 run-instances --cli-input-json file://params.json
```

### Shorthand Syntax

| Pattern | Shorthand |
|---------|-----------|
| Key-value | `--opt key1=val1,key2=val2` |
| List | `--opt val1 val2 val3` |
| List of structs | `Key=A,Value=1 Key=B,Value=2` |

## Return Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | S3 transfer failure |
| 2 | Parsing error or S3 partial skip |
| 130 | SIGINT (Ctrl+C) |
| 252 | Invalid syntax/parameter |
| 253 | Invalid credentials/config |
| 254 | Service returned error |
| 255 | General failure |

## Global Options

| Option | Description |
|--------|-------------|
| `--region <region>` | AWS region |
| `--profile <name>` | Named profile |
| `--output <format>` | json/yaml/text/table/off |
| `--query <jmespath>` | JMESPath filter |
| `--no-cli-pager` | Disable pager |
| `--no-paginate` | First page only |
| `--no-sign-request` | Skip signing |
| `--no-verify-ssl` | Skip SSL verification |
| `--endpoint-url <url>` | Custom endpoint |
| `--debug` | Debug logging to stderr |
| `--ca-bundle <path>` | Custom CA bundle |
| `--cli-connect-timeout <s>` | Socket connect timeout |
| `--cli-read-timeout <s>` | Socket read timeout |
| `--color on\|off\|auto` | Colored output |

---

## S3

### High-Level Commands (`aws s3`)

| Command | Description |
|---------|-------------|
| `aws s3 ls` | List buckets or objects |
| `aws s3 cp` | Copy files to/from S3 |
| `aws s3 mv` | Move files |
| `aws s3 rm` | Delete objects |
| `aws s3 sync` | Sync directories |
| `aws s3 mb` | Make bucket |
| `aws s3 rb` | Remove bucket |
| `aws s3 presign` | Generate presigned URL |

### S3 URI Format

```
s3://bucket-name/key-name
s3://arn:aws:s3:us-west-2:123456789012:accesspoint/myaccesspoint/key
```

### s3 vs s3api

- `aws s3` — High-level convenience commands, automatic multipart uploads
- `aws s3api` — Low-level 1:1 API mapping, fine-grained control

### aws s3 cp

```bash
aws s3 cp file.txt s3://bucket/                          # Upload
aws s3 cp s3://bucket/file.txt ./                        # Download
aws s3 cp s3://bucket1/f.txt s3://bucket2/f.txt          # S3-to-S3
aws s3 cp myDir s3://bucket/ --recursive                 # Recursive upload
aws s3 cp myDir s3://bucket/ --recursive --exclude "*.jpg"
aws s3 cp s3://bucket/logs/ . --recursive --exclude "*" --include "*.log"
aws s3 cp file.txt s3://bucket/ --storage-class GLACIER
aws s3 cp - s3://bucket/stream.txt                       # From stdin
aws s3 cp myDir s3://bucket/ --recursive --dryrun        # Dry run
```

**Key flags:**
- `--recursive` — All files under directory/prefix
- `--exclude` / `--include` — Glob patterns (later takes precedence)
- `--acl` — `private`, `public-read`, `public-read-write`, `authenticated-read`, `bucket-owner-full-control`
- `--storage-class` — `STANDARD`, `STANDARD_IA`, `ONEZONE_IA`, `INTELLIGENT_TIERING`, `GLACIER`, `DEEP_ARCHIVE`, `GLACIER_IR`
- `--sse` — `AES256` or `aws:kms`
- `--dryrun` — Preview without executing
- `--content-type` — Override MIME type

### aws s3 sync

```bash
aws s3 sync . s3://bucket                                # Local to S3
aws s3 sync s3://bucket .                                # S3 to local
aws s3 sync . s3://bucket --delete                       # Delete removed files
aws s3 sync . s3://bucket --exclude "*.jpg" --dryrun
aws s3 sync . s3://bucket --size-only                    # Compare by size only
```

**Key flags:**
- `--delete` — Remove destination files not in source
- `--exact-timestamps` — Match timestamps exactly
- `--size-only` — Use file size only (not timestamps)
- `--dryrun` — Preview

### aws s3 ls

```bash
aws s3 ls                                                # List buckets
aws s3 ls s3://bucket                                    # List objects
aws s3 ls s3://bucket --recursive                        # Recursive
aws s3 ls s3://bucket --recursive --human-readable --summarize
```

### Other S3 Commands

```bash
aws s3 mb s3://new-bucket                                # Make bucket
aws s3 rb s3://bucket                                    # Remove empty bucket
aws s3 rb s3://bucket --force                            # Remove bucket + all contents
aws s3 mv s3://bucket/old.txt s3://bucket/new.txt        # Move/rename
aws s3 rm s3://bucket/file.txt                           # Delete object
aws s3 rm s3://bucket --recursive                        # Delete all objects
aws s3 presign s3://bucket/file.txt --expires-in 300     # Presigned URL (seconds)
```

---

## EC2

### Describe Instances

```bash
aws ec2 describe-instances
aws ec2 describe-instances --instance-ids i-xxx
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running"
aws ec2 describe-instances --filters "Name=tag:Env,Values=prod"
aws ec2 describe-instances --filters Name=instance-type,Values=t2.micro,t3.micro

# Useful query patterns
aws ec2 describe-instances \
    --query 'Reservations[*].Instances[*].{ID:InstanceId,Type:InstanceType,State:State.Name,Name:Tags[?Key==`Name`]|[0].Value}' \
    --output table

aws ec2 describe-instances \
    --filters "Name=instance-state-name,Values=running" \
    --query "Reservations[*].Instances[*].[InstanceId]" --output text
```

**Common filters:** `instance-state-name`, `instance-type`, `vpc-id`, `subnet-id`, `availability-zone`, `ip-address`, `private-ip-address`, `tag:<key>`, `key-name`, `architecture`

### Run Instances

```bash
aws ec2 run-instances \
    --image-id ami-xxx \
    --instance-type t3.micro \
    --key-name my-key \
    --security-group-ids sg-xxx \
    --subnet-id subnet-xxx \
    --count 1 \
    --user-data file://startup.sh \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=MyServer}]' \
    --iam-instance-profile Name=MyProfile \
    --block-device-mappings 'DeviceName=/dev/sda1,Ebs={VolumeSize=50,VolumeType=gp3}'
```

### Instance Lifecycle

```bash
aws ec2 start-instances --instance-ids i-xxx
aws ec2 stop-instances --instance-ids i-xxx
aws ec2 reboot-instances --instance-ids i-xxx
aws ec2 terminate-instances --instance-ids i-xxx
aws ec2 describe-instance-status --instance-ids i-xxx
```

### Security Groups & VPCs

```bash
aws ec2 describe-security-groups
aws ec2 describe-security-groups --filters Name=vpc-id,Values=vpc-xxx
aws ec2 create-security-group --group-name my-sg --description "Desc" --vpc-id vpc-xxx
aws ec2 authorize-security-group-ingress --group-id sg-xxx --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 describe-vpcs
aws ec2 describe-subnets --filters Name=vpc-id,Values=vpc-xxx
aws ec2 create-tags --resources i-xxx --tags Key=Name,Value=MyServer
```

### Waiters

```bash
aws ec2 wait instance-running --instance-ids i-xxx
aws ec2 wait instance-stopped --instance-ids i-xxx
aws ec2 wait instance-terminated --instance-ids i-xxx
aws ec2 wait volume-available --volume-ids vol-xxx
aws ec2 wait snapshot-completed --snapshot-ids snap-xxx
```

---

## IAM

### Users

```bash
aws iam create-user --user-name johndoe
aws iam list-users
aws iam get-user --user-name johndoe
aws iam delete-user --user-name johndoe
aws iam create-access-key --user-name johndoe
aws iam list-access-keys --user-name johndoe
aws iam update-access-key --user-name johndoe --access-key-id AKIAXXX --status Inactive
```

### Roles

```bash
aws iam create-role --role-name MyRole --assume-role-policy-document file://trust.json
aws iam list-roles
aws iam get-role --role-name MyRole
aws iam attach-role-policy --role-name MyRole --policy-arn arn:aws:iam::aws:policy/...
aws iam detach-role-policy --role-name MyRole --policy-arn arn:aws:iam::aws:policy/...
aws iam put-role-policy --role-name MyRole --policy-name Inline --policy-document file://policy.json
aws iam update-assume-role-policy --role-name MyRole --policy-document file://trust.json
```

### Policies

```bash
aws iam create-policy --policy-name MyPolicy --policy-document file://policy.json
aws iam list-policies --scope Local
aws iam get-policy --policy-arn arn:aws:iam::123456789012:policy/MyPolicy
aws iam get-policy-version --policy-arn arn:aws:iam::123456789012:policy/MyPolicy --version-id v1
aws iam attach-user-policy --user-name johndoe --policy-arn arn:aws:iam::aws:policy/...
aws iam attach-group-policy --group-name devs --policy-arn arn:aws:iam::aws:policy/...
```

### Groups

```bash
aws iam create-group --group-name developers
aws iam add-user-to-group --user-name johndoe --group-name developers
aws iam list-groups-for-user --user-name johndoe
aws iam remove-user-from-group --user-name johndoe --group-name developers
```

### STS

```bash
# Who am I?
aws sts get-caller-identity

# Assume role
aws sts assume-role \
    --role-arn arn:aws:iam::123456789012:role/MyRole \
    --role-session-name my-session

# With MFA
aws sts assume-role \
    --role-arn arn:aws:iam::123456789012:role/MyRole \
    --role-session-name my-session \
    --serial-number arn:aws:iam::123456789012:mfa/user \
    --token-code 123456 \
    --duration-seconds 3600

# Use assumed credentials
CREDS=$(aws sts assume-role --role-arn arn:aws:iam::123456789012:role/MyRole --role-session-name s)
export AWS_ACCESS_KEY_ID=$(echo $CREDS | jq -r '.Credentials.AccessKeyId')
export AWS_SECRET_ACCESS_KEY=$(echo $CREDS | jq -r '.Credentials.SecretAccessKey')
export AWS_SESSION_TOKEN=$(echo $CREDS | jq -r '.Credentials.SessionToken')
```

---

## Lambda

### Invoke

```bash
# Synchronous
aws lambda invoke --function-name my-func --payload '{"key":"val"}' response.json

# Async (fire-and-forget)
aws lambda invoke --function-name my-func --invocation-type Event --payload '{}' response.json

# With logs
aws lambda invoke --function-name my-func --log-type Tail --payload '{}' response.json

# Specific version/alias
aws lambda invoke --function-name my-func --qualifier prod --payload '{}' response.json
```

- `--invocation-type`: `RequestResponse` (sync), `Event` (async), `DryRun` (validate)
- `--cli-binary-format raw-in-base64-out` — pass raw JSON payload
- Payload limit: 6MB sync, 1MB async

### Create & Update

```bash
# Create from zip
aws lambda create-function \
    --function-name my-func \
    --runtime python3.12 \
    --zip-file fileb://code.zip \
    --handler lambda_function.lambda_handler \
    --role arn:aws:iam::123456789012:role/lambda-role \
    --environment Variables={STAGE=prod} \
    --timeout 30 --memory-size 256

# Create from container
aws lambda create-function \
    --function-name my-func \
    --role arn:aws:iam::123456789012:role/lambda-role \
    --code ImageUri=123456789012.dkr.ecr.us-east-1.amazonaws.com/repo:latest \
    --package-type Image

# Update code
aws lambda update-function-code --function-name my-func --zip-file fileb://code.zip
aws lambda update-function-code --function-name my-func --s3-bucket b --s3-key code.zip --publish

# List
aws lambda list-functions
```

**Runtimes:** `nodejs22.x`, `python3.12`, `java21`, `dotnet8`, `provided.al2023`, etc.

---

## CloudFormation

```bash
# Deploy (create or update)
aws cloudformation deploy \
    --template-file template.yaml \
    --stack-name my-stack \
    --parameter-overrides Env=prod DbSize=large \
    --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM \
    --tags Environment=prod Team=backend \
    --no-fail-on-empty-changeset

# Create stack
aws cloudformation create-stack \
    --stack-name my-stack \
    --template-body file://template.json \
    --parameters ParameterKey=KeyPair,ParameterValue=mykey \
    --capabilities CAPABILITY_IAM

# Delete stack
aws cloudformation delete-stack --stack-name my-stack

# Describe stacks
aws cloudformation describe-stacks --stack-name my-stack

# List stacks by status
aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE

# Wait for completion
aws cloudformation wait stack-create-complete --stack-name my-stack
aws cloudformation wait stack-delete-complete --stack-name my-stack
```

---

## CloudWatch Logs

```bash
# List log groups
aws logs describe-log-groups
aws logs describe-log-groups --log-group-name-prefix /aws/lambda/

# Tail logs (real-time)
aws logs tail /aws/lambda/my-func --follow --since 5m
aws logs tail /aws/lambda/my-func --follow --filter-pattern "ERROR" --format short

# Filter events
aws logs filter-log-events \
    --log-group-name /aws/lambda/my-func \
    --filter-pattern "ERROR" \
    --start-time 1609459200000 --end-time 1609545600000

# JSON pattern filter
aws logs filter-log-events \
    --log-group-name my-group \
    --filter-pattern '{ $.statusCode = 500 }'
```

- `--since`: Relative time (`5m`, `2h`, `1d`) or ISO timestamp
- `--format`: `detailed` (default), `short`, `json`

---

## ECS

```bash
# List clusters and services
aws ecs list-clusters
aws ecs list-services --cluster my-cluster
aws ecs describe-services --cluster my-cluster --services my-service

# Deploy new task definition
aws ecs update-service --cluster my-cluster --service my-service --task-definition my-task:2

# Scale service
aws ecs update-service --cluster my-cluster --service my-service --desired-count 5

# Force new deployment (same image tag)
aws ecs update-service --cluster my-cluster --service my-service --force-new-deployment

# Run standalone task (Fargate)
aws ecs run-task \
    --cluster my-cluster \
    --task-definition my-task:1 \
    --launch-type FARGATE \
    --count 1 \
    --network-configuration 'awsvpcConfiguration={subnets=[subnet-xxx],securityGroups=[sg-xxx],assignPublicIp=ENABLED}'

# With container overrides
aws ecs run-task \
    --cluster my-cluster \
    --task-definition my-task:1 \
    --launch-type FARGATE \
    --network-configuration 'awsvpcConfiguration={subnets=[subnet-xxx],assignPublicIp=ENABLED}' \
    --overrides '{"containerOverrides":[{"name":"app","command":["python","migrate.py"]}]}'
```

---

## SSM Parameter Store

```bash
# Get parameter
aws ssm get-parameter --name /app/config/db-host
aws ssm get-parameter --name /app/secrets/api-key --with-decryption

# Get specific version/label
aws ssm get-parameter --name "MyParam:2"
aws ssm get-parameter --name "MyParam:prod-label"

# Put parameter
aws ssm put-parameter --name /app/config/db-host --type String --value "db.example.com"
aws ssm put-parameter --name /app/secrets/key --type SecureString --value "secret"
aws ssm put-parameter --name /app/config/db-host --type String --value "new-db" --overwrite

# Get all under path
aws ssm get-parameters-by-path --path /app/ --recursive --with-decryption
```

- Types: `String`, `StringList`, `SecureString`
- Tiers: `Standard` (4KB), `Advanced` (8KB), `Intelligent-Tiering`

---

## Secrets Manager

```bash
# Get secret
aws secretsmanager get-secret-value --secret-id MySecret
aws secretsmanager get-secret-value --secret-id MySecret --version-stage AWSPREVIOUS

# Create secret (use file:// to avoid shell history exposure)
aws secretsmanager create-secret --name MySecret --description "DB creds" --secret-string file://creds.json

# Update secret
aws secretsmanager update-secret --secret-id MySecret --secret-string file://new-creds.json
```

---

## DynamoDB

### Get Item

```bash
aws dynamodb get-item \
    --table-name MyTable \
    --key '{"PK": {"S": "user#123"}, "SK": {"S": "profile"}}' \
    --consistent-read

# Projection (specific attributes)
aws dynamodb get-item --table-name MyTable \
    --key '{"Id": {"N": "102"}}' \
    --projection-expression "Title, Price"
```

### Put Item

```bash
aws dynamodb put-item \
    --table-name MyTable \
    --item '{"PK": {"S": "user#123"}, "SK": {"S": "profile"}, "Name": {"S": "Alice"}}'

# Conditional put
aws dynamodb put-item --table-name MyTable \
    --item file://item.json \
    --condition-expression "attribute_not_exists(PK)"
```

### Query

```bash
# By partition key
aws dynamodb query \
    --table-name MyTable \
    --key-condition-expression "PK = :pk" \
    --expression-attribute-values '{":pk": {"S": "user#123"}}'

# With sort key range and filter
aws dynamodb query \
    --table-name MyTable \
    --key-condition-expression "PK = :pk AND SK BETWEEN :s1 AND :s2" \
    --filter-expression "Status <> :status" \
    --expression-attribute-values '{":pk":{"S":"user#123"},":s1":{"S":"A"},":s2":{"S":"M"},":status":{"S":"deleted"}}' \
    --no-scan-index-forward
```

- Sort key operators: `=`, `<`, `<=`, `>`, `>=`, `BETWEEN`, `begins_with()`
- `--no-scan-index-forward` — descending order
- Filter applies AFTER query (does not reduce consumed capacity)

### Scan

```bash
aws dynamodb scan --table-name MyTable \
    --filter-expression "Age > :age" \
    --projection-expression "Name, Email" \
    --expression-attribute-values '{":age": {"N": "30"}}'
```

- Max 1MB per call; use `LastEvaluatedKey` for pagination
- `--segment` / `--total-segments` for parallel scans

### DynamoDB Type Indicators

`S` (String), `N` (Number), `B` (Binary), `BOOL` (Boolean), `NULL` (Null), `L` (List), `M` (Map), `SS`/`NS`/`BS` (Sets)

---

## SQS

```bash
# Send message
aws sqs send-message \
    --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/MyQueue \
    --message-body "Task payload" \
    --delay-seconds 10

# FIFO queue
aws sqs send-message \
    --queue-url https://sqs.../MyQueue.fifo \
    --message-body "Order data" \
    --message-group-id "order-processing" \
    --message-deduplication-id "unique-id-123"

# Receive messages (long polling)
aws sqs receive-message \
    --queue-url https://sqs.../MyQueue \
    --max-number-of-messages 10 \
    --wait-time-seconds 20 \
    --attribute-names All

# Delete message (use ReceiptHandle from receive)
aws sqs delete-message \
    --queue-url https://sqs.../MyQueue \
    --receipt-handle "AQEBRXTo...q2doVA=="
```

- Message body max: 256KB
- `--delay-seconds`: 0–900
- `--max-number-of-messages`: 1–10
- `--wait-time-seconds`: Long polling (reduces empty responses)
