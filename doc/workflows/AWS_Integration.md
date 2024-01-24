# Integrating AWS with workflows

1. Create an IAM role in AWS.
    - Ensure you pick **Web Identity** for **Trusted entity type**
    - There should be an already created github identity for you to choose.

2. Create a policy with the following permissions (This is the bare minimum for Github Actions to use the role)

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Resource": [
                    "*"
                ],
                "Action": "sts:GetCallerIdentity",
                "Effect": "Allow"
            }
        ]
    }
    ```

3. Create another AWS policy for what your role needs to do (s3, ec2, etc..)
4. Once the role is created and all the policies are attached, you need to go the the roles "Trust Settings"
5. Ensure the following trust policy is set on the role.

   ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": {
                    "Federated": "arn:aws:iam::754634071668:oidc-provider/token.actions.githubusercontent.com"
                },
                "Action": "sts:AssumeRoleWithWebIdentity",
                "Condition": {
                    "StringEquals": {
                        "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
                    },
                    "StringLike": {
                        "token.actions.githubusercontent.com:sub": "repo:DEMGroup/<YOUR_REPO_NAME_HERE>:*"
                    }
                }
            }
        ]
    }
    ```

6. Use the following snippet in your yaml file

    ```yaml
        - name: Configure AWS credentials from Test account
        uses: aws-actions/configure-aws-credentials@v1
        with:
            role-to-assume: ARN of the role you created.
            aws-region: eu-west-1
    ```

7. Run a builD
