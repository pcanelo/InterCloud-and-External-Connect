# PoC completa: Acceso seguro a Amazon S3 con IAM Roles Anywhere

## Prerrequisitos

Antes de comenzar, asegúrese de tener lo siguiente:

- AWS CLI instalada y configurada.
- OpenSSL instalado en su máquina local.
- Permisos de IAM adecuados para crear roles, perfiles y anclajes de confianza en AWS.
- Rol de IAM con los siguientes permisos para operar sobre S3.

## 1. Política de IAM para S3

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListAllMyBuckets",
                "s3:GetObject",
                "s3:PutObject",
                "s3:DeleteObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::nombre-del-bucket",
                "arn:aws:s3:::nombre-del-bucket/*"
            ]
        }
    ]
}
```

> 🎯 Reemplaza `"nombre-del-bucket"` por el bucket real que desees acceder.

## 2. Relación de confianza para el rol de IAM

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "rolesanywhere.amazonaws.com"
            },
            "Action": [
                "sts:AssumeRole",
                "sts:SetSourceIdentity",
                "sts:TagSession"
            ]
        }
    ]
}
```

## 3. Crear una autoridad de certificación (CA) raíz

```bash
openssl genrsa -out privateRootCA.key 2048

openssl req -x509 -new -nodes -key privateRootCA.key -sha256 -days 365 -out privateRootCA.pem \
-subj "/C=CL/O=Learning DevSecOps" \
-addext "basicConstraints=critical,CA:TRUE" \
-addext "keyUsage=critical,keyCertSign,cRLSign"
```

## 4. Crear un trust anchor (anclaje de confianza)

```bash
aws rolesanywhere create-trust-anchor \
--name "MyTrustAnchor" \
--source "sourceType=CERTIFICATE_BUNDLE,sourceData={x509CertificateData=\"$(cat privateRootCA.pem)\"}"
```

## 5. Crear un certificado de cliente

```bash
openssl genrsa -out client.key 2048

openssl req -new -key client.key -out client.csr -config <(cat <<-EOF
[req]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn
[dn]
C = CL
O = Learning DevSecOps
CN = ClientCert
[v3_ext]
basicConstraints = CA:FALSE
keyUsage = digitalSignature
extendedKeyUsage = clientAuth
EOF
)

openssl x509 -req -in client.csr -CA privateRootCA.pem -CAkey privateRootCA.key -CAcreateserial \
-out client.crt -days 365 -sha256 -extfile <(echo -e "basicConstraints = CA:FALSE\nkeyUsage = digitalSignature\nextendedKeyUsage = clientAuth")
```

## 6. Crear un perfil en AWS IAM Roles Anywhere

```bash
aws rolesanywhere create-profile \
--name "S3AccessProfile" \
--role-arns "arn:aws:iam::<AWS_ACCOUNT_ID>:role/<RoleName>" \
--trust-anchor-arn "arn:aws:rolesanywhere:<AWS_REGION>::trust-anchor/<TrustAnchorID>" \
--enabled
```

## 7. Instalar AWS Signing Helper

```bash
curl -O https://rolesanywhere.amazonaws.com/releases/1.4.0/X86_64/Linux/aws_signing_helper
chmod +x aws_signing_helper
sudo mv aws_signing_helper /usr/local/bin/
```

## 8. Asumir el rol con el certificado del cliente

```bash
aws_signing_helper credential-process \
--trust-anchor-arn arn:aws:rolesanywhere:<region>:<account_id>:trust-anchor/<TrustAnchorID> \
--role-arn arn:aws:iam::<account_id>:role/<RoleName> \
--profile-arn arn:aws:rolesanywhere:<region>:<account_id>:profile/<ProfileID> \
--certificate client.crt \
--private-key client.key \
--session-tag SourceIdentity=rolesanywhere-client
```

## 9. Exportar credenciales temporales

```bash
export AWS_ACCESS_KEY_ID=<AccessKeyId>
export AWS_SECRET_ACCESS_KEY=<SecretAccessKey>
export AWS_SESSION_TOKEN=<SessionToken>
```

## 10. Verificación de acceso a S3

```bash
aws s3 ls
echo "Hola desde Roles Anywhere" > prueba.txt
aws s3 cp prueba.txt s3://nombre-del-bucket/
aws s3 cp s3://nombre-del-bucket/prueba.txt .
aws s3 rm s3://nombre-del-bucket/prueba.txt
```

## Recomendación de seguridad adicional

```json
"Condition": {
  "StringEquals": {
    "aws:SourceIdentity": "rolesanywhere-client"
  }
}
```
