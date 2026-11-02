# 🤖 Analisador de Vagas com IA (n8n + Gemini)

Fluxo do [n8n](https://n8n.io) que busca vagas de tecnologia todos os dias, usa IA para avaliar a compatibilidade de cada vaga com o seu perfil profissional e salva as melhores numa planilha do Google Sheets.

Projeto desenvolvido durante o desafio **#7DaysOfCode** de n8n da [Alura](https://www.alura.com.br).

## Como funciona

```
Agendamento diário (9h)
  → Configuração (seu perfil, nota mínima e palavras-chave)
  → Buscar vagas (API pública)
  → Filtro Brasil/Remoto
  → Limitar vagas
  → Preparar dados da vaga
  → Análise de compatibilidade (Gemini)
  → Extrair resultado (nota + resumo)
  → Vaga aderente? (nota mínima + palavras-chave)
  → Salvar na planilha (Google Sheets)
```

1. **Busca** as vagas na API pública [apibr.com/vagas](https://apibr.com/vagas/api/v2/vagas).
2. **Filtra** só as vagas remotas ou no Brasil.
3. **Envia cada vaga para o Gemini**, que compara com o seu perfil e devolve uma nota de 0 a 100 e um resumo com pontos fortes e lacunas.
4. **Seleciona** as vagas com nota acima do mínimo definido e que contenham pelo menos uma das suas palavras-chave.
5. **Salva** na planilha, atualizando a linha se a vaga já existir (usa o link como chave, então não duplica).

## Requisitos

- n8n (self-hosted ou cloud)
- Chave de API do Google Gemini ([Google AI Studio](https://aistudio.google.com/apikey))
- Conta Google com acesso ao Google Sheets

## Instalação

### 1. Importe o fluxo

No n8n, crie um workflow novo, abra o menu `⋯` → **Import from File** e selecione o `workflow.json`.

### 2. Configure seu perfil

Abra o nó **Configuração** e edite os três campos:

| Campo | O que colocar |
|---|---|
| `perfil_candidato` | Resumo do seu perfil: cargo, senioridade, anos de experiência, tecnologias, experiências e formação. Quanto mais detalhado, melhor a análise. |
| `nota_minima` | Nota mínima (0 a 100) para a vaga ser salva. Padrão: `70`. |
| `palavras_chave` | Termos da sua área, separados por vírgula. A vaga precisa ter pelo menos um deles. Maiúsculas e acentos são ignorados. |

### 3. Credencial do Gemini

No nó **Google Gemini Chat Model**, crie uma credencial **Google Gemini (PaLM) API** com a sua chave. No campo **Model**, prefira selecionar da lista, pois o n8n mostra apenas os modelos disponíveis para a sua conta.

### 4. Credencial do Google Sheets

O n8n self-hosted precisa de um OAuth Client próprio:

1. No [Google Cloud Console](https://console.cloud.google.com), crie um projeto e ative a **Google Sheets API** e a **Google Drive API**.
2. Configure a **Tela de consentimento OAuth** (tipo Externo) e clique em **Publicar app**. No modo Teste, o login só funciona para usuários de teste e expira a cada 7 dias.
3. Em **Credenciais**, crie um **ID do cliente OAuth** do tipo **Aplicativo da Web**, com a URL de redirecionamento que o n8n mostra na tela da credencial (ex: `http://localhost:5678/rest/oauth2-credential/callback`).
4. Cole o Client ID e o Client Secret no n8n e clique em **Sign in with Google**.

Alternativa: use uma **Service Account** e compartilhe a planilha com o e-mail dela.

### 5. Prepare a planilha

Crie uma planilha com estes cabeçalhos na primeira linha:

```
nota | resumo | titulo | link | requisitos | data_analise
```

Depois selecione a planilha e a aba no nó **Salvar na planilha**.

## Limites da API gratuita do Gemini

O plano gratuito tem limites por minuto (RPM) e por dia (RPD), que variam por modelo. Consulte os seus em [ai.dev/rate-limit](https://ai.dev/rate-limit).

- O nó de IA processa **uma vaga a cada 6 segundos** (cerca de 10 por minuto) para não estourar o limite por minuto. Se o seu modelo tiver RPM menor, aumente o **Delay Between Batches**.
- O nó **Limitar vagas** controla quantas vagas vão para a IA por execução. Mantenha abaixo do seu limite diário, com folga para testes.
- Em **Workflow Settings**, ative **Timeout Workflow** com tempo suficiente para o volume de vagas (cada vaga leva ~10 segundos).

## Ideias de evolução

- **Notificações:** enviar as vagas aprovadas por Telegram, e-mail, Slack, Discord ou WhatsApp.
- **Evitar reanálise:** adicionar um nó **Remove Duplicates** ("Remove Items Processed in Previous Executions", usando o link da vaga) logo após o filtro, para que a IA só analise vagas novas.
- **Filtrar antes da IA:** mover a checagem de palavras-chave para antes do Gemini e economizar cota.
- **Saída estruturada:** usar um **Structured Output Parser** no nó de IA em vez do `JSON.parse` manual.

## Licença

MIT
