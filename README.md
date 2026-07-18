# Integração TheHive + Cortex + MISP

Com base no vídeo explicativo e nas imagens de apoio fornecidas, elaborei uma documentação completa e passo a passo para configurar e integrar o TheHive, Cortex e MISP.

Esta documentação assume que você utilizará o Docker Compose para subir os serviços e usa como base o nosso projeto [DOCKER-THE_HIVE_5.2_community_CORTEX_MISP_LAB](https://github.com/nks1097/DOCKER-THE_HIVE_5.2_community_CORTEX_MISP_LAB).

> **Vídeo de Referência:** [INSERIR LINK DO VÍDEO AQUI]

---

## Pré-requisitos e Subida dos Contêineres (Docker)

Antes de acessar as interfaces gráficas, é necessário subir a infraestrutura completa (TheHive, Cortex e MISP).

### 1. Subindo TheHive e Cortex
1. Tenha o projeto base clonado e o arquivo `docker-compose.yml` configurado em sua máquina:
   ```bash
   git clone https://github.com/nks1097/DOCKER-THE_HIVE_5.2_community_CORTEX_MISP_LAB.git
   cd DOCKER-THE_HIVE_5.2_community_CORTEX_MISP_LAB
   ```
2. **Atenção (Dica do Vídeo):** Edite o arquivo `docker-compose.yml` e certifique-se de alterar as variáveis de HOSTNAME ou IPs de `10.0.x.x` para o endereço IP real da sua máquina local (exemplo baseado nas imagens: `192.168.0.107`).
3. No terminal, navegue até a pasta do arquivo e execute o comando para subir os contêineres em segundo plano:
   ```bash
   sudo docker-compose up -d
   ```

### 2. Subindo o MISP
Para o MISP, utilizaremos o repositório oficial do Docker do MISP. Em uma pasta separada, execute:
1. Clone o repositório e prepare o ambiente:
   ```bash
   git clone https://github.com/MISP/misp-docker.git
   cd misp-docker
   cp template.env .env
   ```
2. Edite o arquivo `.env` ajustando a variável `BASE_URL` para o IP da sua máquina:
   ```env
   ADMIN_EMAIL=admin@admin.test
   ADMIN_PASSWORD=admin
   BASE_URL=https://<SEU_IP>
   TZ=America/Sao_Paulo
   ```
3. Suba os contêineres do MISP:
   ```bash
   sudo docker-compose pull
   sudo docker-compose up -d
   ```

4. Aguarde alguns minutos para que todos os serviços (Elasticsearch, Cassandra, Cortex, MISP, TheHive) iniciem completamente.

---

## Passo 1: Configuração Inicial do Cortex

O Cortex será responsável por rodar os "Analyzers" e "Responders" (analisadores de inteligência).

1. **Acesso:** Abra o navegador e acesse `http://<SEU_IP>:9001` (ex: `http://192.168.0.107:9001`).
2. **Atualização do Banco:** Na primeira tela, clique no botão **Update Database**.
3. **Criação do Super Admin:** Crie o usuário administrador inicial (ex: admin / senha de sua preferência).
4. **Criar Organização:**
   * Vá em **Organizations** e clique em **+ Add organization**.
   * Nome: `AUTOMATIZAÇÃO` (conforme imagem).
   * Clique em **Save**.
5. **Criar Usuário da Organização:**
   * Entre na organização criada.
   * Vá na aba **Users** e clique em **+ Add user**.
   * Preencha os dados (ex: Login: `automatização`, Nome: `automatização`).
   * Em Roles, selecione `read, analyze`.
   * Clique em **Save user**.
6. **Gerar a API Key (Muito Importante):**
   * Na lista de usuários, na linha do usuário recém-criado, clique em **Create API Key**.
   * Clique no botão azul **Reveal** para exibir a chave.
   * Copie e salve esta chave em um bloco de notas. Ela será usada na integração com o TheHive.

![Cortex Users](imagens/cortex-users.png)

---

## Passo 2: Configuração Inicial do MISP

O MISP será nossa plataforma de compartilhamento de inteligência contra ameaças (Threat Intelligence).

1. **Acesso:** Abra o navegador e acesse `https://<SEU_IP>` (ex: `https://192.168.0.107`). Ignore o aviso de certificado de segurança.
2. **Login Padrão:**
   * Usuário: `admin@admin.test`
   * Senha: `admin`
3. **Troca de Senha:** O sistema exigirá a troca imediata da senha. A nova senha deve ser complexa (letras maiúsculas, minúsculas, números e caracteres especiais).
4. **Criar Organização:**
   * No menu superior, vá em **Administration -> Add Organisations**.
   * Marque a caixa **Local organisation**.
   * Nome (Identifier): `automatização` (conforme imagem).
   * Clique em **Submit**.

![MISP Menu Add Organisation](imagens/misp-menu-add-organisation.png)
![MISP Add Org Form](imagens/misp-add-org-form.png)
5. **Criar Usuário:**
   * Vá em **Administration -> Add User**.
   * Email: `natanael1097@admin.test` (conforme imagem).
   * Organização: Escolha a organização `automatização` que você acabou de criar.
   * Role: `Org Admin`.
   * Defina a senha e clique em **Create user**.

![MISP Users Index](imagens/misp-users-index.png)
6. **Gerar a API Key (Auth Key):**
   * Vá em **Administration -> List Users**.
   * Clique no ID ou email do usuário recém-criado (`natanael1097@admin.test`).
   * Role a página até a seção **Auth keys** e clique em **+ Add authentication key**.
   * Adicione um comentário para identificar e clique em **Submit**.
   * Uma janela aparecerá com a chave (ex: `t7K923sTTyQR4...`). Copie e salve no seu bloco de notas.

![MISP User Profile Auth Key](imagens/misp-user-profile-auth-key.png)
![MISP Generate Auth Key](imagens/misp-generate-auth-key.png)

---

## Passo 3: Configuração do TheHive e Integração Final

O TheHive será o orquestrador principal de incidentes e consumirá os dados do Cortex e do MISP.

1. **Acesso:** Abra o navegador e acesse `http://<SEU_IP>:9000` (ex: `http://192.168.0.107:9000`).
2. **Login Padrão:**
   * Usuário: `admin@thehive.local`
   * Senha: `secret`
3. **Criar Organização:**
   * No menu à esquerda, vá em **Administration -> Organisations** e clique no botão **+ (Add Organisation)**.
   * Nome: `automatização`.
   * Confirme a criação.

![TheHive Add Organization](imagens/thehive-add-organization.png)
4. **Criar Usuário para a Organização:**
   * Vá em **Administration -> Users** e clique no botão **+**.
   * Login: `natanael` / Nome: `natanael`.
   * Organização: Vincule-o à organização `automatização` criada no passo anterior.
   * Perfil (Role): `org-admin`.
   * Confirme e crie uma senha para este usuário.

![TheHive Add User](imagens/thehive-add-user.png)

### Realizando a Integração (Plataform Management)

Ainda logado como Super Admin no TheHive, vamos conectar as plataformas:

1. Acesse o menu **Gestão de plataforma** (Platform Management), localizado no menu lateral esquerdo (ícone de chaves de fenda).

#### A. Integrando o MISP:

1. Vá na aba superior **MISP**.
2. Clique no ícone de **+** na seção "Servidores".
3. Preencha o formulário:
   * Nome do servidor: `misp`
   * Url do servidor: `https://<SEU_IP>/` (ex: `https://192.168.0.107/`)
   * Chave API: Cole a Auth Key gerada no Passo 2 do MISP.
   * Objetivo: Importação e exportação.
   * Definições SSL: Desative a opção "Verificar a autoridade de certificação" (para evitar erros com o certificado autoassinado do Docker).
4. Clique em **Adicione/Atualização**.

#### B. Integrando o Cortex:

1. Vá na aba superior **Cortex**.
2. Clique no ícone de **+** na seção "Servidores".
3. Preencha o formulário:
   * Nome do servidor: `cortex0`
   * Url do servidor: `http://<SEU_IP>:9001/` (ex: `http://192.168.0.107:9001/`)
   * Chave API: Cole a API Key gerada no Passo 1 do Cortex.
   * Definições SSL: Desative a opção de verificação de certificado, se estiver ativa.
4. Clique em **Adicione/Atualização**.

### Validação da Integração

Vá para a aba **Licença** (ou Status) dentro da Gestão de plataforma. Você deverá ver as métricas confirmando a comunicação:

* Servidores MISP: 1 / 1
* Servidores Cortex: 1 / 1

**Sucesso!** O ecossistema está integrado. A partir de agora, quando você criar um caso no TheHive, ele poderá acionar os analisadores do Cortex automaticamente e importar/exportar eventos de Threat Intelligence diretamente do MISP.
