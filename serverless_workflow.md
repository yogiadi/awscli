cat step_functions_basic_execution_cli.json

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "states.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}

aws iam create-role --role-name step_functions_basic_execution_cli --assume-role-policy-document file://step_functions_basic_execution_cli.json

aws iam attach-role-policy --role-name step_functions_basic_execution_cli  --policy-arn "arn:aws:iam::aws:policy/service-role/AWSLambdaRole"

aws iam list-attached-role-policies --role-name step_functions_basic_execution_cli


