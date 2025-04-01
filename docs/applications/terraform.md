---
tags:
  - terraform
  - cheatsheet
---
# Terraform cheat sheet

## Error acquiring the state lock: ConditionalCheckFailedException

This error usually appears when one process fails running `terraform plan` or`
terraform apply`. For example if your network connection interrupts or the
process is terminated before finishing. Then Terraform "thinks" that this
process is still working on the infrastructure and blocks other processes from
working with the same infrastructure and state at the same time in order to
avoid conflicts:

```bash
Acquiring state lock. This may take a few moments...
╷
│ Error: Error acquiring the state lock
│ │ Error message: ConditionalCheckFailedException: The conditional request
│ failed
│ Lock Info:
│   ID:        9b87a784-117a-02bc-eb43-a04a45e7fe32
│   Path:      foo/bar/terraform.tfstate
│   Operation: OperationTypeApply
│   Who:       ary@foobar
│   Version:   1.5.5
│   Created:   2024-01-04 17:26:52.18230457 +0000 UTC
│   Info:   ```

```bash
terraform force-unlock -force <ID>
```

## Listing All Resources (Targets)

If you want to see a list of all resources (potential targets) managed by
Terraform within
your configuration, you can use the `terraform state list` command. This command
displays all the resources that Terraform is currently managing:

```bash
terraform state list
```

Then you can only modify this resource by:

```bash
terraform apply -target=aws_instance.my_instance
```
