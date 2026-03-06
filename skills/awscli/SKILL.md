---
name: awscli
description: >
  Manage AWS services using the AWS CLI (aws command).
  Use when the user wants to interact with AWS services from the command line,
  including S3, EC2, IAM, Lambda, CloudFormation, ECS, SSM, Secrets Manager,
  DynamoDB, SQS, CloudWatch Logs, and more.
  Triggers on mentions of AWS, aws cli, S3, EC2, Lambda, or any AWS service management.
---

# AWS CLI Assistant

Help users manage AWS services using the `aws` CLI (v2).

## Prerequisites

- AWS CLI v2 installed (`brew install awscli` on macOS, or see [reference/commands.md](reference/commands.md))
- Credentials configured via `aws configure`, environment variables, SSO, or IAM role
- Verify setup: `aws sts get-caller-identity`

## Command Structure

```
aws <service> <command> [subcommand] [options]
```

## Configuration

### Files

- `~/.aws/credentials` — access keys (profile names without `profile` prefix)
- `~/.aws/config` — region, output, SSO settings (named profiles use `[profile name]`)

```ini
# ~/.aws/credentials
[default]
aws_access_key_id=AKIAIOSFODNN7EXAMPLE
aws_secret_access_key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

# ~/.aws/config
[default]
region=us-west-2
output=json

[profile staging]
region=us-east-1
output=table
```

### Key Environment Variables

| Variable | Purpose |
|----------|---------|
| `AWS_ACCESS_KEY_ID` | Access key |
| `AWS_SECRET_ACCESS_KEY` | Secret key |
| `AWS_SESSION_TOKEN` | Temporary session token |
| `AWS_DEFAULT_REGION` / `AWS_REGION` | Default region |
| `AWS_PROFILE` | Named profile to use |
| `AWS_PAGER` | Pager program (set `""` to disable) |

### Profiles

```bash
aws s3 ls --profile staging          # Use specific profile
export AWS_PROFILE=staging           # Set for session
aws configure list-profiles          # List all profiles
aws configure list                   # Show active config with sources
```

### Credential Precedence

1. CLI options → 2. Environment variables → 3. SSO → 4. `~/.aws/credentials` → 5. `~/.aws/config` → 6. Container/EC2 instance role

### SSO (Recommended for Human Users)

```ini
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
```

## Output & Filtering

### Output Formats

`--output json|yaml|text|table` (default: `json`)

### JMESPath Filtering (`--query`)

```bash
# Get specific fields as table
aws ec2 describe-instances \
    --query 'Reservations[*].Instances[*].{ID:InstanceId,Type:InstanceType,State:State.Name}' \
    --output table

# Filter by condition
aws ec2 describe-volumes --query 'Volumes[?Size > `50`].{ID:VolumeId,Size:Size}'

# Get single value as text
aws ec2 describe-instances --query 'Reservations[0].Instances[0].InstanceId' --output text

# Sort and limit
aws ec2 describe-images --owners self \
    --query 'reverse(sort_by(Images,&CreationDate))[:5].{ID:ImageId,Date:CreationDate}'
```

### Server-Side Filtering

Use `--filters` first to reduce data, then `--query` for formatting:

```bash
aws ec2 describe-instances \
    --filters "Name=instance-state-name,Values=running" "Name=tag:Env,Values=prod" \
    --query 'Reservations[*].Instances[*].[InstanceId,InstanceType]' --output text
```

### Pagination

```bash
--no-paginate              # First page only
--page-size 100            # Items per API call (reduces timeouts)
--max-items 50             # Total items limit
--starting-token <token>   # Resume from NextToken
--no-cli-pager             # Disable pager for this command
```

## Parameter Input

```bash
# From file (text)
aws ec2 run-instances --cli-input-json file://params.json

# From file (binary)
aws lambda update-function-code --zip-file fileb://function.zip

# Generate skeleton
aws ec2 run-instances --generate-cli-skeleton input > params.json

# Shorthand syntax
aws ec2 create-tags --resources i-123 --tags Key=Name,Value=MyServer Key=Env,Value=prod
```

## Common Patterns

### S3

```bash
aws s3 ls                                        # List buckets
aws s3 ls s3://bucket --recursive --human-readable --summarize
aws s3 cp file.txt s3://bucket/                   # Upload
aws s3 cp s3://bucket/file.txt ./                 # Download
aws s3 sync ./dir s3://bucket/dir --delete        # Sync with delete
aws s3 presign s3://bucket/file --expires-in 300  # Presigned URL
aws s3 rm s3://bucket/ --recursive                # Delete all objects
```

### EC2

```bash
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running"
aws ec2 run-instances --image-id ami-xxx --instance-type t3.micro --key-name mykey
aws ec2 stop-instances --instance-ids i-xxx
aws ec2 start-instances --instance-ids i-xxx
aws ec2 terminate-instances --instance-ids i-xxx
```

### IAM & STS

```bash
aws sts get-caller-identity                       # Who am I?
aws iam list-users
aws iam create-role --role-name MyRole --assume-role-policy-document file://trust.json
aws iam attach-role-policy --role-name MyRole --policy-arn arn:aws:iam::aws:policy/...
aws sts assume-role --role-arn arn:aws:iam::123456789012:role/MyRole --role-session-name sess1
```

### Lambda

```bash
aws lambda invoke --function-name my-func --payload '{"key":"val"}' out.json
aws lambda update-function-code --function-name my-func --zip-file fileb://code.zip
aws lambda list-functions
```

### CloudWatch Logs

```bash
aws logs tail /aws/lambda/my-func --follow --since 5m
aws logs filter-log-events --log-group-name /aws/lambda/my-func --filter-pattern "ERROR"
```

### SSM Parameter Store & Secrets Manager

```bash
aws ssm get-parameter --name /app/config/key --with-decryption
aws ssm put-parameter --name /app/config/key --type SecureString --value "secret"
aws ssm get-parameters-by-path --path /app/ --recursive --with-decryption
aws secretsmanager get-secret-value --secret-id MySecret
aws secretsmanager create-secret --name MySecret --secret-string file://creds.json
```

### Waiters

Block until a resource reaches a desired state:

```bash
aws ec2 wait instance-running --instance-ids i-xxx
aws cloudformation wait stack-create-complete --stack-name my-stack
```

## Global Options

| Option | Description |
|--------|-------------|
| `--region` | AWS region |
| `--profile` | Named profile |
| `--output` | Output format |
| `--query` | JMESPath filter |
| `--no-cli-pager` | Disable pager |
| `--no-paginate` | First page only |
| `--debug` | Debug logging to stderr |
| `--dry-run` | Validate without executing (EC2, etc.) |
| `--no-sign-request` | Skip signing (public resources) |
| `--endpoint-url` | Custom endpoint (LocalStack, etc.) |

## Return Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 252 | Invalid syntax or parameter |
| 253 | Invalid credentials/config |
| 254 | Service returned an error |
| 255 | General failure |

## Best Practices

1. **Use SSO or IAM roles** — avoid long-term access keys
2. **Use `--query` + `--output text`** for scriptable output
3. **Combine server-side `--filters` with client-side `--query`** for efficiency
4. **Use `file://` for JSON params** — avoids shell quoting issues and keeps secrets out of process lists
5. **Use `--generate-cli-skeleton`** to discover parameter structure
6. **Use waiters** instead of polling in scripts
7. **Set `AWS_PAGER=""`** in scripts to prevent pager blocking

For complete service command reference, see [reference/commands.md](reference/commands.md).
