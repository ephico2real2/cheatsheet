variable "ecs_cluster_arn" {
  description = "ARN of the ECS cluster"
  type        = string
}

variable "task_definition_arn" {
  description = "ARN of the Task Definition"
  type        = string
}

variable "schedule_expression" {
  description = "Schedule expression for the task"
  default     = "rate(1 day)"
  type        = string
}

variable "task_count" {
  description = "Number of tasks to run"
  default     = 1
  type        = number
}

resource "aws_cloudwatch_event_rule" "ecs_scheduled_task" {
  name                = "ecs-scheduled-task"
  description         = "Trigger ECS task on a schedule"
  schedule_expression = var.schedule_expression
}

resource "aws_cloudwatch_event_target" "ecs_task" {
  rule      = aws_cloudwatch_event_rule.ecs_scheduled_task.name
  arn       = var.ecs_cluster_arn
  role_arn  = aws_iam_role.ecs_events_role.arn
  input     = jsonencode({
    containerOverrides = [
      {
        name = "container-name"
        command = ["start"]
      }
    ]
  })

  ecs_target {
    task_count          = var.task_count
    task_definition_arn = var.task_definition_arn
  }
}

resource "aws_iam_role" "ecs_events_role" {
  name = "ecs_events_role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Action = "sts:AssumeRole",
        Effect = "Allow",
        Principal = {
          Service = "events.amazonaws.com"
        }
      },
      {
        Action = "sts:AssumeRole",
        Effect = "Allow",
        Principal = {
          Service = "ecs-tasks.amazonaws.com"
        }
      }
    ]
  })
}

resource "aws_iam_policy" "ecs_tasks_policy" {
  name        = "ecs_tasks_policy"
  description = "Policy to allow ECS tasks to run"

  policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Action = [
          "ecs:RunTask",
          // Add other necessary ECS actions here
        ],
        Effect = "Allow",
        Resource = "*"
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "ecs_tasks_attach" {
  role       = aws_iam_role.ecs_events_role.name
  policy_arn = aws_iam_policy.ecs_tasks_policy.arn
}






Let's adapt the pipeline for ECS tasks with parameterized options for the Terraform module we discussed earlier. The pipeline will include parameters for the ECS cluster ARN, task definition ARN, schedule expression, and the desired task count.

Here's how your `Jenkinsfile` might look with these parameters:

```groovy
pipeline {
    agent any
    parameters {
        string(name: 'ECS_CLUSTER_ARN', defaultValue: '', description: 'ARN of the ECS cluster')
        string(name: 'TASK_DEFINITION_ARN', defaultValue: '', description: 'ARN of the Task Definition')
        string(name: 'SCHEDULE_EXPRESSION', defaultValue: 'rate(1 day)', description: 'Schedule expression for the task')
        string(name: 'TASK_COUNT', defaultValue: '1', description: 'Number of tasks to run')
    }
    environment {
        // Environment variables for AWS credentials
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://generic-git-repo-url.com/your-organization/terraform-repo.git'
            }
        }
        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }
        stage('Terraform Plan') {
            steps {
                sh """
                    terraform plan \
                        -var 'ecs_cluster_arn=${params.ECS_CLUSTER_ARN}' \
                        -var 'task_definition_arn=${params.TASK_DEFINITION_ARN}' \
                        -var 'schedule_expression=${params.SCHEDULE_EXPRESSION}' \
                        -var 'task_count=${params.TASK_COUNT}' \
                        -out=tfplan
                   """
            }
        }
        stage('Terraform Apply') {
            steps {
                script {
                    def userInput = input(id: 'userInput', message: 'Apply Terraform plan?', parameters: [booleanParam(defaultValue: false, description: 'Check to apply', name: 'APPLY')])
                    if (userInput) {
                        sh 'terraform apply tfplan'
                    } else {
                        error "User rejected Terraform apply"
                    }
                }
            }
        }
    }
    post {
        always {
            // Include any cleanup or additional steps if needed
        }
    }
}
```

This script does the following:

- Defines parameters at the beginning of the pipeline which can be specified when the pipeline is triggered.
- Uses the `params` variable to pass the parameter values into Terraform commands.
- Executes `terraform plan` with the provided parameters to generate an execution plan.
- Prompts the user to approve the `terraform apply` step.

Before using this in a production environment, please test this pipeline in a development or staging environment. Remember to replace the placeholder for the Git repository URL with the actual URL of your Terraform configuration repository. Ensure the AWS credentials (identified by 'AWS_ACCESS_KEY_ID' and 'AWS_SECRET_ACCESS_KEY') are properly configured in your Jenkins credentials store.
