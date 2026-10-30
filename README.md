# OptimizerPDF

Aplicação web de arquivo único para **organizar imagens e gerar um PDF otimizado**, ajustado automaticamente a um limite de tamanho. Funciona 100% offline, sem bibliotecas e sem enviar nenhum dado a servidores.

**Versão atual:** 1.5.0  
**Autor original:** Guilherme Medeiros Stelmach (Gui Stelmach)

---

## Sumário

- [Contexto e objetivo](#contexto-e-objetivo)
- [Funcionalidades](#funcionalidades)
- [Como usar](#como-usar)
- [Regras de paginação e layout](#regras-de-paginação-e-layout)
- [Tamanho do PDF](#tamanho-do-pdf)
- [Arquitetura e funcionamento](#arquitetura-e-funcionamento)
- [Configuração](#configuração)
- [Segurança e privacidade](#segurança-e-privacidade)
- [Compatibilidade](#compatibilidade)
- [Limitações conhecidas](#limitações-conhecidas)
- [Solução de problemas](#solução-de-problemas)
- [Estrutura do projeto](#estrutura-do-projeto)
- [Histórico de versões](#histórico-de-versões)
- [Licença](#licença)

---

## Contexto e objetivo

Muitos fluxos de trabalho exigem consolidar capturas de tela e comprovantes em um único PDF: anexar evidências a um chamado, preencher um formulário, enviar documentos por e-mail. Os arquivos chegam soltos, em formatos e tamanhos variados, e o destino costuma impor um **limite de tamanho** baixo para o anexo. Fazer isso à mão (colar imagem por imagem, exportar, conferir o tamanho e recomeçar) é lento e propenso a erro.

O OptimizerPDF automatiza essa etapa:

1. recebe as imagens do usuário;
2. permite organizar a ordem e visualizar as páginas;
3. gera o PDF final já ajustado ao limite de tamanho.

## Funcionalidades

- **Seleção flexível:** escolha de arquivos ou arraste e solte em qualquer lugar da página.
- **Organização visual:** reordenação por arrastar e soltar ou pelos botões ↑ ↓, remoção individual e ampliação da imagem. Arrastar uma imagem já anexada só a reordena: novas imagens entram apenas a partir de arquivos externos.
- **Ordem preservada:** a ordem da lista é exatamente a ordem do PDF.
- **Pré-visualização das páginas:** mostra como as imagens ficarão distribuídas, com destaque cruzado entre a lista e as páginas.
- **PDF gerado internamente:** o arquivo é baixado direto, sem janela de impressão e sem depender de impressora virtual.
- **Ajuste automático ao limite:** o limite padrão é de 2 MB (configurável). O app reduz resolução e qualidade em etapas até o PDF caber.
- **Tamanho exato:** o medidor mostra o tamanho real do PDF gerado em relação ao limite.
- **Janela de progresso:** durante a geração, uma janela bloqueia a tela e mostra a barra de andamento, a tentativa em curso (resolução e qualidade), o resultado de cada tentativa, o tempo decorrido e o botão **Cancelar**.
- **Confirmação ao exceder o limite:** se o PDF passar do limite mesmo na menor qualidade, nada é baixado automaticamente. O app mostra o tamanho original, o tamanho do PDF, o limite e o excedente, e pergunta se o usuário quer salvar assim mesmo.
- **Nome padrão de arquivo:** `Documento_AAAA-MM-DD.pdf`, com a data local do dia.
- **Janela de ajuda:** orientações diretas sobre o uso, abertas ao carregar a página e pelo botão **Como usar**.
- **Tratamento de duplicatas e erros:** arquivos repetidos são ignorados e imagens ilegíveis são removidas com aviso.

## Destaques técnicos

- **Gerador de PDF próprio**, escrito do zero (PDF 1.4), que embute JPEGs diretamente (`DCTDecode`), sem nenhuma dependência.
- **Controle de tamanho por medição real**: o app testa combinações decrescentes de resolução e qualidade e mede o PDF pronto, em vez de estimar.
- **Paginação como função pura**, testada com 20.000 listas aleatórias (ordem preservada, limites de linhas e de imagens, imagens dentro da área útil).
- **Interface orientada a estado**: a tela é reconstruída a partir de uma única fonte de verdade, o que evita inconsistências.
- **Segurança por desenho**: política de segurança de conteúdo que bloqueia qualquer acesso à rede, nenhum armazenamento local e nomes de arquivo sempre tratados como texto.
- **Experiência cuidada**: pré-visualização fiel das páginas, janela de progresso com cancelamento, confirmação antes de salvar um PDF acima do limite, acessibilidade por teclado.

## Como usar

1. Abra o arquivo `index.html` em um navegador.
2. **Selecione** as imagens ou arraste-as para a página. São aceitos JPG, PNG, WebP, GIF, BMP e AVIF.
3. **Organize** a ordem dos cards. Clique em uma imagem para ampliá-la e em **×** para removê-la.
4. Escolha os **espaços por página** (1, 2 ou 4) e confira a pré-visualização.
5. Ajuste o **limite (MB)**, se necessário.
6. Clique em **Gerar PDF**. O arquivo é baixado automaticamente.

> As imagens existem apenas na memória do navegador. Ao fechar ou recarregar a página, elas são descartadas, então gere o PDF antes de sair.

## Regras de paginação e layout

A página é A4 (210 × 297 mm), com margens de 12 mm nas laterais e 14 mm em cima e embaixo. O espaçamento entre imagens é de 3 mm na horizontal e 5 mm na vertical.

| Regra | Comportamento |
|---|---|
| Ordem | Sempre a ordem da lista. Nenhuma imagem é puxada de outra posição. |
| Imagens verticais | Duas verticais **consecutivas** ficam lado a lado, quando houver espaço. |
| Imagens horizontais | Ocupam uma linha inteira. |
| Linhas por página | No máximo 2. |
| Espaços por página | Número máximo de imagens por página (1, 2 ou 4). Quatro imagens só ocorrem com dois pares de verticais. |
| Ampliação | Imagens pequenas não são ampliadas além do tamanho natural (referência de 96 dpi). |
| Posicionamento | Cada imagem é ajustada à célula, mantendo a proporção e centralizada. |

## Tamanho do PDF

O limite padrão de **2 MB** é pensado para anexos com restrição de tamanho. O fluxo de geração é:

1. Cada imagem é redimensionada para o tamanho em que será impressa, limitado à sua resolução original, e convertida em JPEG sobre fundo branco.
2. O PDF é montado e seu tamanho é medido.
3. Se passar do limite, o app tenta a próxima combinação, mais leve, de resolução e qualidade.

As tentativas, em ordem:

| Tentativa | Resolução | Qualidade JPEG |
|---|---|---|
| 1 | 200 DPI | 88% |
| 2 | 200 DPI | 80% |
| 3 | 160 DPI | 80% |
| 4 | 160 DPI | 72% |
| 5 | 130 DPI | 72% |
| 6 | 110 DPI | 65% |

Pontos de atenção:

- O valor **"originais"** exibido na tela é a soma dos arquivos de origem e **não** é o tamanho do PDF. O tamanho real aparece no medidor depois de gerar.
- Se mesmo a última tentativa passar do limite, o PDF **não é baixado automaticamente**: o app exibe o tamanho original, o tamanho do PDF, o limite e o excedente, e pergunta se o usuário deseja salvar assim mesmo. Para reduzir, escolha **Não salvar**, remova imagens ou aumente o limite.
- Prints com texto muito pequeno podem perder nitidez nas tentativas de menor qualidade. Confira o PDF antes de enviar.

## Arquitetura e funcionamento

O projeto é um único arquivo `index.html` com HTML, CSS e JavaScript puro.

```
Seleção de arquivos
        │
        ▼
Fila de leitura (3 simultâneas)
 ├─ decodifica a imagem (createImageBitmap, respeitando a orientação EXIF)
 ├─ obtém largura e altura
 └─ gera a miniatura (≈320 px)
        │
        ▼
Paginação (ordem preservada) ──► Layout em milímetros
        │                               │
        ▼                               ▼
Lista de cards + pré-visualização   Geração do PDF
                                     ├─ redimensiona cada imagem (canvas) e codifica em JPEG
                                     ├─ monta o PDF (gerador interno)
                                     └─ tenta combinações mais leves até caber no limite
                                             │
                                             ▼
                                     Download do arquivo (Blob)
```

**Gerador de PDF interno.** Em vez de usar `window.print()` ou bibliotecas, o app escreve o PDF diretamente (PDF 1.4): catálogo, páginas, imagens JPEG incorporadas (`/DCTDecode`), fluxos de conteúdo e tabela de referências cruzadas. Isso dá ao app controle total do resultado e permite informar o tamanho exato do arquivo.

**Principais APIs do navegador:** Canvas, `createImageBitmap`, `Blob`, `URL.createObjectURL`, `TextEncoder`, Drag and Drop.

## Configuração

Os parâmetros ficam no início do bloco `<script>` do `index.html`.

| Parâmetro | Local | Descrição |
|---|---|---|
| `BRAND` / `VERSION` | constantes | Nome exibido no cabeçalho e versão exibida no rodapé. |
| `HELP_AUTO_OPEN` | constante | `true` abre a janela de ajuda ao carregar. `false` abre só pelo botão **Como usar**. |
| `PW`, `PH`, `MX`, `MY`, `GX`, `GY` | constantes | Dimensões da página e margens e espaçamentos, em mm. |
| `tries` | função `generate` | Combinações de resolução e qualidade tentadas na geração do PDF. |
| Limite padrão | campo `limit` no HTML | Valor inicial do limite de tamanho, em MB. |

## Segurança e privacidade

- **Sem rede:** o arquivo não faz requisições e não carrega recursos externos. Uma política de segurança de conteúdo (CSP) no próprio HTML bloqueia conexões de rede (`connect-src 'none'`), e só permite imagens locais (`blob:` e `data:`).
- **Sem persistência:** nada é salvo em `localStorage`, cookies ou disco. Ao fechar a página, as imagens e os endereços temporários são liberados da memória.
- **Processamento local:** toda leitura, compressão e montagem do PDF acontecem no navegador do usuário.
- **Auditabilidade:** o código é legível, sem minificação e sem dependências de terceiros. Para controle de distribuição, recomenda-se registrar o hash do arquivo aprovado:

  ```bash
  sha256sum index.html
  ```

## Compatibilidade

Recomenda-se um navegador moderno baseado em Chromium (Google Chrome ou Microsoft Edge) atualizado. O app depende de `createImageBitmap` com opções de redimensionamento, de `canvas.toBlob` com saída JPEG e do download via `Blob`.

> Valide o funcionamento no navegador e na versão em que o app será usado antes de distribuir, incluindo políticas que possam bloquear downloads ou o uso de canvas.

## Limitações conhecidas

- O PDF é composto somente por imagens JPEG: não há texto selecionável e a transparência é convertida em fundo branco.
- Formato de página fixo em A4, em retrato.
- Máximo de 2 linhas por página.
- Formatos que o navegador não decodifica (por exemplo, HEIC em alguns navegadores) não são lidos e a imagem é removida com aviso.
- Não existe plano alternativo de geração via impressão do navegador: se uma política bloquear o download, o PDF não é gerado.
- Imagens muito grandes ou em grande quantidade aumentam o uso de memória durante a geração.

## Solução de problemas

| Sintoma | Causa provável | O que fazer |
|---|---|---|
| Arquivo ignorado ao adicionar | Formato não suportado | Converta para JPG ou PNG. |
| "Não foi possível ler" uma imagem | Arquivo corrompido ou formato não decodificado pelo navegador | Reexporte a imagem ou converta para JPG ou PNG. |
| Geração demorada | Muitas imagens grandes, com várias tentativas | A janela de progresso mostra o andamento. Aguarde ou use **Cancelar**. |
| PDF acima do limite | Muitas imagens ou imagens complexas | O app pede confirmação antes de salvar. Escolha **Não salvar** e remova imagens ou aumente o limite (MB). |
| Texto do print pouco nítido | Qualidade reduzida para caber no limite | Aumente o limite ou reduza a quantidade de imagens. |
| O download não inicia | Política do navegador bloqueando downloads | Verifique as configurações do navegador ou consulte as políticas da sua organização. |
| Páginas com poucas imagens | Regra de pares de verticais consecutivas | Reordene para colocar verticais lado a lado. |

## Estrutura do projeto

```
.
├── index.html                      # aplicação completa (HTML, CSS e JavaScript)
├── README.md                       # este documento
└── LICENSE                         # licença MIT
```

## Histórico de versões

| Versão | Mudanças |
|---|---|
| 1.5.0 | Janela de progresso na geração do PDF (barra, tentativas, tempo decorrido e cancelamento), com bloqueio da tela durante o processamento. |
| 1.4.0 | Confirmação antes de salvar: se o PDF gerado exceder o limite, o app mostra tamanho original, tamanho do PDF e excedente, e só baixa se o usuário confirmar. |
| 1.3.1 | Correção: soltar uma imagem já anexada fora de um card não a adiciona mais como nova imagem (cópia). |
| 1.3.0 | Janela de ajuda (pop-up) com orientações de uso, tamanho do PDF e nome do arquivo. |
| 1.2.0 | Revisão visual: área de trabalho em duas colunas, pré-visualização fixa, medidor de tamanho, destaque cruzado entre lista e páginas e divisórias de página. |
| 1.1.0 | Ordem preservada na paginação, geração do PDF pelo próprio app com tamanho exato, ajuste automático ao limite, miniaturas leves, fila de leitura e política de segurança de conteúdo. |
| Inicial | Organização das imagens e geração do PDF pela impressão do navegador. |

## Licença

Este projeto é distribuído sob a licença [MIT](LICENSE).