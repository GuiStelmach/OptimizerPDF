# OptimizerPDF

Aplicação web offline para organizar imagens, dividir PDFs e reduzir o tamanho de documentos PDF sem depender de serviços externos.

Teste online: [abrir no GitHub Pages](https://guistelmach.github.io/OptimizerPDF/)

Versão atual: 1.9.0  
Autor: Guilherme Medeiros Stelmach (Gui Stelmach)

---

## Visão geral

Este projeto nasceu de uma dor real dentro da empresa em que trabalhei: clientes enviavam diversos prints e comprovantes em imagens, e a equipe precisava organizar tudo em um único PDF para atender a um limite de tamanho muito rígido — em geral 2 MB. Antes, a tarefa era feita manualmente no Microsoft Word: cada imagem era posicionada em uma folha, o arquivo era salvo e, quando excedia o limite, o processo era repetido, criando uma rotina lenta e pouco confiável.

Em muitos casos, quando o arquivo ultrapassava o limite, a solução era abrir o PDF em um navegador, simular a impressão e salvar novamente apenas parte das páginas, repetindo o processo várias vezes para dividir em 2, 3 ou mais partes. Isso demandava tempo, atenção e retrabalho manual, além de aumentar o risco de erro humano.

O OptimizerPDF foi criado para resolver esse problema de forma prática, rápida e segura. Ele organiza automaticamente as imagens em páginas, gera o PDF com o layout adequado e ainda permite dividir PDFs já existentes em partes menores com rapidez. Em vez de gastar mais de 30 minutos em uma tarefa repetitiva, o processo passou a ser concluído em menos de 1 minuto em muitos casos.

Além disso, a decisão de desenvolver toda a solução em um único arquivo HTML com JavaScript puro foi estratégica: o ambiente da empresa era altamente controlado e exigia segurança, isolamento e ausência de dependências externas. A ferramenta foi pensada para funcionar de forma local, sem envio de dados para servidores e sem uso de frameworks, reforçando o princípio de ser "blindada" em termos de infraestrutura e risco operacional.

Hoje, o projeto representa uma solução real para um problema de produtividade e automação de processos internos, demonstrando uma abordagem orientada a resolver desafios concretos com tecnologia simples, eficiente e segura.

O app roda 100% no navegador e trabalha com três fluxos principais:

1. Criar um novo PDF a partir de imagens;
2. Dividir um PDF existente em partes dentro de um limite;
3. Otimizar um PDF existente para atingir uma meta de tamanho.

O projeto é entregue em um único arquivo HTML com JavaScript e CSS puros, sem dependências externas nem envio de dados a servidores.

## Funcionalidades

- Criação de PDF a partir de imagens em qualquer ordem.
- Reordenação visual por arrastar, mover para cima/baixo e remover itens individualmente.
- Pré-visualização fixa das páginas com destaque cruzado entre a lista e a página selecionada.
- Geração interna de PDF sem usar `window.print()` ou impressora virtual.
- Ajuste automático de qualidade e resolução para respeitar um limite em MB.
- Confirmação explícita antes de baixar um PDF acima do limite.
- Divisão de PDF existente em partes com limite por parte e máximo de anexos.
- Preservação do original quando o arquivo já cabe no limite.
- Otimização de PDFs existentes com comparação antes/depois e meta de tamanho.
- Piso de leitura para não degradar excessivamente arquivos que precisam permanecer legíveis.
- Bloqueio de operações com arquivos incompatíveis e mensagens claras sobre o motivo.

## Como usar

### 1) Criar PDF a partir de imagens

1. Abra `index.html` no navegador ou use a versão publicada no GitHub Pages.
2. Clique em `Criar novo PDF`.
3. Selecione imagens ou arraste-as para a página.
4. Organize a ordem e confirme a pré-visualização.
5. Ajuste o `Limite (MB)` e clique em `Gerar PDF`.
6. O navegador solicitará o download do arquivo final.

Formatos aceitos em imagens: JPG, PNG, WebP, GIF, BMP e AVIF.

Nome padrão do arquivo de saída: `PDFGerado__AAAA-MM-DD.pdf`.

### 2) Dividir um PDF existente

1. Clique em `Dividir PDF existente`.
2. Selecione o arquivo PDF.
3. Ajuste o `Limite por parte (MB)` e o `Máx. de anexos`.
4. O app analisa se o arquivo já cabe ou se precisa ser dividido.
5. Se a divisão for liberada, o download em lote pode ser solicitado.

Arquivos protegidos por senha, com anotações, formulários ou recursos que não podem ser preservados podem ser bloqueados com explicação.

### 3) Otimizar um PDF existente

1. Clique em `Otimizar PDF`.
2. Selecione o arquivo PDF.
3. Informe a meta de tamanho desejada em MB.
4. O app analisa as imagens, reduz resolução e qualidade quando necessário e mostra a comparação antes/depois.
5. Se a meta for atingida, o botão de download fica disponível.

O arquivo original nunca é alterado; o resultado sai como um novo PDF com sufixo `_otimizado`.

## Regras de paginação e layout

Quando o app cria um PDF a partir de imagens, a paginação usa um layout em A4, em retrato, com margens e espaçamentos fixos.

- Ordem preservada da lista.
- Imagens verticais consecutivas podem ficar lado a lado quando houver espaço.
- Imagens horizontais ocupam uma linha inteira.
- Máximo de duas linhas por página.
- Espaços por página: 1, 2 ou 4 imagens.
- Ajuste proporcional e centralizado da imagem dentro da célula.

## Tamanho do PDF e otimização

O limite padrão de 2 MB é usado para cenários de anexos com restrição de arquivo. O fluxo de geração é:

1. cada imagem é redimensionada para a área impresa;
2. o PDF é montado e seu tamanho real é medido;
3. se o resultado exceder o limite, o app tenta combinações mais leves até encontrar uma versão aceitável.

Na criação de PDF a partir de imagens, as tentativas de compressão seguem a lógica de resolução e qualidade, em ordem decrescente de peso:

- 200 DPI / 88%
- 200 DPI / 80%
- 160 DPI / 80%
- 160 DPI / 72%
- 130 DPI / 72%
- 110 DPI / 65%

Se mesmo o menor nível não atender ao limite, o app pede confirmação antes de baixar o arquivo acima do teto.

Para otimização de PDF existente, a ferramenta reduz primeiro a resolução e depois a qualidade, apenas até o ponto necessário, respeitando o piso de leitura de 110 DPI e qualidade 65%.

## Arquitetura e funcionamento

O projeto é um único arquivo `index.html` com HTML, CSS e JavaScript puro, sem bibliotecas externas.

```
Interface
  ├─ seleção de imagens ou PDF
  ├─ organização e pré-visualização
  ├─ planejamento de paginação/partes
  └─ geração ou otimização do arquivo

      ├─ leitura local dos arquivos
      ├─ inspeção/validação do PDF
      ├─ redimensionamento e compressão
      ├─ montagem do documento final
      └─ download do arquivo no navegador
```

Principais recursos técnicos:

- PDF gerado internamente com composição manual de páginas e streams de imagem.
- leitura local de imagens com suporte a `createImageBitmap` e `canvas`.
- geração de arquivos em `Blob` e download direto via navegador.
- validação de integridade e regras de divisão para evitar resultados inválidos.
- processamento local e sem rede.

## Segurança e privacidade

- o app não envia arquivos para nenhum servidor;
- não usa `localStorage`, cookies ou persistência de sessão para manter arquivos;
- o processamento acontece localmente no navegador;
- a política de segurança do HTML bloqueia conexões de rede e recursos externos;
- o código permanece legível e sem dependências de terceiros.

Para verificar uma distribuição, é possível registrar o hash do arquivo final com:

```bash
sha256sum index.html
```

## Compatibilidade

Recomendado usar um navegador moderno baseado em Chromium, como Google Chrome ou Microsoft Edge.

O app depende de recursos do navegador como `createImageBitmap`, `canvas.toBlob`, `Blob` e download por arquivo local. Em ambientes com políticas rígidas de segurança, o download pode ser bloqueado; nessas situações, é necessário validar o comportamento do navegador antes de distribuir a ferramenta.

## Limitações conhecidas

- PDF gerado a partir de imagens é composto por imagens JPEG e não mantém texto selecionável.
- A página é fixada em A4, em retrato.
- O layout da criação de PDF limita o máximo de duas linhas por página.
- Formatos que o navegador não consegue decodificar, como HEIC em alguns navegadores, podem ser rejeitados.
- PDFs protegidos por senha, com formulários, anotações ou revisões incrementais podem ser bloqueados pela operação de divisão ou otimização.
- PDFs extremamente grandes ou com muitas páginas aumentam o uso de memória e o tempo de processamento.

## Estrutura do projeto

```text
.
├── index.html   # aplicação completa (HTML, CSS e JavaScript)
├── README.md    # documentação do projeto
└── LICENSE      # licença MIT
```

## Histórico de versões

| Versão | Mudanças principais |
|---|---|
| 1.9.0 | Novo módulo de otimização de PDF, meta de tamanho, piso de leitura, comparação antes/depois, avisos padronizados e melhor experiência visual. |
| 1.8.x | Ajustes de acessibilidade, responsividade e downloads. |
| 1.7.0 | Novo layout com cabeçalho, menu, título da página e cartões. |
| 1.6.0 | Divisão de PDF grande em partes sem recompressão, com limite de anexos. |
| 1.5.0 | Janela de progresso na geração do PDF, com barra, tentativas e cancelamento. |
| 1.4.0 | Confirmação antes de salvar PDF acima do limite. |
| 1.3.1 | Correção para evitar cópias de imagens ao arrastar itens já anexados. |
| 1.3.0 | Janela de ajuda e orientações para uso. |
| 1.2.0 | Revisão visual da área de trabalho e pré-visualização. |
| 1.1.0 | Ordem preservada, geração interna do PDF, ajuste automático ao limite e miniaturas leves. |
| Inicial | Organização de imagens e geração de PDF pela impressão do navegador. |

## Licença

Este projeto é distribuído sob a licença [MIT](LICENSE).

## Observações finais

O OptimizerPDF foi construído para ser simples, local e útil em trabalho diário. O objetivo principal é permitir que o usuário resolva a maior parte dos casos sem sair do navegador, preservando o controle do arquivo final e evitando dependência de serviços externos.
