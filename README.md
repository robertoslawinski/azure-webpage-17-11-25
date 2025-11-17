# CI/CD Website on VM do Azure (via GitHub Actions + SSH)

Este repositório mostra como fazer o **deploy automático** de um site estático para uma
**máquina virtual Ubuntu no Azure**, usando **GitHub Actions** e **SSH**.

Fluxo geral:

1. Gerar um par de chaves SSH no PC.
2. Autorizar a chave pública na VM (usuário `gestor`).
3. Criar um repositório no GitHub e configurar **Environment + Secrets**.
4. Criar um workflow (`.github/workflows/blank.yml`) que:
   - decodifica a chave privada a partir do secret,
   - conecta na VM via SSH,
   - instala/atualiza o Apache,
   - publica o conteúdo do site.

---

## 1. Pré-requisitos

- Conta Azure com uma **VM Linux (Ubuntu)** já criada.
- A VM deve ter:
  - Porta **22 (SSH)** aberta.
  - Porta **80 (HTTP)** aberta.
- GitHub com um repositório (ex.: `azure-webpage-17-11-25`).
- Acesso ao PowerShell no Windows.

---

## 2. Gerar par de chaves SSH no PC

No PowerShell, criar a pasta `.ssh` (se ainda não existir):

powershell
mkdir C:\Users\Cesae\.ssh 2>$null
Gerar a chave ED25519 para o deploy:
ssh-keygen -t ed25519 -C "github-deploy"

3. Autorizar a chave pública na VM (usuário gestor)

Importante: aqui assume-se que neste momento ainda tens algum meio de acesso à VM
(por exemplo, a chave original criada no Azure).

Conecta à VM (ajusta para a chave que tens agora):
ssh -i C:\Users\Cesae\Downloads\gestor.pem gestor@SEU_IP_PUBLICO

4. Criar Environment e Secret no GitHub

No repositório:

Vai em Settings → Environments.

Clica em New environment.

Nomeia como: TESTE (é o que o workflow usa).

Entra no Environment TESTE.

Em Environment secrets, clica em Add environment secret:

Name: DEPLOY_KEY

Value: cola (Ctrl+V) o texto em Base64 gerado anteriormente.

Clica em Add secret (ou Update secret, se já existir).

O conteúdo do secret não volta a ser exibido depois de salvo — isso é normal.

6. Criar o workflow GitHub Actions (blank.yml)

Cria a pasta e o ficheiro:

Diretório: .github/workflows/blank.yml

Conteúdo:

name: CI/CD Website on VM do Azure

on:
  push:
    branches: [ "main" ]
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: TESTE   # usa os secrets do Environment TESTE

    steps:
      - name: Checkout do código
        uses: actions/checkout@v4

      - name: Configurar chave SSH (decodificar Base64)
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.DEPLOY_KEY }}" | base64 -d > ~/.ssh/deploy_key
          chmod 600 ~/.ssh/deploy_key
          ssh-keyscan -H SEU_IP_PUBLICO >> ~/.ssh/known_hosts

      - name: Instalar Apache e publicar site
        run: |
          ssh -i ~/.ssh/deploy_key -o StrictHostKeyChecking=no gestor@SEU_IP_PUBLICO "sudo apt update && sudo apt install -y apache2"
          ssh -i ~/.ssh/deploy_key -o StrictHostKeyChecking=no gestor@SEU_IP_PUBLICO "sudo rm -rf /var/www/html/*"
          ssh -i ~/.ssh/deploy_key -o StrictHostKeyChecking=no gestor@SEU_IP_PUBLICO "sudo cp -r $GITHUB_WORKSPACE/* /var/www/html/"
          ssh -i ~/.ssh/deploy_key -o StrictHostKeyChecking=no gestor@SEU_IP_PUBLICO "sudo systemctl restart apache2"

      - name: Finalizar
        run: echo "Deploy concluído com sucesso (se todos os passos acima estiverem verdes)."
