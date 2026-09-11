# Kente Retail — Staging Infrastructure Spec

Written by the platform team after the console fat-finger incident. Codify this in
Terraform — don't click it together in the console. Document every gap you find, every
value you had to choose yourself, and why (or why not) in your Assumptions Log.

## 1. Network

- One VPC, one public subnet, one internet gateway, one route table sending
  `0.0.0.0/0` traffic out through the gateway.
- Single AZ is fine for staging — this is not a highly-available production spec.
- CIDR ranges are your call; document the ones you picked and make sure the subnet
  range is actually contained inside the VPC range.

## 2. Compute

- One EC2 instance running the staging app.
- Instance size: small enough to be cheap, big enough to actually run something.
  `t3.micro` is the reference size — if you pick something else, say why in your
  cost-estimate sizing justification.
- Must sit in the public subnet from section 1 and get a public IP so it's reachable
  for the verification step in the Acceptance Criteria.

## 3. Storage

- One S3 bucket for application data.
- Block all public access at the bucket level (public bucket policies/ACLs are not an
  acceptable way to make this reachable — that's exactly the kind of console
  fat-finger this lab exists to prevent).
- The bucket must actually accept an object as part of your lifecycle evidence — an
  empty, never-tested bucket does not satisfy the Acceptance Criteria.

## 4. Security group

- One security group attached to the EC2 instance.
- SSH (port 22) inbound restricted to a specific IP or CIDR range you control —
  **never `0.0.0.0/0`.** This is the single most-checked line in this entire spec;
  the grading script checks it directly.
- Whatever port your app listens on inbound, scoped the same way — to a specific
  range, not the whole internet, unless you have a documented reason the app must be
  public (and even then, SSH stays restricted regardless).
- Outbound: unrestricted is fine for a staging box.

## 5. Identity — IAM role

- One IAM role attached to the EC2 instance (via an instance profile) so the app can
  talk to its own S3 bucket without static credentials baked into the box.
- Scope the role's policy to *that specific bucket* (by ARN), not `s3:*` on `*`. A
  wildcard resource on an IAM policy is graded the same way an open security group is
  — as a finding, not a shortcut.

## 6. Tagging & naming convention (required — this is how teardown gets verified)

- Pick one project identifier for yourself, e.g. `kente-iac-<your-name-or-id>`, and:
  - Tag **every** resource above with `Project = <that identifier>`.
  - Name your S3 bucket and IAM role so they **start with** that same identifier
    (e.g. `kente-iac-jdoe-data`, `kente-iac-jdoe-ec2-role`).
- The instructor's teardown check greps the sandbox account for exactly this pattern.
  If your resources aren't tagged/named consistently, a clean teardown can't be
  confirmed even if you did destroy everything — don't lose points on a naming
  mismatch.

## 7. State

- Local state only for this module (a `terraform.tfstate` file on your own machine).
  Remote/shared state backends are Module 4's topic, not this one — don't reach for
  an S3 backend block here, even if you already know how.
- Do not commit `terraform.tfstate` or `terraform.tfstate.backup` to your repo — they
  can contain sensitive values in plain text. Your starter's `.gitignore` already
  excludes them; don't remove that.

## 8. Cost & teardown

- Sandbox budget is modest — this is a staging proof, not a production footprint.
  Favor the smallest instance/storage class that still lets you demonstrate the
  Acceptance Criteria honestly.
- Teardown deadline: **by end of the sprint this lab is assigned in** (your instructor
  will confirm the exact date/time at kickoff). Everything tagged with your project
  identifier must be gone — `terraform destroy` and then confirm, don't just assume it
  worked.

## 9. Out of scope for this pass

Multi-AZ/HA, autoscaling, a load balancer, remote state, and CI/CD wiring are not
assessed in this lab (several of these are later modules). If you notice a gap here,
note it in your Assumptions Log — you are not required to build it.
