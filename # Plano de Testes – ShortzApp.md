# Plano de Testes – ShortzApp

* **Versão**: 1.0
* **Data**: 25 de Março de 2026
* **Autor(es)**: Pedro Elizeu dos Santos Junior, Ana Beatriz Adriano da Silva
* **Projeto**: ShortzApp 

---

## 1. Introdução

Este documento detalha o planejamento estratégico das atividades de teste para a plataforma **ShortzApp**, um sistema de compartilhamento de vídeos curtos (até 60 segundos) que permite aos usuários visualizar, publicar e interagir com conteúdos por meio de curtidas, comentários, follows e playlists.

O objetivo principal deste plano é assegurar que as funcionalidades críticas — como cadastro/autenticação, upload/streaming de vídeos e interações sociais funcionem de acordo com os requisitos funcionais (RF) e regras de negócio (RN) definidos na especificação do software. Através de uma abordagem sistemática, buscamos mitigar riscos de segurança, usabilidade, desempenho e integridade de dados.

---

## 2. Escopo dos Testes

### 2.1 Em Escopo

As seguintes funcionalidades serão foco neste ciclo de testes:

* **Gestão de Usuários e Autenticação**
    * Cadastro de novos usuários (validação de e-mail, nome de usuário único, senha mínima de 8 caracteres e confirmação de senha).
    * Login por e-mail ou nome de usuário.
    * Logout e controle de sessão.
    * Upload/atualização de foto de perfil.
    * Edição de perfil (nome completo, biografia com limite e foto).

* **Gestão de Vídeos**
    * Upload de vídeo com validação de formato (MP4/WebM), duração máxima (60s) e título obrigatório com no máximo 100 caracteres.
    * Upload de capa com validações de formato, proporção e tamanho da imagem.
    * Atualização de contagem de visualizações.

* **Feed e Descoberta**
    * Exibição de feed com prioridades ao usuários que está seguindo (Para usuários autenticados).
    * Feed global de mais assistidos quando não estiver autenticado
    * Ordenação de resultados (relevância, mais visualizados, mais recentes).

* **Interações Sociais**
    * Curtir/descurtir vídeos e atualização de contador.
    * Comentar em vídeos (Usuário autenticados)
    * Seguir/deixar de seguir usuários e atualização de contadores de seguidores.

* **Playlists**
    * Criação de playlist.
    * Adição e remoção de vídeos em playlists.
    * Visualização de playlists do usuário.

* **Notificações**
    * Notificações para novo seguidor, curtida, comentário e novo vídeo de perfil seguido.
    * Marcar notificações como lidas.

* **Administração e Moderação**
    * Visualização e gestão de usuários, vídeos e denúncias.
    * Ações administrativas (Remoção de vídeo, bloqueio/suspensão de usuário).

### 2.2 Fora de Escopo (neste ciclo)

As seguintes funcionalidades **não** serão abordadas neste ciclo inicial:

* Testes de compatibilidade em navegadores diferentes
* Testes de infraetrutura 
* Testes de app nativo (Android/iOS), caso não haja cliente nativo nesta etapa.

---

## 3. Estratégia de Testes

A estratégia adota princípios de **Shift-Left Testing**, incorporando validações desde o início do desenvolvimento para reduzir retrabalho e aumentar a qualidade.

A pirâmide de testes será aplicada da seguinte forma:

1. **Testes Unitários**
     Foco em validações de negócio e utilitários (ex.: regras de senha, validação de campos, validações de upload).

2. **Testes de Integração**
     Foco na comunicação entre camadas (banco de dados, APIs, autenticação e upload).

3. **Testes de Sistema / Black-Box (E2E funcional)**
     Foco nos fluxos críticos de usuário (cadastro, login, upload, assistir, interagir, buscar, seguir e playlists).

### 3.1 Técnicas de Projeto de Teste

* Particionamento de equivalência.
* Análise de valores-limite.
* Tabela de decisão para regras condicionais.
* Testes baseados em risco (priorização por impacto e probabilidade).

### 3.2 Ambientes

* Ambiente de desenvolvimento local (Node.js + Express + MySQL).
* Ambiente de homologação (quando disponível), com dados de teste controlados.

---

## 4. Riscos e Mitigação

| Risco | Probabilidade | Impacto | Prioridade | Estratégia de Mitigação (Testes) |
| :--- | :--- | :--- | :--- | :--- |
| Upload de arquivo malicioso ou acima dos limites (vídeo/imagem). | Média | Alto | Alta | Testes de valores-limite e formatos permitidos; validação server-side e middleware de upload. |
| Falha de autenticação/autorização (acesso indevido). | Baixa | Alto | Crítica | Testes de integração em login, sessão e middlewares de proteção de rota. |
| XSS em comentários, bios ou campos textuais. | Alta | Médio | Alta | Testes com entradas maliciosas, validação/sanitização e verificação de encoding na renderização. |
| Inconsistência em contadores (likes, views, seguidores, comentários). | Média | Alto | Alta | Testes de integração e concorrência básica em operações de interação. |
| Falha em HTTP Range Requests e streaming parcial. | Média | Alto | Alta | Testes de endpoint de vídeo para status 206, headers corretos e segmentos de mídia. |
| Busca com resultados incorretos (parcial/ordenação). | Média | Médio | Média | Testes funcionais por termo parcial e cenários de ordenação por critério. |
| Ações administrativas indevidas (sem perfil admin). | Baixa | Alto | Alta | Testes de autorização por papel e bloqueio de acesso a rotas administrativas. |
| Mensagens de erro genéricas/insuficientes para usuário. | Alta | Baixo | Média | Testes de UX funcional para feedback claro em falhas de validação e operação. |

---

## 5. Casos de Teste Planejados (Black-Box)

### 5.1 Particionamento de Equivalência e Análise de Valores-Limite

#### Funcionalidade: Cadastro de Usuário - Campo Senha
**Regra:** mínimo de 8 caracteres.

| Campo | Classe Válida | Classes Inválidas |
| :--- | :--- | :--- |
| Tamanho | >= 8 caracteres | < 8 caracteres |

**Valores de teste:**
* Valor abaixo: `Abc@123` (7)
* Valor limite: `Abc@1234` (8)
* Valor válido comum: `Shortz@2026`

> Observação: caso a equipe adote regra forte (maiúscula/minúscula/número/especial), atualizar os casos para refletir a política final.

---

#### Funcionalidade: Upload de Vídeo
**Regra:** duração máxima 60s, formatos suportados (ex.: MP4/WebM).

| Campo | Classe Válida | Classes Inválidas |
| :--- | :--- | :--- |
| Duração | 1s a 60s | 0s; > 60s |
| Formato | MP4, WebM | AVI, MOV, MKV (Tipo de arquivos não suportados) |
| Título | 1 a 100 caracteres (ou limite definido) | vazio; acima do limite |

**Valores de teste (duração):**
* 59s (válido)
* 60s (limite válido)
* 61s (inválido)

---

#### Funcionalidade: Upload de Thumbnail/Capa
**Regra:** imagem JPG/PNG, até 2MB, proporção 16:9 e limites de resolução (mín. 640x360, máx. 1280x720).

| Campo | Classe Válida | Classes Inválidas |
| :--- | :--- | :--- |
| Formato | JPG, PNG | GIF, BMP, PDF |
| Tamanho | > 0 e <= 2MB | 0 bytes; > 2MB |
| Resolução | 640x360 a 1280x720 | abaixo do mínimo; acima do máximo |
| Proporção | 16:9 | proporções diferentes |

---

### 5.2 Tabela de Decisão: Publicar Vídeo

| Condições | R1 | R2 | R3 | R4 | R5 |
| :--- | :---: | :---: | :---: | :---: | :---: |
| Usuário autenticado? | S | S | S | S | N |
| Título válido? | S | N | S | S | S |
| Vídeo válido (formato/duração)? | S | S | N | S | S |
| Capa válida? | S | S | S | N | S |
| **Ação: Publicar vídeo** | Sim | Não | Não | Não | Não |

---

### 5.3 Fluxos Críticos (E2E Funcional)

1. **Cadastro + Login + Logout**
2. **Upload de vídeo + exibição no feed/perfil**
3. **Assistir vídeo + contagem de views**
4. **Curtir/descurtir + atualização de contador**
5. **Comentar + exibição imediata do comentário**
6. **Seguir/deixar de seguir + atualização de contadores**
7. **Busca parcial por usuário/vídeo + ordenação**
8. **Criar playlist + adicionar/remover vídeo**
9. **Acesso admin + moderação de vídeo/usuário/denúncia**
10. **Proteção de rotas autenticadas e rotas admin**

---

## 6. Critérios de Aceitação

### 6.1 Critérios de Entrada

* Requisitos funcionais e regras de negócio aprovados.
* Ambiente de testes configurado e estável.
* Build implantada com sucesso em ambiente de teste.
* Base de dados de teste disponível.
* Ferramentas de execução e evidência (logs, prints, relatórios) disponíveis.

### 6.2 Critérios de Saída

* 100% dos testes de prioridade **Crítica** e **Alta** executados.
* Sem defeitos abertos de severidade **Crítica** e **Alta**.
* Cobertura mínima de requisitos priorizados: **90%**.
* Fluxos críticos de negócio aprovados (cadastro/login/upload/stream/interações).

### 6.3 Critérios de Suspensão

* Falha em funcionalidade bloqueadora (ex.: login, upload, streaming) sem workaround.
* Instabilidade de ambiente por mais de 4 horas.
* Taxa de falha acima de 20% em casos de prioridade Alta na primeira rodada sem correção incremental.

---

## 7. Priorização de Execução (por risco)

**Prioridade 1 (executar primeiro):**
* Autenticação/autorização.
* Upload e validações de vídeo/capa.
* Streaming de vídeo.
* Interações principais (like/comentário/follow).

**Prioridade 2:**
* Feed priorizado/global.
* Busca e ordenação.
* Playlists.

**Prioridade 3:**
* Notificações e funcionalidades administrativas complementares.

---
