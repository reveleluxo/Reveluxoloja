# Reveluxo — Moda Feminina · pacote para gerar o APK

Catálogo de moda feminina com área de administração. Este pacote contém tudo o que o app precisa: a aplicação, a logo e as imagens de demonstração.

## Conteúdo

| Arquivo | O que é |
|---|---|
| `Catálogo.dc.html` | A aplicação completa (vitrine + administração). Abre direto no navegador. |
| `support.js` | Runtime que a aplicação carrega. **Deve ficar na mesma pasta do HTML.** |
| `logo-reveluxo.png` | Logo da loja — topo, rodapé, ícone do app e cabeçalho do catálogo em PDF. |
| `fotos/` | Imagens ilustrativas das peças de demonstração. |

## Identidade

- Nome: **Reveluxo — Moda Feminina**
- Fundo: `#000000` · Dourado: `#c9a44c` / `#e3c684` · Texto: `#eae5da`
- Tipografia: Cormorant Garamond (títulos) + Jost (interface)
- WhatsApp: **(85) 98583-6214** · Instagram: **@reveluxo**
- Local: Fortaleza, Brazil 60325820

## Funcionalidades

**Vitrine (cliente)**
- Todas as peças na home, mais recentes primeiro; menu superior filtra por categoria.
- Selo **Novo** por 14 dias a partir do cadastro.
- Anúncio com galeria (setas e miniaturas), "Ver maior" em tela cheia, seleção de tamanho e cor (clicar de novo desmarca).
- **Comprar** abre o WhatsApp da loja com peça, preço, tamanho e cor na mensagem.
- **Compartilhar** usa o compartilhamento nativo do celular.

**Administração** (link discreto no rodapé, protegido por login)
- Usuário `admin` · senha `capivara2026` (alteráveis em Dados da loja).
- Cadastro de peça: foto de capa, nome, categoria, preço, tamanhos (P, M, G, GG, XG), descrição.
- Por cor: nome, quantidade em estoque e várias fotos, reordenáveis arrastando (a primeira é a capa).
- Envio de fotos por **Galeria** (vários arquivos) ou **Câmera** (foto tirada na hora) — ver requisito abaixo.
- Criação e exclusão de categorias.
- **Exportar catálogo em PDF** — ver requisito abaixo.
- Campos de Supabase (URL, anon key, tabela) para sincronizar o catálogo entre dispositivos.

## ⚠ Requisito 1 — Câmera e galeria no cadastro de fotos

Cada cor tem dois botões, e a foto de capa também: **Galeria** (`<input type="file" accept="image/*" multiple>`) e **Câmera** (`<input type="file" accept="image/*" capture="environment">`). As duas formas precisam funcionar no APK:

- Implementar `WebChromeClient.onShowFileChooser` tratando os três casos: seleção de arquivos, **seleção múltipla** (`MODE_OPEN_MULTIPLE`) e **captura pela câmera** quando o `accept` é `image/*` — abrindo um chooser com `ACTION_IMAGE_CAPTURE` + `ACTION_GET_CONTENT`.
- Criar arquivo temporário via `FileProvider` para receber a foto da câmera e devolver a Uri ao WebView; nunca retornar `null` no callback (isso trava o input para sempre).
- Pedir em tempo de execução as permissões `CAMERA` e `READ_MEDIA_IMAGES` (ou `READ_EXTERNAL_STORAGE` abaixo da API 33).
- Declarar `<uses-feature android:name="android.hardware.camera" android:required="false" />` e o `provider` do FileProvider no manifest.
- A foto tirada deve ir para o mesmo fluxo de upload do HTML (o app não precisa tratar nada além de entregar a Uri).

## ⚠ Requisito 2 — Exportação do catálogo (NÃO ALTERAR)

O botão **Exportar catálogo em PDF** já está implementado em `Catálogo.dc.html` (métodos `exportPdf` / `buildPages`). O PDF **deve sair exatamente como sai no teste do projeto**:

- Um **único arquivo** `catalogo-reveluxo.pdf`, gerado dentro do próprio app — **sem** impressão do navegador/WebView e **sem** bibliotecas externas.
- Páginas **A4 em 300 dpi** (canvas de **2480 × 3508 px**), **fundo preto `#000000`**.
- Cabeçalho: logo centralizada em faixa própria e, abaixo, a linha `(85) 98583-6214 • @reveluxo`, sem sobreposição, seguida de um fio horizontal.
- **6 peças por página** (2 colunas × 3 linhas).
- Cada peça: **foto de capa inteira, ajustada dentro do quadro (contain), sem corte, sem distorção e sem esticar**; nome em branco, preço em dourado `#e3c684`, tamanhos em cinza claro e selo **NOVO** quando aplicável.
- Rodapé: `Reveluxo · Moda Feminina — página X de Y`.
- Imagens embutidas em JPEG qualidade 0.95, preservando a nitidez das fotos originais.

**Regra para o app Android:** não substituir, não reimplementar e não "melhorar" esse gerador. O WebView só precisa salvar o download que ele produz. Geração nativa de PDF ou `window.print()` produz resultado diferente do teste e não deve ser usada.

## Como gerar o APK no Google AI Studio

1. Envie este ZIP no AI Studio.
2. Use o prompt abaixo.
3. Baixe o projeto Android gerado e compile (Android Studio → Build → Build Bundle(s)/APK(s) → Build APK(s)).

### Prompt

> Crie um aplicativo Android nativo (Kotlin) que exibe o catálogo de moda feminina **Reveluxo** contido neste pacote.
>
> **Estrutura**
> - Copie `Catálogo.dc.html`, `support.js`, `logo-reveluxo.png` e a pasta `fotos/` para `app/src/main/assets/`, mantendo a mesma hierarquia (o HTML carrega `support.js` e as imagens por caminho relativo).
> - A Activity principal abre um WebView em tela cheia apontando para `file:///android_asset/Catálogo.dc.html`.
>
> **Configuração do WebView**
> - `javaScriptEnabled = true`, `domStorageEnabled = true`, `allowFileAccess = true`, `allowFileAccessFromFileURLs = true`, `allowUniversalAccessFromFileURLs = true`, `databaseEnabled = true`.
> - Cache habilitado; o app deve funcionar totalmente offline.
> - Sem barra de endereço, sem controles de zoom, `WebViewClient` próprio.
>
> **Câmera e galeria (requisito crítico)**
> - Implemente `WebChromeClient.onShowFileChooser` cobrindo: arquivo único, **múltiplos arquivos** (`MODE_OPEN_MULTIPLE`) e **captura pela câmera** — monte um `Intent.createChooser` com `ACTION_IMAGE_CAPTURE` e `ACTION_GET_CONTENT`/`ACTION_OPEN_DOCUMENT` quando o `accept` for `image/*`.
> - Use `FileProvider` (com `provider` e `file_paths.xml` declarados) para o arquivo temporário da câmera e devolva a Uri ao callback. Jamais chame o callback com `null`, nem deixe de chamá-lo ao cancelar (use `onReceiveValue(null)` apenas no cancelamento).
> - Solicite em runtime `CAMERA` e `READ_MEDIA_IMAGES` (ou `READ_EXTERNAL_STORAGE` < API 33). Declare `<uses-feature android:name="android.hardware.camera" android:required="false" />`.
>
> **Exportação do catálogo (requisito crítico)**
> - O PDF é gerado pelo próprio HTML (canvas + Blob + link de download). Não implemente geração de PDF nativa nem use `createPrintDocumentAdapter()` ou `window.print()`.
> - Implemente `DownloadListener` tratando **URLs `blob:`** e **data URLs**: leia o conteúdo (via `XMLHttpRequest`/`FileReader` injetado por JS, se necessário) e salve em `Downloads` com o nome informado (`catalogo-reveluxo.pdf`), pedindo a permissão conforme a API level e notificando a conclusão.
> - Implemente `WebChromeClient.onCreateWindow` para não bloquear o download aberto em nova janela.
> - Após salvar, ofereça um Intent `ACTION_SEND` (`application/pdf`) para enviar o catálogo pelo WhatsApp.
>
> **Comportamento**
> - Botão voltar do Android navega no histórico do WebView (`canGoBack()`); só sai do app quando não houver histórico.
> - Links externos (`wa.me`, `api.whatsapp.com`, `instagram.com`, `mailto:`, `tel:`) abrem no app correspondente ou no navegador via Intent, nunca dentro do WebView.
>
> **Identidade visual**
> - Nome do app: **Reveluxo**.
> - Ícone adaptativo e splash screen com `logo-reveluxo.png` centralizada sobre fundo preto `#000000`.
> - Barra de status e navegação preta (`#000000`) com ícones claros; tema escuro fixo.
> - Orientação livre (retrato e paisagem).
>
> **Permissões no manifest**
> - `INTERNET`, `CAMERA`, `READ_MEDIA_IMAGES` e escrita em Downloads conforme a API level.
> - `minSdk 24`, `targetSdk` mais recente estável.
>
> Entregue o projeto Android completo, compilável no Android Studio, pronto para gerar o APK.

### Observações

- As fotos cadastradas ficam no armazenamento local do aparelho. Para o catálogo aparecer igual em vários celulares, preencha os campos do Supabase e use "Enviar catálogo" / "Baixar catálogo".
- Fotos com pelo menos 1200 px de largura aproveitam melhor os 300 dpi do PDF.
- Troque a senha do admin antes de distribuir o app.
