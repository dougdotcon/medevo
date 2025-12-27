# 🚀 Gerenciador de Estado de Conversa para N8N

## 📋 Problema Resolvido

Este fluxo de trabalho resolve o desafio fundamental de gerenciar conversas não lineares em chatbots: manter o contexto ao longo de várias etapas. Ele garante que o bot aguarde entradas específicas do usuário, gerencia o estado da conversa para prevenir reativações incorretas e sabe exatamente onde o usuário está no diálogo.

## 🔧 Como Funciona

### 1. **Estado Persistente**
- Utiliza o Google Sheets como banco de dados para armazenar o estado da conversa de cada usuário.
- Cada usuário possui um registro único contendo: `from` (ID), `step`, `data` (JSON), `timestamp` e `status`.

### 2. **Lógica Inteligente do Fluxo**

Webhook → Processador de Mensagem → Buscador de Estado -> Nó de Decisão de Ação -> Atualizador de Estado


### 3. **Estágios do Estado**
- `null` ou `vazio`: Usuário novo - envia lista de boas-vindas.
- `awaiting_specialty`: Usuário aguardando escolher a especialidade.
- `awaiting_date`: Usuário escolheu especialidade, aguardando a data.
- `awaiting_time`: Usuário informou a data, aguardando o horário.
- `finished`: Conversa concluída.

## 📊 Estrutura da Planilha Google

Crie uma planilha com a aba "States" e as seguintes colunas:

| A (from) | B (step) | C (data) | D (timestamp) | E (status) |
|----------|----------|----------|---------------|------------|
| 5511999999999@c.us | awaiting_specialty | {} | 2025-01-17T10:30:00Z | active |

## ⚙️ Configuração

### 1. **Variáveis de Ambiente do N8N**
Adicione as seguintes variáveis na sua instância do N8N:

GOOGLE_SHEET_ID = "seu_id_da_planilha_aqui"


### 2. **Webhook da Evolution API**
Configure o webhook na Evolution API para apontar para:

https://sua-instancia-n8n.com/webhook/whatsapp-webhook


### 3. **Credenciais do Google Sheets**
- Configure a conexão com o Google Sheets no N8N.
- Garanta que a conta de serviço tenha permissões de leitura/escrita na planilha.

## 🎯 Melhorias Principais

### ✅ **Controle de Estado Robusto**
- Cada usuário mantém um estado independente.
- Sem conflitos entre conversas simultâneas.
- O estado persiste mesmo se a instância do N8N for reiniciada.

### ✅ **Validação de Entrada**
- Verifica se a mensagem é uma resposta válida (seleção de lista, etc.).
- Lida com mensagens de texto padrão com elegância.
- Valida opções selecionadas contra entradas esperadas.

### ✅ **Suporte a Fluxo Não Linear**
- Usuários podem retomar conversas de onde pararam.
- Não reinicia o fluxo do zero se já estiver em progresso.
- Lida eficazmente com erros e opções inválidas.

### ✅ **Escalabilidade**
- Suporta múltiplos usuários interagindo simultaneamente.
- Fácil de estender com novas etapas e lógica.
- Estrutura modular para fácil expansão.

## 🔄 Guia de Expansão

### Adicionando uma Nova Etapa:
1. **Defina o novo estado** (ex: `awaiting_time`).
2. **Crie um novo nó de verificação** no fluxo para detectar esse estado.
3. **Adicione lógica de processamento** para essa etapa específica.
4. **Atualize o estado** para a próxima etapa após o processamento.

### Exemplo - Adicionando Etapa de Seleção de Horário:
javascript
// Dentro do nó de verificação de estado
if (userState.step === 'awaiting_date') {
  // Processa a data informada
  // Envia lista de horários disponíveis
  // Atualiza estado para 'awaiting_time'
}


## 🚨 Notas Importantes

### **Por que esta arquitetura funciona?**
1. **Webhook Único**: Todas as mensagens entram por um único ponto de entrada, simplificando a configuração da API.
2. **Estado Centralizado**: O Google Sheets atua como a fonte da verdade, permitindo que o bot mantenha o contexto sem armazenamento de memória complexo.
3. **Ramificação Condicional**: O fluxo usa lógica condicional simples para rotear usuários com base no estado armazenado, garantindo uma resposta personalizada sempre.

### **Pré-requisitos:**
- Instância do N8N (Cloud ou Self-hosted).
- Projeto no Google Cloud com API do Sheets habilitada.
- Instância da Evolution API em execução e acessível.

---
*Este projeto foi projetado para ser um ponto de partida para fluxos conversacionais complexos que exigem persistência de estado.*