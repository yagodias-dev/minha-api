# ⚙️ minha-api — FastAPI com CI/CD

![Python](https://img.shields.io/badge/Python_3.11-1a0b2e?style=flat-square&logo=python&logoColor=E040FB)
![FastAPI](https://img.shields.io/badge/FastAPI-1a0b2e?style=flat-square&logo=fastapi&logoColor=E040FB)
![Docker](https://img.shields.io/badge/Docker-1a0b2e?style=flat-square&logo=docker&logoColor=E040FB)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-1a0b2e?style=flat-square&logo=githubactions&logoColor=E040FB)
![AWS EC2](https://img.shields.io/badge/AWS_EC2-1a0b2e?style=flat-square)

🇧🇷 API simples em FastAPI, feita para praticar um **pipeline completo de CI/CD**: cada push roda os testes, gera uma imagem Docker, publica no Docker Hub e faz o deploy em uma instância **AWS EC2**.

🇺🇸 *A small FastAPI service built to practice a **full CI/CD pipeline**: every push runs the tests, builds a Docker image, pushes it to Docker Hub and deploys it to an **AWS EC2** instance.*

## O que é

| Rota | Método | O que faz |
| --- | --- | --- |
| `/` | GET | Retorna o status da API |
| `/health` | GET | Health check, usado pelo deploy para confirmar que subiu |
| `/items` | POST | Recebe um item (`name`, `price`) validado com Pydantic e devolve o que foi criado |
| `/docs` | GET | Documentação interativa gerada pelo FastAPI (Swagger) |

### Pipeline (`.github/workflows/python-app.yml`)

```text
push / pull request
   ↓
Build ── instala dependências e roda os testes (pytest)
   ↓   (só em push)
Imagem ── build da imagem Docker e push para o Docker Hub (tag da branch + tag do commit)
   ↓
├─ develop → Deploy Homologação (EC2 via SSH)
└─ main    → Deploy Produção   (EC2 via SSH)
              └─ troca o container e confere GET /health
```

Os deploys usam **environments** do GitHub (`homologacao` e `producao`) e todas as credenciais ficam em **secrets**, nunca no código.

### Docker

O `Dockerfile` é **multi-stage**: o primeiro estágio instala as dependências e o segundo copia só o necessário, rodando com um **usuário sem privilégios** (`appuser`). A API escuta na porta `8080`.

## Tecnologias

- Python 3.11, FastAPI, Pydantic e Uvicorn
- pytest e httpx (testes com `TestClient`)
- Docker (build multi-stage)
- GitHub Actions, Docker Hub e AWS EC2

## Como rodar

### Localmente

```bash
python -m venv venv
source venv/bin/activate        # no Windows: venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8080
```

Acesse `http://localhost:8080/docs`.

### Testes

```bash
PYTHONPATH=. pytest tests/ -v
```

### Com Docker

```bash
docker build -t minha-api .
docker run -d -p 8080:8080 --name minha-api minha-api
curl http://localhost:8080/health
```

### Configurar o pipeline no seu fork

Cadastre em **Settings → Secrets and variables → Actions**:

| Secret | Para que serve |
| --- | --- |
| `DOCKERHUB_USERNAME` | Usuário do Docker Hub |
| `DOCKERHUB_TOKEN` | Token de acesso do Docker Hub (não a senha) |
| `EC2_HOST` | Endereço da instância EC2 |
| `EC2_USERNAME` | Usuário SSH da instância |
| `EC2_SSH_KEY` | Chave privada SSH |

E crie os environments `homologacao` e `producao` em **Settings → Environments**.
