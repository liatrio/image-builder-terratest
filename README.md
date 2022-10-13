## What is the image for?
The intended purpose of this image is for it to be used as a Jenkins agent. By using the installed features the user is able to create Jenkins pipelines that can trigger Terraform scripts and test them using [Terratest](https://terratest.gruntwork.io/docs/getting-started/quick-start/). Additionally, the aws-iam-authenticator tool allows users to test Terraform in AWS. An example of using this image as a Jenkins agent via [Kubernetes](https://plugins.jenkins.io/kubernetes/) can be seen below. 

First, an example of configuring the pod template in yaml to create the agent.

```yaml
jenkins:
  clouds:
    - kubernetes:
        name: "kubernetes"
        templates:
          - name: "image-builder-terratest"
            label: "image-builder-terratest"
            nodeUsageMode: NORMAL
            containers:
              - name: "image-terratest"
                image: "ghcr.io/liatrio/image-builder-terratest:${builder_images_version}"
```
And then specifying the agent in the Jenkinsfile for an example step.

```jenkins
stage('Build') {
  agent {
    label "image-builder-terratest"
  }
  steps {
    container('image-terratest') {
      sh "aws-iam-authenticator help"
      sh "terraform plan"
      sh "go test -v -run TestTerraformAwsHelloWorldExample"
    }
  }
}
```

## What is installed on this image?
- Version [1.2.X](https://releases.hashicorp.com/terraform/1.2.9/) of infrastructure as code tool Terraform
- Version [1.21.X](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html) of command-line tool aws-iam-authenticator
- Version [1.25.X](https://storage.googleapis.com/kubernetes-release/release/v1.25.0/bin/linux/amd64/kubectl) of kubectl, a command-line tool that allows you to run commands against Kubernetes clusters
- Packages included in [Alpine 3.15](https://alpinelinux.org/posts/Alpine-3.15.0-released.html)
