# Reveluxo — Moda Feminina

Catálogo online com vitrine para clientes e painel de administração separado.

## Arquivos

| Arquivo | Função |
|---|---|
| `index.html` | Vitrine para as clientes (endereço principal do site). |
| `admin.html` | Painel de administração, com login. Não aparece no Google (noindex). |
| `support.js` | Runtime usado pelas duas páginas. Não apague. |
| `logo-reveluxo.png` | Logo com fundo transparente. |
| `icon-192.png`, `icon-512.png` | Ícones do app instalado. |
| `manifest.json`, `sw.js` | Permitem instalar o site como app (botão "Instalar app" no rodapé). |
| `fotos/` | Fotos das peças de exemplo. |
| `.nojekyll` | Faz o GitHub Pages servir os arquivos sem processamento. |

## Publicar no GitHub Pages

1. Crie um repositório (ex.: `reveluxo`) e envie **todo o conteúdo desta pasta** para a raiz dele (não a pasta em si).
2. No repositório: **Settings → Pages → Build and deployment → Source: Deploy from a branch**, branch `main`, pasta `/ (root)` → Save.
3. Em alguns minutos o site fica em `https://SEU-USUARIO.github.io/reveluxo/`.
   - Vitrine: `…/reveluxo/`
   - Painel: `…/reveluxo/admin.html` (também pelo link "Administração" no rodapé).

## Acesso ao painel

- Usuário: `admin` · Senha: `capivara2026`
- Troque a senha em **Configurações → Conta** logo no primeiro acesso.

## Importante sobre os dados

- Peças, categorias, configurações e usuários ficam salvos **no navegador de quem cadastra**. As clientes só verão as peças cadastradas em outro aparelho se o catálogo for sincronizado.
- Para sincronizar, crie no Supabase a tabela abaixo e preencha **Configurações → Banco de dados**; use "Enviar catálogo" depois de cadastrar e "Baixar catálogo" nos outros aparelhos:

```sql
create table catalogo (id int primary key, data jsonb, updated_at timestamptz);
alter table catalogo enable row level security;
create policy "leitura publica" on catalogo for select using (true);
create policy "escrita anon" on catalogo for insert with check (true);
create policy "atualizacao anon" on catalogo for update using (true);
```

  A escrita liberada para `anon` permite que qualquer pessoa com a chave altere o catálogo. Para produção, restrinja a escrita (ex.: Supabase Auth).
- As fotos são guardadas junto com os dados. Muitas fotos grandes podem encher o armazenamento do navegador; o painel avisa quando isso acontece.

## Depois de publicar

- Nas meta tags de `index.html`, troque `logo-reveluxo.png` em `og:image` e `twitter:image` pela URL completa (ex.: `https://SEU-USUARIO.github.io/reveluxo/logo-reveluxo.png`) para o card aparecer ao compartilhar o link no WhatsApp.
- As peças de exemplo podem ser removidas em **Painel → Remover exemplos**.
