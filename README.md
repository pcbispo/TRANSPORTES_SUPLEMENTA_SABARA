# Transportes Suplementa de Sabará — versão online Netlify

Este pacote transforma a versão HTML em uma aplicação publicada na Netlify com uma API server-side e dados compartilhados em um banco Supabase. **Não publique somente o arquivo HTML por arrastar e soltar**: isso publica a tela, mas não publica a função nem cria o banco compartilhado.

## O que está preparado

- `site/index.html`: sistema de frota com acesso online, área individual do motorista, ponto/intervalo, frota e módulos operacionais.
- `functions/frota-api.mjs`: API Netlify para login, leitura e gravação de dados no banco compartilhado.
- `supabase-schema.sql`: estrutura de banco de dados.
- `netlify.toml`: configuração de publicação e funções.

## 1. Criar o banco Supabase

1. Entre em https://supabase.com/ e crie um projeto.
2. Abra **SQL Editor → New query**.
3. Copie todo o conteúdo de `supabase-schema.sql`, execute e confirme que a tabela `frota_records` foi criada.
4. Em **Project Settings → API**, copie a **Project URL** e a chave **service_role** (se a interface mostrar `secret`, use a chave secreta do servidor equivalente). Essa chave é confidencial: **não a coloque no HTML, não envie em mensagens e não publique no Git**.

## 2. Publicar com funções na Netlify

1. Descompacte este pacote em uma pasta.
2. Coloque a pasta em um repositório privado no GitHub/GitLab e conecte o repositório à Netlify, ou publique com a Netlify CLI. A Netlify precisa receber a pasta que contém `netlify.toml`, `site/` e `functions/`.
3. Na configuração do site Netlify, deixe o diretório de publicação como `site` (já indicado no `netlify.toml`) e as funções em `functions`.
4. Em **Site configuration → Environment variables**, crie estas variáveis:
   - `SUPABASE_URL` = Project URL do seu projeto Supabase.
   - `SUPABASE_SERVICE_ROLE_KEY` = chave secreta de servidor/service role do Supabase.
   - `SESSION_SECRET` = segredo aleatório longo, exclusivo, com pelo menos 32 caracteres.
   - `BOOTSTRAP_ADMIN_PASSWORD` = uma senha inicial forte, exclusiva. Ela será a senha inicial do usuário `admin` quando ainda não houver administrador no banco.
5. Faça um novo deploy para que as funções recebam as variáveis.
6. Abra o endereço HTTPS gerado pela Netlify. Entre com usuário `admin` e a senha definida em `BOOTSTRAP_ADMIN_PASSWORD`.
7. Cadastre os motoristas, operadores e gestores em **Usuários**. Cada pessoa usa o mesmo endereço Netlify, mas sua própria conta.

Para gerar `SESSION_SECRET` em um terminal com Node.js, execute: `node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"`.

## 3. Migração dos dados do HTML anterior

No primeiro login online do administrador, o sistema tenta importar os registros locais deste navegador para o banco compartilhado, sem substituir registros de mesmo identificador já existentes. Faça backup JSON antes de iniciar. Dados existentes em outros celulares/computadores continuam nesses dispositivos até que sejam importados individualmente ou cadastrados no sistema online.

## Importante sobre segurança e operação

- O login e as gravações online passam pela função Netlify; a chave de serviço do banco fica somente no servidor.
- Não reutilize a senha `bispo` nem outra senha padrão conhecida. Configure uma senha forte em `BOOTSTRAP_ADMIN_PASSWORD` antes do primeiro login.
- Se o banco já tiver usuários, a variável de bootstrap não redefine automaticamente as contas existentes.
- As permissões são aplicadas pela API para leitura/gravação de áreas. Faça teste com uma conta de cada perfil antes de uso real.
- O navegador precisa de internet. GPS requer permissão de localização e HTTPS.
- A marcação online grava no banco compartilhado; não feche a página antes de receber a confirmação.
- O sistema ajuda a registrar jornada, intervalos e datas, mas não determina sozinho adicional, compensação ou enquadramento legal. Confirme as regras aplicáveis à atividade e à convenção coletiva com o responsável trabalhista.
- Este pacote não inclui envio de SMS/e-mail, aplicativo nativo nem funcionamento offline com sincronização automática garantida.

## Solução de problemas

- **“Configure SUPABASE_URL…”**: confira as quatro variáveis na Netlify e faça novo deploy.
- **Login inválido**: confirme se é o primeiro administrador, se `BOOTSTRAP_ADMIN_PASSWORD` foi definido antes do primeiro login e se a tabela SQL foi criada.
- **Erro 500 de banco**: confira URL, chave de servidor e tabela `frota_records`.
- **Os dispositivos não veem os mesmos dados**: confirme que todos acessam o mesmo domínio Netlify e que a API não retorna erro no login/gravação.

### Contas de motoristas e operadores

Depois de entrar como administrador:

1. Abra **Motoristas** e cadastre os motoristas com CPF corretamente preenchido.
2. Abra **Usuários → Novo usuário**.
3. Para operadores, escolha o perfil **Operador**. Para motoristas, escolha **Motorista** e vincule o cadastro correto no campo de vínculo.
4. Defina uma senha individual e entregue as credenciais ao usuário por canal privado.

Por padrão, a criação de conta pelo botão “Primeiro acesso do motorista” fica desativada no servidor. Para ativá-la, adicione `ENABLE_DRIVER_SELF_REGISTRATION=true` nas variáveis de ambiente da Netlify e faça novo deploy. Nesse modo, o motorista precisará informar o CPF completo cadastrado. Por segurança, prefira criar as contas pelo painel de administrador.
