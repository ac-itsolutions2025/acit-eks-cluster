pipeline {
  agent any

  environment {
    STACK_NAME      = 'acit-eks-demo-cluster'
    TEMPLATE_FILE   = 'eks-cluster-with-addons.yaml'
    REGION          = 'us-east-2'

    CLUSTER_NAME    = 'acit-eks'
    K8S_VERSION     = '1.33'
    VPC_ID          = 'vpc-0ab04656e45f63cb7'            // Replace with actual VPC ID
    SUBNET_IDS      = 'subnet-0f466e83f27e135df,subnet-03d42a7f417e15476'  // Must be in different AZs
    ADMIN_ROLE_ARN  = 'arn:aws:iam::124355683348:role/acit-EKSClusterRole'
    NODE_ROLE_ARN   = 'arn:aws:iam::124355683348:role/acit-EKSNodeInstanceRole'
    NODEGROUP_NAME  = 'acit-nodegroup'
    INSTANCE_TYPES  = 't3.xlarge'
    DESIRED_CAP     = '2'
    MIN_SIZE        = '1'
    MAX_SIZE        = '4'
  }

  options {
    timestamps()
    disableConcurrentBuilds()
  }

  stages {
    stage('Checkout Template') {
      steps {
        echo '📦 Pulling CloudFormation template'
        checkout scm
      }
    }

    stage('Lint Template') {
      steps {
        echo '🔍 Validating template with cfn-lint'
        sh '''
          python3 -m venv .venv
          . .venv/bin/activate
          .venv/bin/pip install --upgrade pip cfn-lint
          .venv/bin/cfn-lint "$TEMPLATE_FILE"
        '''
      }
    }

    stage('Deploy EKS Stack') {
      steps {
        echo '🚀 Deploying EKS control plane, node group, and add-ons'
        sh '''
          aws cloudformation deploy \
            --stack-name "$STACK_NAME" \
            --template-file "$TEMPLATE_FILE" \
            --region "$REGION" \
            --capabilities CAPABILITY_NAMED_IAM \
            --parameter-overrides \
              ClusterName="$CLUSTER_NAME" \
              KubernetesVersion="$K8S_VERSION" \
              VpcId="$VPC_ID" \
              SubnetIds="$SUBNET_IDS" \
              ClusterAdminRoleArn="$ADMIN_ROLE_ARN" \
              NodeInstanceRoleArn="$NODE_ROLE_ARN" \
              NodeGroupName="$NODEGROUP_NAME" \
              InstanceTypes="$INSTANCE_TYPES" \
              DesiredCapacity="$DESIRED_CAP" \
              MinSize="$MIN_SIZE" \
              MaxSize="$MAX_SIZE"
        '''
      }
    }

    stage('Describe Stack Outputs') {
      steps {
        echo '📡 EKS Deployment Outputs:'
        sh '''
          aws cloudformation describe-stacks \
            --stack-name "$STACK_NAME" \
            --region "$REGION" \
            --query "Stacks[0].Outputs[*].[OutputKey,OutputValue]" \
            --output table
        '''
      }
    }
  }

  post {
    always {
      echo '🧼 Cleaning up virtualenv...'
      sh 'rm -rf .venv || true'
    }
    success {
      echo '✅ EKS platform deployed successfully!'
    }
    failure {
      echo '❌ Stack deployment failed. Check logs above.'
    }
  }
}
