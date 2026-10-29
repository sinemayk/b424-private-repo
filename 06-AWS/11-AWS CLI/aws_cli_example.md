# Key pair oluştur
aws ec2 create-key-pair --key-name MyKeyPair --query 'KeyMaterial' --output text > MyKeyPair.pem

# İzinleri ayarla (dosya içindeki izinleri sınırlayın)
chmod 400 MyKeyPair.pem
# Security group oluştur
aws ec2 create-security-group --group-name MySecurityGroup --description "My security group"

# Security group'a inbound kurallar ekle (22 ve 80 portları açık)
aws ec2 authorize-security-group-ingress --group-name MySecurityGroup --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-name MySecurityGroup --protocol tcp --port 80 --cidr 0.0.0.0/0
# EC2 oluştur
aws ec2 run-instances --image-id ami-0005e0cfe09cc9050 --instance-type t2.micro --key-name MyKeyPair --security-groups MySecurityGroup
# EC2 örneğinizin ID'sini alın
export instance_id=$(aws ec2 describe-instances --query 'Reservations[].Instances[].InstanceId' --output text)

# Public IP bilgisini alın
export public_ip=$(aws ec2 describe-instances --instance-ids $instance_id --query 'Reservations[].Instances[].PublicIpAddress' --output text)

# SSH bağlantısı
ssh -i MyKeyPair.pem ec2-user@54.173.65.253



# S3 bucket oluştur
aws s3 mb s3://my-static-website-bucketbekir

# index.html dosyasını S3 bucket'ına kopyala
aws s3 cp index.html s3://my-static-website-bucketbekir/
aws s3 cp amasya_landscape.jpg s3://my-static-website-bucketbekir/


# S3 bucket'ını static website olarak yapılandır
aws s3 website s3://my-static-website-bucketbekir/ --index-document index.html


# S3 bucket'ının website endpoint'ini al
website_endpoint=$(aws s3 get-bucket-website --bucket my-static-website-bucketbekir --query 'Endpoint' --output text)

# DNS adresini ekrana yazdır
echo "Static Website DNS Adresi: $website_endpoint"


bucket_name="your-bucket-name"
region="your-region"

website_url=$(aws s3api get-bucket-website --bucket $bucket_name --query 'RedirectAllRequestsTo.HostName' --output text)
website_url="http://$website_url"

echo "Website URL: $website_url"


scp -i ~/.ssh/key.pem /home/ubuntu/file.txt ubuntu@destination-instance-ip:/home/ubuntu/


aws s3api put-bucket-policy --bucket your-bucket-name --policy "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Sid\":\"PublicReadGetObject\",\"Effect\":\"Allow\",\"Principal\":\"*\",\"Action\":\"s3:GetObject\",\"Resource\":\"arn:aws:s3:::your-bucket-name/*\"}]}"