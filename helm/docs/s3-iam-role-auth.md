# S3 storage: IAM role (EKS Pod Identity) or access/secret key

New value `s3Storage.iamRoleAuth` selects the authentication mode.

| `iamRoleAuth` | Behaviour |
|---|---|
| `true`  | Sets `BOLD_SERVICES_AWS_AUTH_TYPE=IAMRole` and `serviceAccountName` on the pods. No access/secret key env vars are rendered. |
| `false` (default) | Renders `BOLD_SERVICES_AWS_ACCESS_KEY_ID` and `BOLD_SERVICES_AWS_SECRET_ACCESS_KEY` from `accessKey` / `secretKey`. No auth-type var, no service account. |

## Steps performed
1. `boldbi/templates/deployment.yaml`: added `serviceAccountName` only when `s3Storage.enabled` and `s3Storage.iamRoleAuth` are true.
2. Same file, S3 env block: if `iamRoleAuth` is true emit `BOLD_SERVICES_AWS_AUTH_TYPE=IAMRole`; otherwise emit the access and secret key variables. Bucket, region, root folder and storage type are always emitted.
3. `boldbi/values.yaml` and `custom-values/eks-alb-values.yaml`: added `iamRoleAuth: false` and `serviceAccountName: boldbi-sa`.

## Usage (IAM role)
Create the service account and Pod Identity association (service account name must match `serviceAccountName`, same namespace), then:
```
helm install boldbi ./boldbi -f custom-values/eks-alb-values.yaml \
  --set s3Storage.enabled=true --set s3Storage.iamRoleAuth=true \
  --set s3Storage.bucketName=<bucket> --set s3Storage.region=<region> --set s3Storage.storageType=<type>
```
The chart does not create the service account.
