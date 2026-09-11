1. aws_iam_instance_profile, aws_iam_role_policy does not have a tag. Fixed by adding tags = { Project = var.project_tag } to match the pattern used everywhere else.

2. The subnets cidr range is contained in the vpc cidr range. so no changes needed here

3. Reused my existing ansible-key-pair key pair( created  earlier) rather than creating a new one, since I already held the matching .pem locally. Added a key_name variable and wired it into aws_instance.app — this was missing entirely in the starter, which would have left the instance unreachable via SSH despite the security group allowing port 22.