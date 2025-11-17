# CI/CD Website on Azure VM (GitHub Actions)

Este projeto faz o deploy automático de um site estático para uma máquina virtual Ubuntu no Azure, usando GitHub Actions e SSH.

---

## 🚀 Funcionamento

1. O conteúdo deste repositório (ex.: `index.html`) é enviado automaticamente para a VM.
2. Sempre que ocorre um commit na branch **main**, o GitHub Actions:
   - Decodifica a chave privada salva no secret `DEPLOY_KEY`;
   - Conecta na VM via SSH (usuário `gestor`);
   - Instala/atualiza o Apache;
   - Limpa o diretório `/var/www/html`;
   - Copia os ficheiros do repositório para o servidor;
   - Reinicia o Apache.

A página fica disponível em: http://<IP-PUBLICO-DA-VM>


### 2. No GitHub
- Criar Environment: **TESTE**
- Criar secret dentro do Environment:
- **DEPLOY_KEY** → chave privada em Base64

### 3. Workflow
O workflow está em:

