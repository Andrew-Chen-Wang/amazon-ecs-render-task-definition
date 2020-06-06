## Amazon ECS "Render Task Definition" Action for GitHub Actions

Inserts a container image URI into an Amazon ECS task definition JSON file, creating a new task definition file.

**Table of Contents**

<!-- toc -->

- [Usage](#usage)
- [License Summary](#license-summary)
- [Security Disclosures](#security-disclosures)

<!-- tocstop -->

## Usage

To insert the image URI `amazon/amazon-ecs-sample:latest` as the image for the `web` container in the task definition file, and then deploy the edited task definition file to ECS:

```yaml
    - name: Configure AWS credentials from Test account
      id: aws-credentials-configuration
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.TEST_AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.TEST_AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1

    - name: Render Amazon ECS task definition
      id: render-web-container
      uses: aws-actions/amazon-ecs-render-task-definition@v1
      with:
        task-definition: task-definition.json
        container-name: web
        image: amazon/amazon-ecs-sample:latest
        secrets: external-secrets.json
        region-name: us-east-2
        aws-account-id: ${{ steps.aws-credentials-configuration.outputs.aws-account-id }}

    - name: Deploy to Amazon ECS service
      uses: aws-actions/amazon-ecs-deploy-task-definition@v1
      with:
        task-definition: ${{ steps.render-web-container.outputs.task-definition }}
        service: my-service
        cluster: my-cluster
```

If your task definition file holds multiple containers in the `containerDefinitions`
section which require updated image URIs, chain multiple executions of this action
together using the output value from the first action for the `task-definition`
input of the second:

```yaml
    - name: Render Amazon ECS task definition for first container
      id: render-web-container
      uses: aws-actions/amazon-ecs-render-task-definition@v1
      with:
        task-definition: task-definition.json
        container-name: web
        image: amazon/amazon-ecs-sample-1:latest

    - name: Modify Amazon ECS task definition with second container
      id: render-app-container
      uses: aws-actions/amazon-ecs-render-task-definition@v1
      with:
        task-definition: ${{ steps.render-web-container.outputs.task-definition }}
        container-name: app
        image: amazon/amazon-ecs-sample-2:latest

    - name: Deploy to Amazon ECS service
      uses: aws-actions/amazon-ecs-deploy-task-definition@v1
      with:
        task-definition: ${{ steps.render-app-container.outputs.task-definition }}
        service: my-service
        cluster: my-cluster
```

If you choose to add environment variables using the "secrets" input, add the "secrets" input
and set it to `true`, your AWS "region-name", and AWS account ID (from configure AWS credentials
action) in your GitHub workflow and add the secrets section to your 
[task definition](https://aws.amazon.com/premiumsupport/knowledge-center/ecs-data-security-container-task/).
You can either use the task definition file, an external file, or both (the last two, you must specify the
filename and have the secrets array in the containerDefinition).
The "valueFrom" in your task definition should include this specific format:

```
{
    "name": "var-from-parameter-store",
    "valueFrom": "ssm:/path/to/variable"
},
{
    "name": "var-from-secrets-manager",
    "valueFrom": "secretsmanager:/path/to/variable"
}
```

If you use an external JSON file for secrets, then your secrets input
should be the external JSON file and the file should have the format
that must abide as follows:

```
{
    secrets: [
        {
            "name": "var-from-parameter-store",
            "valueFrom": "ssm:/path/to/variable"
        },
        {
            "name": "var-from-secrets-manager",
            "valueFrom": "secretsmanager:/path/to/variable"
        }
    ]
}
```

Notice the "ssm" and "secretsmanager" prefix, both of which are followed by a ":" (colon) and variable path.
The parameter path must start with "/" (a forward slash). You can have an external secrets JSON
file AND secrets already inside of your task definition as they'll be combined. 

See [action.yml](action.yml) for the full documentation for this action's inputs and outputs.

## License Summary

This code is made available under the MIT license.

## Security Disclosures

If you would like to report a potential security issue in this project, please do not create a GitHub issue.  Instead, please follow the instructions [here](https://aws.amazon.com/security/vulnerability-reporting/) or [email AWS security directly](mailto:aws-security@amazon.com).
