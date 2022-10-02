## What?

[Karpenter](https://karpenter.sh) is a Kubernetes just-in-time scaler that'll eventually be for all providers, but is currently only for EKS. 

## How to try it?

[Click here to open in Gitpod](https://gitpod.io/#https://github.com/martyjhenderson/karpenter-demo)

**Note:** This will cost real money, around $0.50.

This is basically some light formatting changes of [karpenter's getting started with terraform](https://karpenter.sh/v0.16.3/getting-started/getting-started-with-terraform/) and a few Terraform cleanup things.

This is assuming you have AWS credentials in Gitpod, otherwise, run `aws configure` and follow the directions.

Once that is done

1. Run `aws iam create-service-linked-role --aws-service-name spot.amazonaws.com`
    * If this throws an error, that's fine, it means you already linked it
1. Go to the `/terraform` directory
1. Run `terraform init` and then `terraform apply`
1. Wait - EKS takes a hot minute.
1. Go back to the root directory and deploy the inflation deployment AWS wrote with `kubectl apply -f inflate.yaml`
    * This will deploy with no pods active
1. Scale up the inflate with `kubectl scale deployment inflate --replicas 5`
1. Watch the node management with `kubectl logs -f -n karpenter -l app.kubernetes.io/name=karpenter -c controller` (and use c-c to escape)
1. Remove the deployment with `kubectl delete deployment inflate`
1. After 30 seconds, go back to watching nodes with `kubectl logs -f -n karpenter -l app.kubernetes.io/name=karpenter -c controller` and see how the empty nodes drop off.
1. Once you're done playing wiht it nuke it with the following commands:
    * `kubectl delete deployment inflate` (This is a repeate of step 8)
    * `kubectl delete node -l karpenter.sh/provisioner-name=default`
    * `terraform destroy` - EKS and components take a while to shut down, so you might be here a few minutes.
    * `aws ec2 describe-launch-templates 
    | jq -r ".LaunchTemplates[].LaunchTemplateName" 
    | grep -i "Karpenter-karpenter-demo" 
    | xargs -I{} aws ec2 delete-launch-template --launch-template-name {}`