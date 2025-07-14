Here are several Terraform coding challenge problem statements you can use to evaluate senior-level skills. Each is phrased as a standalone exercise—feel free to pick and choose:

---

### Problem 1: Multi-Environment Module with Workspaces

**Statement:**
Create a Terraform module `shared-network` that provisions a VPC, subnets, and route tables across three availability zones. The module must:

* Support two environments, “staging” and “production”, selected via workspaces.
* In “staging”, create one public and one private subnet per AZ; in “production”, create two of each.
* Tag all resources with `Environment = "<workspace>"`.
* Expose outputs: VPC ID, lists of subnet IDs, and public route table IDs.

**Requirements:**

* Use `count` or `for_each` so you don’t repeat resource blocks.
* Detect workspace (`terraform.workspace`) to branch logic.
* Demonstrate calling the module in a root configuration for both workspaces.

**Acceptance Criteria:**

* `terraform plan -var="environment=staging"` shows exactly 3 public + 3 private subnets.
* Switching to “production” workspace doubles the subnets.
* All tags correctly reflect the workspace name.

---

### Problem 2: Dynamic IAM Policy Generator

**Statement:**
Write a Terraform configuration that builds IAM policies from a single variable map. Given:

```hcl
variable "policy_statements" {
  type = list(object({
    sid       = string
    actions   = list(string)
    resources = list(string)
  }))
}
```

Generate a single `aws_iam_policy` whose JSON is constructed via `jsonencode()`, iterating over `policy_statements`.

**Requirements:**

* Use a local value or data block to assemble the `statement` list.
* Ensure the final JSON matches AWS’s policy schema.
* No hard-coded statement—everything driven by the variable.

**Acceptance Criteria:**

* Given two statements in the var, plan shows an IAM policy with exactly those two `Statement` entries.
* Adding/removing entries in `policy_statements` updates the policy accordingly.

---

### Problem 3: Remote State Data Source Chain

**Statement:**
You have two Terraform projects:

1. **VPC stack** outputs a map `subnet_ids` keyed by AZ name.
2. **App stack** needs to consume specific subnets from the VPC stack.

Write the “App stack” configuration to:

* Retrieve the VPC stack’s remote state from an S3 backend.
* Select only the private subnets whose AZ ends with “a” or “c”.
* Provision an auto-scaling group using those filtered subnets.

**Requirements:**

* Use `terraform_remote_state` data source.
* Filter the map using `for` expressions and `regex()` or `slice()`.
* Pass the subnet list into the ASG resource.

**Acceptance Criteria:**

* Plan shows ASG attached to exactly the two desired subnet IDs.
* No manual listing of AZs in the App stack.

---

### Problem 4: Custom Provisioner via Null Resource

**Statement:**
Sometimes you need to bootstrap databases before Terraform can import schemas. Create a `null_resource` that:

* Runs a shell script (via `local-exec`) to wait for an RDS instance endpoint to become available (use AWS CLI stub).
* On success, writes the endpoint to a file in `${path.module}/endpoint.txt`.
* Only re-runs if the RDS instance’s endpoint or engine version changes.

**Requirements:**

* Use `triggers = { ... }` on the `null_resource` to detect changes.
* Script should retry with exponential backoff (e.g., up to 5 times).
* Demonstrate in a plan/apply with a mocked RDS resource.

**Acceptance Criteria:**

* First apply creates `endpoint.txt` with the mock endpoint.
* Changing the RDS engine version causes re-run.
* No re-run on unrelated changes (e.g., tags).

---

### Problem 5: Writing a Simple Custom Provider (Design Only)

**Statement:**
Skip actual Go code—draft the HCL and Go entry-points for a provider `terraform-provider-notifier` that sends Slack alerts. Specifically:

* HCL usage example:

  ```hcl
  provider "notifier" {
    token = var.slack_token
  }

  resource "notifier_alert" "deploy" {
    channel = "#deployments"
    message = "Deployment complete for ${var.app_name}"
  }
  ```
* In Go, outline:

  * Provider schema (token).
  * Resource schema (channel, message).
  * Where you’d implement Create/Read/Update/Delete (just signatures and high-level comments).

**Requirements:**

* Show a minimal `main.go`/`provider.go` and `resource_alert.go` scaffolding.
* Explain how you’d handle secrets and rate-limiting.

**Acceptance Criteria:**

* Clear mapping from HCL to Go types.
* Thoughtful notes on error handling and idempotency.

---

### Problem 6: Terraform Testing with Terratest (Go)

**Statement:**
Design a Go test (using Terratest) that verifies your S3 module:

* Creates a bucket with versioning enabled and tagging.
* Checks that bucket ACL is private.
* Cleans up the resources afterward.

**Requirements:**

* Show the Go test code structure: `TestAwsS3Module`, `terraform.Options`, `defer terraform.Destroy`.
* Use `terraform.InitAndApply` and `aws.GetBucketVersioning` / `aws.GetBucketAcl`.

**Acceptance Criteria:**

* Test compiles and logically covers create, verify, and destroy phases.
* Proper assertions for versioning status and ACL grants.

---

Each of these problem statements tests a different aspect of Terraform coding—from pure HCL module design to real-world integration and testing. Adjust the scope or time allotment as needed!
