# Guia: ligar o Supabase ao site Reveluxo

Este guia é para quem nunca usou o Supabase. Siga na ordem. Leva cerca de 20 minutos.

## Para que serve

Sem o Supabase, as peças cadastradas ficam salvas **só no navegador de quem cadastrou**. As clientes, em outros celulares, não veem nada novo.

Com o Supabase, o catálogo fica guardado na internet:

- A administradora cadastra as peças em `admin.html` e clica em **Enviar catálogo**.
- Quando uma cliente abre o site (`index.html`), ele baixa o catálogo mais recente sozinho.

---

## Parte 1 — Criar a conta e o projeto

1. Acesse **https://supabase.com** e clique em **Start your project**.
2. Entre com uma conta do GitHub ou com e-mail e senha.
3. Clique em **New project** e preencha:
   - **Name:** `reveluxo`
   - **Database Password:** crie uma senha forte e **guarde-a** (não é a senha do site).
   - **Region:** **South America (São Paulo)**.
4. Clique em **Create new project** e espere 1 a 2 minutos até o painel carregar.

## Parte 2 — Criar a tabela do catálogo

A tabela é onde o catálogo fica guardado. Ela terá uma única linha com todos os dados.

1. No menu da esquerda, clique em **SQL Editor** → **New query**.
2. Copie e cole:

```sql
create table catalogo (
  id int primary key,
  data jsonb,
  updated_at timestamptz
);

alter table catalogo enable row level security;

create policy "leitura publica" on catalogo
  for select using (true);

create policy "escrita anon" on catalogo
  for insert with check (true);

create policy "atualizacao anon" on catalogo
  for update using (true);
```

3. Clique em **Run** (ou Ctrl + Enter). Deve aparecer **Success. No rows returned**.
4. Para conferir: menu **Table Editor** → deve existir a tabela `catalogo`, ainda vazia.

O que cada parte faz:
- `create table` cria a tabela com três colunas: o número da linha, os dados e a data da última atualização.
- `enable row level security` liga a proteção. Sem regras, ninguém acessa.
- As três `policy` liberam: qualquer pessoa pode **ler** (as clientes precisam ver o catálogo) e o site pode **gravar**.

## Parte 3 — Pegar a URL e a chave

1. No menu da esquerda, clique em **Project Settings** (engrenagem) → **API** (ou **Data API**).
2. Copie dois valores:
   - **Project URL** — algo como `https://abcdefgh.supabase.co`
   - **anon public** (em *Project API keys*) — texto longo que começa com `eyJ...`

> **Nunca use a chave `service_role`** no site. Ela dá acesso total ao banco e ficaria visível para qualquer pessoa.

## Parte 4 — Colocar a URL e a chave no site

### Forma A — No código (recomendada)

É a que faz as clientes receberem o catálogo sozinhas.

1. Abra o `index.html` num editor de texto (Bloco de Notas, VS Code ou direto no GitHub, clicando no lápis ✏️).
2. Procure (Ctrl + F) por `SB_CONFIG`. Você vai achar:

```js
const SB_CONFIG = { url: '', key: '', table: 'catalogo' };
```

3. Cole a URL e a chave entre as aspas:

```js
const SB_CONFIG = { url: 'https://abcdefgh.supabase.co', key: 'eyJhbGciOi...', table: 'catalogo' };
```

4. Salve. **Faça a mesma alteração no `admin.html`.**
5. Envie os dois arquivos para o GitHub (se editou no próprio GitHub, clique em **Commit changes**).

### Forma B — Pelo painel (só para testar)

1. Abra `admin.html` e entre com usuário e senha.
2. Vá em **Configurações → Banco de dados (Supabase)**.
3. Cole a **Project URL** e a **Anon key**. Deixe a tabela como `catalogo`.

Isso vale **só para aquele navegador**. As clientes não recebem essa configuração, por isso o site publicado precisa da Forma A.

## Parte 5 — Enviar o catálogo pela primeira vez

1. Abra `admin.html` e entre.
2. Vá em **Configurações → Banco de dados (Supabase)**.
3. Clique em **Enviar catálogo**. Deve aparecer **Catálogo enviado.**
4. No Supabase, abra **Table Editor → catalogo**. Deve existir uma linha com `id = 1`.

## Parte 6 — Testar como cliente

1. Abra o `index.html` publicado em outro celular, ou numa aba anônima.
2. As peças enviadas devem aparecer.
3. Cadastre uma peça nova no admin, clique em **Enviar catálogo** e recarregue o site da cliente. A peça nova deve aparecer.

---

## Rotina do dia a dia

1. Cadastrar ou editar peças em `admin.html`.
2. Clicar em **Configurações → Enviar catálogo**.
3. As clientes veem a atualização na próxima vez que abrirem o site.

Se for cadastrar em **outro aparelho**, clique antes em **Baixar catálogo**. O envio sempre substitui o catálogo inteiro pelo que está naquele aparelho.

Desde a versão atual, salvar uma peça já envia o catálogo ao Supabase automaticamente — não é mais preciso clicar em Enviar catálogo depois de cada cadastro.

## Fotos por link (evita encher o armazenamento)

Fotos por upload ficam em base64 dentro dos dados da peça — poucas peças enchem o armazenamento do navegador e deixam o catálogo pesado no Supabase. Para catálogos grandes, use fotos por link:

1. No repositório do site, no GitHub, entre na pasta `fotos/`.
2. Clique em **Add file → Upload files** e envie a foto. Faça **Commit changes**.
3. Clique na foto enviada → botão **⋯** ou clique direito na imagem → copie a URL "raw" dela (algo como `https://raw.githubusercontent.com/SEU-USUARIO/SEU-REPO/main/fotos/vestido.jpg`). No GitHub Pages, a URL do próprio site também funciona: `https://SEU-USUARIO.github.io/SEU-REPO/fotos/vestido.jpg`.
4. No painel, ao cadastrar a peça, cole esse link no campo **"Colar link da foto"** (abaixo dos botões de upload) e clique em **Adicionar por link**.

Fotos por link não ocupam espaço no armazenamento do navegador nem no Supabase — só o texto do link é salvo. Dá para misturar fotos por upload e por link na mesma peça.

## Problemas comuns

| Mensagem | Causa provável | Solução |
|---|---|---|
| Preencha URL e anon key. | Campos vazios | Refazer a Parte 4. |
| Erro 401 ao enviar | Chave errada | Copiar de novo a chave **anon public**. |
| Erro 404 ao enviar | Tabela não existe ou tem outro nome | Refazer a Parte 2 e conferir o nome `catalogo`. |
| Erro 403 ao enviar | Faltam as regras (policy) | Rodar de novo o SQL da Parte 2. |
| Erro 413 ao enviar | Catálogo grande demais | Usar fotos menores ou remover peças antigas. |
| Nenhum catálogo salvo no Supabase. | Ainda não houve envio | Fazer a Parte 5. |
| Falha de conexão com o Supabase. | Sem internet ou URL errada | Conferir a URL (termina em `.supabase.co`, sem barra no fim). |
| Cliente não vê as peças novas | Faltou enviar, ou falta a Forma A | Clicar em Enviar catálogo e conferir o `SB_CONFIG` no `index.html`. |

## Sobre as fotos

Fotos por upload vão junto com os dados, na mesma linha da tabela — o plano gratuito tem 500 MB, suficiente para um catálogo pequeno/médio. Para catálogos maiores, prefira fotos por link (veja a seção acima): elas não entram no Supabase, só o link de texto.

## Sobre segurança

- A chave **anon** fica visível no código do site. Isso é esperado no Supabase.
- Com as regras deste guia, quem tiver a chave consegue **alterar** o catálogo. Para uma loja pequena o risco é baixo; o ideal a longo prazo é exigir login do Supabase (Supabase Auth) para gravar.
- A senha do painel protege só a tela de administração. Troque-a em **Configurações → Conta**.
