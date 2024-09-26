deneme
# GitHub Action ile Basit Apache Sunucuya Deploy İşlemi
Bu projede, GitHub Action kullanarak bir Ubuntu sunucuya Apache Web Sunucusu kurulumu ve ardından index.html dosyasını deploy ediyoruz. Projeye eklediğimiz her değişiklik, GitHub Actions aracılığıyla otomatik olarak sunucuda dağıtılır.

# Adım 1: Ubuntu Sunucusuna Apache Kurulumu
Sunucuda, index.html dosyasını dağıtabilmemiz için Apache Web Server kurulmuş olmalıdır. Aşağıdaki komut ile Apache kurulumu yapabilirsiniz:

```bash
sudo apt update
sudo apt install apache2 -y
sudo systemctl start apache2
```

# Adım 2: SSH Anahtarları Oluşturma
GitHub Actions, sunucuya bağlanmak için SSH anahtarları kullanır. İki önemli adım vardır:

Private Key: GitHub Action'da Secret olarak saklanır.
Public Key: Sunucuya ~/.ssh/authorized_keys dosyasına eklenir.

# SSH Anahtar Çifti Oluşturma
Yerel bilgisayarınızda SSH anahtar çifti oluşturun:
```bash
ssh-keygen -t rsa -b 4096 
```
Oluşan id_rsa.pub dosyasındaki public key'i sunucudaki ~/.ssh/authorized_keys dosyasına ekleyin.
id_rsa dosyasındaki private key'i GitHub reposundaki Secret'lara ekleyin.

# Adım 3: GitHub Action Workflow Yapısı
Bu proje, main dalına yapılan her push işleminde, dosyaları sunucuya kopyalayarak otomatik olarak deploy eder.

.github/workflows/deploy.yml dosyasının içeriği şu şekildedir:
```bash
name: Deploy Index HTML

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest  
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup SSH Key
        run: |
          echo "${{ secrets.DEPLOY_KEY }}" > deploy_key
          chmod 600 deploy_key
          ls -l

      - name: Check SSH Key
        run: |
          cat deploy_key
          echo "SSH key loaded successfully."

      - name: Copy index.html to server
        env:
          SERVER_IP: ${{ secrets.SERVER_IP }}  # Sunucu IP adresi
          USERNAME: ${{ secrets.USERNAME }}  # SSH kullanıcı adı
        run: |
          echo "Deploying index.html to server..."
          rsync -avz -e "ssh -o StrictHostKeyChecking=no -i deploy_key" index.html $USERNAME@$SERVER_IP:/var/www/html/
```
# Adım 4: GitHub Secrets Tanımlama
GitHub repo'nuzda aşağıdaki Secrets'ları tanımlayın:

DEPLOY_KEY: SSH Private Key (id_rsa dosyasının içeriği)
SERVER_IP: Sunucunuzun IP adresi
USERNAME: SSH kullanıcı adı (genellikle root veya başka bir kullanıcı)

# Adım 5: index.html Dosyası
Aşağıda örnek bir index.html dosyası bulunmaktadır. Bu dosya, sunucunuzda /var/www/html/ dizinine kopyalanacaktır.
```bash
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>GitHub Action Deploy</title>
</head>
<body>
  <h1 style="text-align: center;">GITHUB ACTION BAŞLANGIÇ</h1>
  <p style="text-align: center;">Bu sayfa, GitHub Actions kullanılarak deploy edilmiştir.</p>
</body>
</html>

```
