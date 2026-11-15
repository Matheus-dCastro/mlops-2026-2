# Guia Prático: Obtenção de Credenciais e Execução dos Labs 3 e 4

Este documento reúne o passo a passo completo e detalhado para você obter suas próprias credenciais (W&B, Colab e MongoDB Atlas), cadastrá-las no GitHub Actions e concluir a esteira de CD no Azure Container Apps.

---

## 1. Obtenção das Suas Credenciais Pessoais

As instruções desta seção correspondem à **Parte 2 do PDF do Laboratório 3 (Páginas 2 a 4)**.

### 1.1 Chave da API do Weights & Biases (`WANDB_API_KEY`)
* **Referência:** Lab 3, Página 2 — Seção 2.1.
* **Passos:**
  1. Acesse: [https://wandb.ai](https://wandb.ai) e faça login (pode usar login com GitHub ou Google).
  2. Acesse diretamente o link de autorização: [https://wandb.ai/authorize](https://wandb.ai/authorize).
  3. Copie a chave exibida (ela começa com `wandb_v1_...`).
  4. Guarde essa chave como sua `WANDB_API_KEY`.

---

### 1.2 Treinamento do Modelo no Google Colab (`WANDB_MODELS`)
* **Referência:** Lab 3, Páginas 2 e 3 — Seção 2.2.
* **Passos:**
  1. Acesse o [Google Colab](https://colab.research.google.com) e abra o notebook `IntentClassifier` fornecido pelo professor.
  2. Clique em **Arquivo > Salvar uma cópia no Drive**.
  3. Na barra lateral esquerda do Colab, clique no ícone de **Chave 🔑 (Secrets)**:
     - Clique em **Add new secret**.
     - Nome: `WANDB_KEY`
     - Valor: Cole a chave copiada no passo 1.1 (`wandb_v1_...`).
     - Ative a chave de alternância **Notebook access** (acesso ao notebook).
  4. Na barra lateral esquerda do Colab, clique no ícone de **Pastinha 📁 (Arquivos)**:
     - Faça o upload do arquivo `confusion_examples.yml` (disponível em `Unidade 2/API/confusion_examples.yml`).
  5. No menu superior, clique em **Ambiente de execução > Executar tudo** (aguarde o treinamento finalizar).
  6. **Localizar o Identificador do Modelo:**
     - Volte ao [https://wandb.ai](https://wandb.ai).
     - Abra o projeto **`mlops-2026-2`**.
     - No menu lateral esquerdo, clique na aba **Artifacts**.
     - Clique no artefato **`confusion-clf`** e copie o identificador completo que aparece no topo da página.
     - Formato: `seu_usuario/mlops-2026-2/confusion-clf:v0`.
     - Guarde esse valor como sua `WANDB_MODELS`.

---

### 1.3 Banco Gratuito no MongoDB Atlas (`MONGO_URL`)
* **Referência:** Lab 3, Páginas 3 e 4 — Seção 2.3.
* **Passos:**
  1. Acesse [https://www.mongodb.com/cloud/atlas](https://www.mongodb.com/cloud/atlas) e crie uma conta gratuita.
  2. Crie um cluster gratuito: selecione a camada **M0 Free** (provedor AWS, região `us-east-1` ou `sa-east-1`).
  3. **Criar Usuário do Banco:**
     - No menu lateral esquerdo, vá em **Security > Database Access**.
     - Clique em **Add New Database User**.
     - Método de autenticação: **Password**.
     - Defina usuário (ex.: `matheus`) e senha **sem caracteres especiais** (ex.: `SenhaMlops2026`).
     - Permissão: **Read and write to any database**.
     - Clique em **Add User**.
  4. **Liberar Acesso de Rede (IP):**
     - No menu lateral esquerdo, vá em **Security > Network Access**.
     - Clique em **Add IP Address**.
     - Clique no botão **Allow Access from Anywhere** (preencherá `0.0.0.0/0`).
     - Clique em **Confirm**.
  5. **Obter a URL de Conexão:**
     - No menu lateral esquerdo, vá em **Deployment > Database**.
     - No card do seu cluster, clique no botão **Connect**.
     - Selecione **Drivers** (Python).
     - Copie a connection string exibida. Exemplo:
       ```text
       mongodb+srv://matheus:<password>@cluster0.xxxxx.mongodb.net/?retryWrites=true&w=majority
       ```
     - Substitua `<password>` pela senha cadastrada no passo 3.
     - Guarde esse valor como sua `MONGO_URL`.

---

## 2. Cadastro dos Segredos no GitHub Actions

* **Referência:** Lab 4, Página 3.
* **Onde cadastrar:** Acesse no seu repositório:  
  `https://github.com/Matheus-dCastro/mlops-2026-2/settings/secrets/actions`

Cadastre os 4 segredos clicando em **New repository secret**:

| Nome do Secret | Valor |
| :--- | :--- |
| `WANDB_API_KEY` | Chave copiada no passo 1.1 (`wandb_v1_...`) |
| `WANDB_PROJECT` | `mlops-2026-2` |
| `WANDB_MODELS` | Caminho do artefato copiado no passo 1.2 (`seu_usuario/mlops-2026-2/confusion-clf:v0`) |
| `MONGO_URL` | String de conexão MongoDB copiada no passo 1.3 |

> Após cadastrar os 4 segredos, vá na aba **Actions** do seu repositório, abra a execução que falhou e clique em **Re-run jobs > Re-run all jobs**. Aguarde os jobs `test` e `smoke` ficarem **verdes (✔️)**.

---

## 3. Publicar Imagem Base no GHCR

* **Referência:** Lab 4, Páginas 3 e 4 (Parte 2).

1. **Gerar Token Clássico no GitHub:**
   - Acesse: [https://github.com/settings/tokens/new](https://github.com/settings/tokens/new)
   - Nome: `GHCR Deploy`
   - Escopo: selecione a caixa **`write:packages`**
   - Clique em **Generate token** e copie o token gerado.
2. **Executar no terminal local:**
   ```bash
   cd "/home/alomy/Projetos/UFRN_06/MLOPS/mlops-2026-2"

   # Login no GHCR (substitua SEU_TOKEN pelo token gerado):
   echo SEU_TOKEN | docker login ghcr.io -u Matheus-dCastro --password-stdin

   # Build e envio da imagem inicial:
   docker build --platform linux/amd64 -t ghcr.io/matheus-dcastro/intent-clf:latest .
   docker push ghcr.io/matheus-dcastro/intent-clf:latest
   ```
3. **Tornar o Pacote Público:**
   - Acesse: `https://github.com/Matheus-dCastro?tab=packages`
   - Clique no pacote **`intent-clf`** > **Package settings**.
   - Na seção final (Danger Zone), altere a visibilidade para **Public**.
   - Em **Manage Actions access**, clique em **Add repository**, selecione `mlops-2026-2` e atribua permissão **Write**.

---

## 4. Provisionamento no Azure Container Apps

* **Referência:** Lab 4, Páginas 5 a 7.

Execute os comandos no terminal:

```bash
# 1. Login na conta institucional da UFRN (Azure for Students):
az login

# 2. Criar Grupo de Recursos e Ambiente:
az group create --name mlops-2026-2 --location eastus
az containerapp env create --name mlops-env --resource-group mlops-2026-2 --location eastus

# 3. Criar Identidade Gerenciada e Permissões:
az identity create --name mlops-deployer --resource-group mlops-2026-2

CLIENT_ID=$(az identity show --name mlops-deployer --resource-group mlops-2026-2 --query clientId -o tsv)
PRINCIPAL_ID=$(az identity show --name mlops-deployer --resource-group mlops-2026-2 --query principalId -o tsv)
SUB_ID=$(az account show --query id -o tsv)
TENANT_ID=$(az account show --query tenantId -o tsv)

az role assignment create \
  --assignee $PRINCIPAL_ID \
  --role Contributor \
  --scope "/subscriptions/$SUB_ID/resourceGroups/mlops-2026-2"

# 4. Criar o Container App (substitua as credenciais pelas suas):
az containerapp create \
  --name intent-clf \
  --resource-group mlops-2026-2 \
  --environment mlops-env \
  --image ghcr.io/matheus-dcastro/intent-clf:latest \
  --target-port 8000 \
  --ingress external \
  --cpu 1.0 --memory 2.0Gi \
  --secrets wandb-key="SUA_WANDB_API_KEY" mongo-url="SUA_MONGO_URL" \
  --env-vars \
    WANDB_API_KEY=secretref:wandb-key \
    MONGO_URL=secretref:mongo-url \
    WANDB_PROJECT=mlops-2026-2 \
    WANDB_MODELS="SEU_USUARIO_WANDB/mlops-2026-2/confusion-clf:v0" \
    APP_VERSION="manual-v1.0.0"
```

> Ao final do comando, copie a **`Application Url`** exibida (exemplo: `https://intent-clf.xxxx.eastus.azurecontainerapps.io`).

---

## 5. Configurar OIDC e Environment de Produção

* **Referência:** Lab 4, Páginas 7 e 8.

1. **Criar Credencial Federada no Azure:**
   ```bash
   az identity federated-credential create \
     --name github-actions-prod \
     --identity-name mlops-deployer \
     --resource-group mlops-2026-2 \
     --issuer "https://token.actions.githubusercontent.com" \
     --subject "repo:Matheus-dCastro/mlops-2026-2:environment:production" \
     --audiences "api://AzureADTokenExchange"
   ```
2. **Consultar os IDs para cadastrar no GitHub:**
   ```bash
   echo "AZURE_CLIENT_ID: $CLIENT_ID"
   echo "AZURE_TENANT_ID: $TENANT_ID"
   echo "AZURE_SUBSCRIPTION_ID: $SUB_ID"
   ```
3. **Cadastrar Secrets do Azure no GitHub:**
   - Em **Settings > Secrets and variables > Actions > Secrets**, adicione:
     - `AZURE_CLIENT_ID`
     - `AZURE_TENANT_ID`
     - `AZURE_SUBSCRIPTION_ID`
4. **Cadastrar Variável `APP_URL` no GitHub:**
   - Em **Settings > Secrets and variables > Actions > Variables**, crie a variável:
     - `APP_URL`: URL completa da aplicação gerada pelo Azure (com `https://`).
5. **Configurar o Environment `production`:**
   - Em **Settings > Environments > New environment**, crie o ambiente com o nome exato: `production`.
   - Marque a opção **Required reviewers** e adicione o seu usuário (`Matheus-dCastro`).
   - Salve as alterações.

---

## 6. Publicação de Versões, Aprovação e Rollback (Evidências de Entrega)

* **Referência:** Lab 4, Páginas 14 e 15.

### 6.1 Publicar versão `v1.0.1`
```bash
git tag v1.0.1
git push origin v1.0.1
```
* Acesse a aba **Actions** do GitHub e abra a execução da tag `v1.0.1`.
* No job `deploy`, clique em **Review deployments**, selecione o ambiente `production` e aprove.
* 📸 **Print 1:** Captura do GitHub Actions mostrando a solicitação de aprovação / deploy verde.
* Abra no navegador a sua `APP_URL`.
* 📸 **Print 2:** Captura da tela do navegador com a resposta:  
  `{"status":"ok","version":"v1.0.1"}`

### 6.2 Publicar versão `v1.0.2`
```bash
git tag v1.0.2
git push origin v1.0.2
```
* Aprove o deploy no GitHub Actions.
* 📸 **Print 3:** Captura da tela do navegador com a resposta:  
  `{"status":"ok","version":"v1.0.2"}`

### 6.3 Teste de Rollback para `v1.0.1`
1. Na aba **Actions** do GitHub, clique no workflow **CD** no menu lateral esquerdo.
2. Clique no botão **Run workflow** à direita.
3. No campo da tag, digite: `v1.0.1` e clique em **Run workflow**.
4. Aprove a execução de deploy no ambiente `production`.
5. 📸 **Print 4:** Captura do GitHub Actions mostrando o fluxo de rollback (jobs CI e Release pulados, Deploy e Verify executados com sucesso).
6. 📸 **Print 5:** Captura do navegador mostrando o retorno da aplicação para `v1.0.1`.

---

## 7. Desprovisionamento dos Recursos no Azure

Assim que coletar todas as evidências (prints), apague o grupo de recursos para evitar o consumo de créditos educacionais da conta:

```bash
az group delete --name mlops-2026-2 --yes --no-wait
```
