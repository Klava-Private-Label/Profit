# Klava Private Label — versão Supabase + GitHub

Sistema de controle das operações de Private Label da Klava no exterior, com login e senha.

- **Banco de dados, login e arquivos:** Supabase
- **Site:** GitHub Pages (um único `index.html`)
- **Acesso:** a pessoa clica em "Solicitar acesso", o pedido chega por e-mail para **tadeu@klavabrands.com**, e você aprova no menu **Usuários**.

## O que tem nesta pasta

| Arquivo | Para que serve |
|---|---|
| `index.html` | O sistema (mesmas telas da versão Apps Script, mais login e Usuários) |
| `config.js` | Endereço e chave pública do seu projeto Supabase (você preenche) |
| `supabase/01_estrutura.sql` | Cria tabelas, regras, segurança, aprovação de usuários e o espaço de arquivos |
| `supabase/functions/solicitar-acesso/index.ts` | Envia o e-mail de "nova solicitação de acesso" |
| `tools/` | Arquivos usados para gerar o `index.html` a partir do Index.html do Apps Script |

`supabase/02_dados.sql` traz todos os dados da planilha em 04/11/2026: 10 clientes, 12 fornecedores, 20 projetos, 28 pedidos, 31 custos, 91 lançamentos (todos com banco MERCURY) e 84 documentos.

---

## Passo a passo

### 1. Crie o projeto no Supabase
1. Acesse https://supabase.com, entre e clique em **New project**.
2. Dê o nome `klava-private-label`, escolha a região **South America (São Paulo)** e crie uma senha forte para o banco. Guarde essa senha.
3. Quando o projeto estiver pronto, abra **SQL Editor → New query**.
4. Cole todo o conteúdo de `supabase/01_estrutura.sql` e clique em **Run**. O resultado esperado é "Success. No rows returned".

### 2. Configure o login
Vá em **Authentication → Sign In / Providers → Email** e deixe assim:
- **Enable Email provider:** ligado.
- **Confirm email:** **desligado**. Ninguém entra sem a sua aprovação, então a confirmação por e-mail não é necessária.
- **Allow new users to sign up:** ligado.

### 3. Conta de e-mail (Resend)
1. Crie uma conta em https://resend.com **usando o e-mail tadeu@klavabrands.com**. Sem domínio próprio configurado, o Resend só entrega para o dono da conta, e é exatamente para ele que o aviso vai.
2. Abra **API Keys → Create API Key** e copie a chave (começa com `re_`).

### 4. Função de e-mail (Edge Function)
1. No Supabase, abra **Edge Functions → Deploy a new function → Via Editor**.
2. Use o nome `solicitar-acesso`, apague o exemplo, cole o conteúdo de `supabase/functions/solicitar-acesso/index.ts` e clique em **Deploy**.
3. Em **Edge Functions → Secrets**, crie:
   - `RESEND_API_KEY`: a chave do passo 3.
   - `APP_URL`: o endereço do site do passo 5 (dá para preencher depois).

### 5. Publique no GitHub
1. No GitHub, crie o repositório `klava-private-label`.
2. Clique em **Add file → Upload files**, arraste **todo o conteúdo desta pasta** e confirme com **Commit changes**.
3. Abra `config.js` no GitHub (ícone de lápis) e preencha os três campos:
   - `supabaseUrl` e `supabaseAnonKey`: estão em Supabase → **Project Settings → API**. Use a chave **anon / public**, nunca a `service_role`.
   - `appUrl`: `https://SEU-USUARIO.github.io/klava-private-label/`.
4. Vá em **Settings → Pages**, escolha **Deploy from a branch**, branch `main`, pasta `/ (root)`, e salve. Em 1 a 2 minutos o site fica no ar.

O GitHub Pages em repositório **privado** só funciona em planos pagos (Pro/Team). Na conta gratuita, o repositório precisa ser **público**. Isso expõe só o código. Os dados continuam protegidos no Supabase, e ninguém vê nada sem login aprovado.

### 6. Endereço do site no Supabase
Em **Authentication → URL Configuration**:
- **Site URL:** o endereço do GitHub Pages.
- **Redirect URLs:** adicione o mesmo endereço.

### 7. Seu primeiro acesso (administrador)
Abra o site, clique em **Solicitar acesso** e cadastre-se com **tadeu@klavabrands.com**. Esse e-mail entra direto como **administrador aprovado**, sem precisar de aprovação.

### 8. Carregue os dados
No **SQL Editor**, cole o conteúdo de `supabase/02_dados.sql` e clique em **Run**. Se der erro, nada é gravado, porque o arquivo roda em bloco único.

### 9. Equipe
Cada pessoa abre o site e clica em **Solicitar acesso**. Você recebe o e-mail e aprova em **Usuários**, no menu lateral. Lá também dá para recusar, bloquear e tornar alguém administrador.

---

## Documentos
- **Documentos antigos:** continuam no Google Drive. O link abre para quem tem acesso à pasta "Klava Private Label/Documentos".
- **Documentos novos:** vão para o Supabase Storage, numa área privada. Abrem por um link temporário, só para usuários aprovados.

## Opcional: "Esqueci minha senha" para toda a equipe
O envio de e-mail padrão do Supabase só entrega para membros da equipe do projeto. Para o link de nova senha chegar a qualquer pessoa:
1. No Resend, verifique o domínio **klavabrands.com** em **Domains → Add domain**. A TI precisa criar os registros DNS indicados.
2. No Supabase, vá em **Authentication → Emails → SMTP Settings** e preencha:
   - Host: `smtp.resend.com`
   - Porta: `465`
   - Usuário: `resend`
   - Senha: a chave do Resend
   - Remetente: `sistema@klavabrands.com`
3. Com o domínio verificado, crie também o secret `EMAIL_FROM` = `Klava Private Label <sistema@klavabrands.com>`.

## Atualizar o sistema no futuro
As telas são as mesmas do `Index.html` do Apps Script. Depois de alterar esse arquivo, rode o comando abaixo e suba o novo `index.html` no GitHub:

```
python3 tools/montar_index.py caminho/do/Index.html
```
