# Fargate Fluent Bit Logger

Amazon EKS on Fargate offers a built-in log router based on Fluent Bit.
This means that you don't explicitly run a Fluent Bit container as a sidecar, but Amazon runs it for you.
All that you have to do is configure the log router.
The configuration happens through a dedicated ConfigMap that must meet the following criteria:

Named `aws-logging`

Created in a dedicated namespace called `aws-observability`

Once you've created the ConfigMap, Amazon EKS on Fargate automatically detects it and configures the log router with it.
Fargate uses a version of AWS for Fluent Bit, an upstream compliant distribution of Fluent Bit managed by AWS.
For more information, see AWS for Fluent Bit on GitHub.

The log router allows you to use the breadth of services at AWS for log analytics and storage.
You can stream logs from Fargate directly to `Amazon CloudWatch`, `Amazon OpenSearch Service`.
You can also stream logs to destinations such as `Amazon S3`, `Amazon Kinesis Data Streams`, and partner tools through Amazon Kinesis Data Firehose.

# Fluent Bit CloudWatch Config
Please find the updated configuration from [AWS Docs](https://docs.aws.amazon.com/eks/latest/userguide/fargate-logging.html)

```hcl
 #---------------------------------------
  # FARGATE FLUENTBIT
  #---------------------------------------
  fargate_fluentbit_enable = true

  fargate_fluentbit_config = {
      output_conf  = <<EOF
[OUTPUT]
  Name cloudwatch_logs
  Match *
  region eu-west-1
  log_group_name /${local.cluster_name}/fargate-fluentbit-logs
  log_stream_prefix "fargate-logs-"
  auto_create_group true
    EOF
      filters_conf = <<EOF
[FILTER]
  Name parser
  Match *
  Key_Name log
  Parser regex
  Preserve_Key On
  Reserve_Data On
    EOF
      parsers_conf = <<EOF
[PARSER]
  Name regex
  Format regex
  Regex ^(?<time>[^ ]+) (?<stream>[^ ]+) (?<logtag>[^ ]+) (?<message>.+)$
  Time_Key time
  Time_Format %Y-%m-%dT%H:%M:%S.%L%z
  Time_Keep On
  Decode_Field_As json message
    EOF
  }
```

<!--- BEGIN_TF_DOCS --->
## Requirements

No requirements.

## Providers

| Name | Version |
|------|---------|
| <a name="provider_aws"></a> [aws](#provider\_aws) | n/a |
| <a name="provider_kubernetes"></a> [kubernetes](#provider\_kubernetes) | n/a |

## Modules

No modules.

## Resources

| Name | Type |
|------|------|
| [kubernetes_config_map.aws_logging](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs/resources/config_map) | resource |
| [kubernetes_namespace.aws_observability](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs/resources/namespace) | resource |
| [aws_region.current](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/region) | data source |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_eks_cluster_id"></a> [eks\_cluster\_id](#input\_eks\_cluster\_id) | EKS cluster Id | `string` | n/a | yes |
| <a name="input_fargate_fluentbit_config"></a> [fargate\_fluentbit\_config](#input\_fargate\_fluentbit\_config) | Fargate fluentbit configuration | `any` | `{}` | no |

## Outputs

No outputs.

<!--- END_TF_DOCS --->
