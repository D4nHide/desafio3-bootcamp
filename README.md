# **Desafio # 3 Bootcamp DevOps Engineer**
Configurar y usar roles de AWS IAM desde la línea de comandos (CLI) para permitir la escritura en un bucket de S3.

1. Crear un bucket en s3, asignar un nombre único.
2. Crear un rol con una política que permita escribir en el bucket.
3. Generar un usuario IAM llamado s3-support y crear credenciales programáticas.
4. Actualizar la política del rol para que permita al usuario s3-support asumir el rol.
5. Conecta el CLI con las credenciales del usuario s3-support.
6. Asume el rol válido que permita escribir en el bucket.

# 1) Instalar AWS CLI

### Linux en ARM (Raspberry Pi)
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-aarch64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
![image](https://github.com/user-attachments/assets/a4e8596b-d6e4-4164-813f-0783780cc935)
### Mac OS 
```
brew install awscli
```
### Linux estándar (Ubuntu x86_64)
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
# 2) Chequear versión
```
aws --version
```
![image](https://github.com/user-attachments/assets/22cdcd31-038f-4e70-ac4c-9224d5dc1d34)

# 3) Crear un usuario IAM llamado s3-support con credenciales programáticas
```
aws iam create-user --user-name s3-support
```
![image](https://github.com/user-attachments/assets/a22b7fa1-2ffd-4ca3-9de6-4dc9787d4c3a)

```
aws iam create-access-key --user-name s3-support
```
![image](https://github.com/user-attachments/assets/f80197e8-5631-4c59-8e7c-dc78e8422321)

# 4) Crear un bucket en S3 y listarlo
```
aws s3 mb s3://nombre-de-mi-bucket
```
![image](https://github.com/user-attachments/assets/b8ef353b-4fc8-4651-8436-3e0294fdbfdf)

# 5) Crear una política que permita escribir en el bucket de S3
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::nombre-de-mi-bucket/*"
    }
  ]
}
```
## Guardar la politica s3-write-policy.json y ahora la creamos en aws
```
aws iam create-policy --policy-name nombre-de-la-politica --policy-document file://s3-write-policy.json
```
![image](https://github.com/user-attachments/assets/8e854e1f-61e6-4bdf-8687-de5829744533)

# 6) Crear un Role con la política de escritura en S3
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<YOUR_ACCOUNT_ID:user/nombre-de-usuario"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```
## Guardar trust-policy.json y ahora la creamos en aws
```
aws iam create-role --role-name Nombre-Role --assume-role-policy-document file://trust-policy.json
```
![image](https://github.com/user-attachments/assets/72b1b259-e7a3-4314-8ac0-9684e3c72523)

# 7) Actualizar la política del rol para que el usuario s3-support pueda asumir el rol
```
aws iam update-assume-role-policy --role-name Nombre-Role --policy-document file://trust-policy.json
```
![image](https://github.com/user-attachments/assets/91f77573-9439-4d7b-ab7b-7240bc0f0449)

# 8) Conecta el CLI con las credenciales del usuario s3-support (Pedirá las claves obtenidas en el paso 2)
```
aws configure --profile s3-support 
```
Se obtendrá una salida como se muestra a continuación que nos solicitará algunos datos

- AWS Access Key ID [None]: 
- AWS Secret Access Key [None]: 
- Default region name [None]: us-east-2
- Default output format [None]: json

![image](https://github.com/user-attachments/assets/6a93d029-d178-4ad0-bbd7-22a1f42f87b3)

# 9) Tratar de copiar un archivo al bucket creado
```
aws s3 cp README.md s3://nombre-de-mi-bucket/ --region us-east-2
```
![image](https://github.com/user-attachments/assets/2d749470-10eb-4955-ac2a-f8fc229e5e40)

El error que se muestra es porque  no tiene permisos el usuario y no se ha asumido el rol

# 10) Asumir el rol y exporta las credenciales temporales:
```
aws sts assume-role --role-arn arn:aws:iam::YOUR_ACCOUNT_ID:role/Nombre-Role --role-session-name s3session --profile s3-support
```
- export AWS_ACCESS_KEY_ID=<AccessKeyId>
- export AWS_SECRET_ACCESS_KEY=<SecretAccessKey>
- export AWS_SESSION_TOKEN=<SessionToken>

![image](https://github.com/user-attachments/assets/ef8d6925-2cb6-471f-b4ed-b8f459eb2353)

# 11) Adjuntar la política al rol
```
aws iam attach-role-policy --role-name Nombre-Role --policy-arn arn:aws:iam::YOUR_ACCOUNT_ID:policy/nombre-de-la-politica 
```
![image](https://github.com/user-attachments/assets/3d2a854b-4812-42c5-88b4-67fc9761161e)

# 12) Escritura en bucket
```
aws s3 cp archivo s3://nombre-de-mi-bucket/ --region us-east-2
```
![image](https://github.com/user-attachments/assets/d5d06297-db18-4097-9580-c0cd5d417635)
