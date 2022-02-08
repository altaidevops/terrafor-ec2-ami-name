properties([
    parameters([
        choice(choices: ['dev', 'qa', 'prod'], description: 'Choose environment', name: 'environment'),
        string(description: 'Enter AMI Name:', name: 'aminame', trim: true),
        choice(choices: ['plan', 'apply', 'destroy'], description: 'Choose Terraform Action', name: 'terraformaction')
    ])
])

if( params.environment == "dev" ){
    aws_region_var = "us-east-1"
}
else if( params.environment == "qa" ){
    aws_region_var = "us-east-2"
}
else if( params.environment == "prod" ){
    aws_region_var = "us-west-2"
}
else {
    error 'Parameter was not set'
}

def tfvars = """
s3_bucket = \"YOUR_BUCKET_NAME\"
s3_folder_project = \"terraform_ec2\"
s3_folder_region = \"us-east-1\"
s3_folder_type = \"class\"
s3_tfstate_file = \"infrastructure.tfstate\"
environment = \"${params.environment}\"

region        = \"${aws_region_var}\"	
public_key    = \"YOUR_LOCAL_PUBLIC_KEY\"	
ami_name      = \"${params.aminame}\"
"""

node('terraform'){
    stage("Pull Code"){
        git 'https://github.com/altaidevops/terraform-ec2-by-ami-name.git'

        writeFile file: "${params.environment}.tfvars", text: "${tfvars}"
    }

    withCredentials([usernamePassword(credentialsId: 'aws-key', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
        withEnv(["AWS_REGION=${aws_region_var}"]) {
            stage("Terraform Init"){
                sh """
                    #!/bin/bash
                    source ./setenv.sh ${params.environment}.tfvars

                    terraform init
                """
            }   
        
            if( params.terraformaction == "plan" ){
                stage("Terraform Plan"){
                    sh "terraform plan -var-file ${params.environment}.tfvars"
                }
            }
            
            if( params.terraformaction == "apply" ){
                stage("Terraform Apply"){
                    sh "terraform apply -var-file ${params.environment}.tfvars -auto-approve"
                }
            }

            if( params.terraformaction == "destroy" ){
                stage("Terraform Destroy"){
                    sh "terraform destroy -var-file ${params.environment}.tfvars -auto-approve"
                }
            }
        }        
    }
}
