# Claude Code + n8n via MCP

Guia prático para usar o **Claude Code** (agente de terminal) para criar, ler, validar e rodar workflows do **n8n** usando linguagem natural, através do servidor **n8n-MCP**.

Setup de referência: n8n em **Docker no Windows**, container `n8n_iagen`, volume `n8n_data`, acessível em `http://localhost:5678`.

---

## Conceito: os dois "MCP" do n8n

Antes de tudo, evite a confusão mais comum. Existem duas coisas com o nome "MCP" no ecossistema n8n, e elas apontam em **direções opostas**:

| | O que faz | Quem é o cliente |
|---|---|---|
| **n8n-MCP** (este guia) | Dá ao Claude Code as ferramentas para **construir e gerenciar** seus workflows | Claude Code |
| **MCP Server Trigger / MCP Client** (nodes nativos) | Seus *workflows* expõem ou consomem ferramentas MCP | Outros agentes de IA |

Aqui usamos o **n8n-MCP**: o Claude Code é o cliente, e o n8n é controlado por ele.

---

## Como funciona

```
Claude Code  →  servidor n8n-MCP  →  API REST do n8n  →  seu n8n (localhost:5678)
```

Você descreve o que quer em português. O Claude Code descobre os nodes corretos, monta o JSON do workflow, valida e — se você autorizar com a API — cria/atualiza direto na sua instância. Isso elimina o problema de a IA "inventar" nodes que não existem ou parâmetros renomeados, porque ela consulta o catálogo real de nodes.

---

## Pré-requisitos

- **Claude Code** instalado e funcionando no terminal.
- **Node.js** (o `npx` vem junto) — necessário para rodar o servidor.
- **n8n rodando** em `http://localhost:5678`.
- (Opcional, mas recomendado) uma **API Key do n8n** para o modo leitura/escrita.

---

## Passo 1 — Gerar a API Key do n8n

Só é necessária se você quiser que o Claude Code **leia e escreva** na instância (modo completo). Para apenas consultar documentação de nodes, pule para o Passo 2.

No n8n (`http://localhost:5678`):

```
Settings → n8n API → Create an API key
```

Dê um nome (ex.: `claude-code`), copie a chave e guarde. Ela só é exibida uma vez.

> Como o n8n roda em Docker, a URL da API vista pelo Claude Code (que roda no host Windows) é `http://localhost:5678`, pois a porta do container `n8n_iagen` está mapeada para o host.

---

## Passo 2 — Registrar o n8n-MCP no Claude Code (modo documentação)

Comece **sem credenciais**. Nesse modo o Claude só consulta o catálogo de nodes — é seguro e não toca na sua instância.

**Linux / macOS / WSL / Git Bash:**

```bash
claude mcp add n8n-mcp \
  -e MCP_MODE=stdio \
  -e LOG_LEVEL=error \
  -e DISABLE_CONSOLE_OUTPUT=true \
  -- npx n8n-mcp
```

**Windows PowerShell** (a continuação de linha é a crase `` ` ``, não a barra invertida):

```powershell
claude mcp add n8n-mcp `
  '-e MCP_MODE=stdio' `
  '-e LOG_LEVEL=error' `
  '-e DISABLE_CONSOLE_OUTPUT=true' `
  -- npx n8n-mcp
```

**Versão em uma linha só** (funciona em qualquer shell, útil se a continuação de linha der problema):

```bash
claude mcp add n8n-mcp -e MCP_MODE=stdio -e LOG_LEVEL=error -e DISABLE_CONSOLE_OUTPUT=true -- npx n8n-mcp
```

---

## Passo 3 — Ativar leitura/escrita (modo completo)

Quando confiar no fluxo, adicione a URL e a API Key do n8n. Agora o Claude Code pode criar, atualizar e executar workflows na sua instância.

**Linux / macOS / WSL / Git Bash:**

```bash
claude mcp add n8n-mcp \
  -e MCP_MODE=stdio \
  -e LOG_LEVEL=error \
  -e DISABLE_CONSOLE_OUTPUT=true \
  -e N8N_API_URL=http://localhost:5678 \
  -e N8N_API_KEY=sua-api-key-aqui \
  -- npx n8n-mcp
```

**Windows PowerShell:**

```powershell
claude mcp add n8n-mcp `
  '-e MCP_MODE=stdio' `
  '-e LOG_LEVEL=error' `
  '-e DISABLE_CONSOLE_OUTPUT=true' `
  '-e N8N_API_URL=http://localhost:5678' `
  '-e N8N_API_KEY=sua-api-key-aqui' `
  -- npx n8n-mcp
```

> Se você já tinha registrado o `n8n-mcp` no modo documentação, remova antes de recriar:
> ```bash
> claude mcp remove n8n-mcp
> ```

---

## Passo 4 — Verificar a conexão

Reinicie o Claude Code e rode:

```
/mcp
```

Se deu certo, aparece o servidor `n8n-mcp` e a lista de ferramentas, incluindo `search_nodes`, `get_node`, `validate_node` e `search_templates`.

---

## Como usar

Basta descrever o que você quer, em português, dentro do Claude Code. Exemplos:

```
Crie um workflow no n8n que, ao receber uma mensagem no Telegram,
consulte uma linha no Google Sheets e responda com o resultado.
```

```
Abra meu workflow "atendimento-bot" e adicione um node de
condição (IF) que verifica se o campo status é "aberto".
```

```
Valide o workflow atual e me diga se há algum node com
parâmetro obrigatório faltando.
```

O Claude descobre os nodes, monta a configuração, valida e (se autorizado) aplica na instância. Você mantém o controle da lógica de negócio e da decisão de publicar.

---

## Ferramentas principais expostas pelo n8n-MCP

| Ferramenta | Função |
|---|---|
| `search_nodes` | Busca nodes no catálogo real do n8n |
| `get_node` | Retorna os parâmetros reais de um node |
| `validate_node` | Valida a configuração antes de importar |
| `search_templates` | Busca templates de workflow prontos |

(No modo completo, somam-se ferramentas de criar/atualizar/executar workflows via API.)

---

## Segurança — leia antes de produção

- **Nunca deixe a IA editar workflows de produção diretamente.** Teste numa instância separada primeiro.
- Use uma **API key com escopo restrito** e comece numa instância não-produtiva.
- Comece em **modo documentação** (sem credenciais) enquanto aprende o fluxo; só adicione a API key quando confiar.
- O recurso de build via MCP nativo do n8n ainda está em **Public Preview** — revise sempre o resultado.

---

## Alternativa: MCP nativo do n8n

Além do projeto community, o próprio n8n passou a oferecer um servidor MCP nativo que também constrói e atualiza workflows a partir de um prompt. A conexão em clientes Claude segue o padrão:

```
Settings → Connectors → Add custom connector → informar a URL do servidor n8n → autorizar
```

O `czlonkowski/n8n-mcp` (deste guia) continua sendo a opção mais completa e madura para uso via Claude Code.

---

## Troubleshooting

- **`/mcp` não mostra o servidor:** confirme que o Node.js/`npx` está no PATH e reinicie o Claude Code.
- **Falha de conexão com a API:** teste a instância com uma chamada simples e confirme que `N8N_API_URL` aponta para `http://localhost:5678` (e que o container `n8n_iagen` está de pé).
- **A maioria das falhas do MCP n8n** vem de acesso (API key/URL), modo de rede ou workflow mal adaptado — raramente é falha "aleatória".
