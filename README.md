# Diagnóstico e Limpeza de Qualidade de Dados

[![Abrir no Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/SEU_USUARIO/diagnostico-qualidade-dados/blob/main/02_diagnostico_limpeza.ipynb)
[![Ver relatório](https://img.shields.io/badge/ver%20relat%C3%B3rio-exemplo-2563EB?style=flat-square)](https://SEU_USUARIO.github.io/diagnostico-qualidade-dados/)

**[▶ Executar o notebook no Colab](https://colab.research.google.com/github/mikelevianna-eng/diagnostico-qualidade-dados/blob/main/02_diagnostico_limpeza.ipynb)** · **[📊 Ver o relatório gerado](https://mikelevianna-eng.github.io/diagnostico-qualidade-dados/)**

Dois notebooks que recebem qualquer arquivo CSV, apontam os problemas que distorcem indicadores, aplicam as correções possíveis e entregam a base tratada junto com um relatório do que foi feito.

Rodam inteiramente no Google Colab, sem instalação local.

> ### ⚠️ Todos os dados são fictícios
>
> A base de exemplo é gerada por script. Nomes, CPFs, e-mails, telefones, cidades e valores são sorteados e **não correspondem a nenhuma pessoa ou empresa real**. Os CPFs têm onze dígitos aleatórios e não passam na validação do dígito verificador, ou seja, são inválidos por construção.
>
> A escolha é deliberada: cadastro real de cliente é informação pessoal protegida pela LGPD e não pode ser publicado. O que se demonstra aqui é a **metodologia**, que é real e se aplica a qualquer base.

---

## O problema

Pequenas empresas acumulam anos de cadastro preenchido à mão, por pessoas diferentes, em sistemas que mudaram no meio do caminho. Quando alguém finalmente tenta extrair um indicador, o número não fecha.

A cidade aparece como Porto Alegre, porto alegre, P. Alegre e Poa, e o relatório mostra quatro praças onde existe uma. O mesmo cliente está cadastrado três vezes com identificadores diferentes, inflando a contagem da carteira e dividindo o histórico de compras. O campo de valor mistura vírgula e ponto, e a soma sai errada sem que nenhum erro apareça na tela.

Nenhum desses defeitos trava o sistema. Todos comprometem a decisão.

---

## O que os notebooks entregam

Na base de exemplo, com 636 linhas e 13 colunas, o diagnóstico encontrou **21 problemas** e aplicou **15 correções**, entregando 600 linhas limpas e nota **53 de 100**.

**Saídas:**

- `dados_tratados.csv` com a base corrigida e os tipos certos
- `relatorio_qualidade.html` com o que foi encontrado e o que foi feito

---

## O que é detectado

| Problema | O que causa se não for tratado |
|---|---|
| Campos em branco | Médias e contagens calculadas sobre um universo menor que o real |
| Espaços invisíveis | O mesmo valor tratado como dois valores distintos |
| Número guardado como texto | `1.234,56` lido como 1,23, um erro de mil vezes sem aviso |
| Datas em formatos diferentes | `03/05` interpretado como maio ou março conforme o padrão |
| Categoria escrita de várias formas | Contagem agrupada sai fragmentada em categorias inexistentes |
| Cadastros duplicados | Carteira inflada e histórico de compras dividido entre registros |
| Documento com dígitos incorretos | Impossibilidade de validar ou cruzar com outras bases |
| E-mail inválido | Base de contato menor do que o cadastro sugere |
| Valores negativos onde não cabem | Somas e médias contaminadas por erro de lançamento |

---

## O que é corrigido e o que não é

O tratamento padroniza espaços, converte números e datas respeitando o padrão brasileiro, unifica categorias equivalentes pela grafia mais frequente, reduz documentos a dígitos, remove e-mails inválidos, padroniza campos de sim e não, e elimina duplicatas.

**Valores apenas parecidos ficam sinalizados e intactos.** Unificar por semelhança juntaria corretamente Sta. Maria e Santa Maria, mas o mesmo critério juntaria Santo Ângelo e Santa Ângela. Correção que adivinha é pior do que problema declarado, então a decisão fica com quem conhece a operação.

Essa distinção entre o que a máquina resolve e o que exige julgamento humano é o centro do projeto.

---

## Decisões técnicas que valem explicar

**A conversão de número não apaga pontos cegamente.** Havendo ponto e vírgula juntos, o ponto é separador de milhar. Havendo apenas ponto, ele é decimal. Sem essa verificação, `89.90` viraria 8990.

**A conversão de data testa formatos explícitos** em vez de deixar a biblioteca adivinhar. A inferência automática lê `03/05/2025` como 5 de março pelo padrão americano, e o erro segue direto para a análise mensal sem nenhuma mensagem.

**A nota considera o pior problema de cada coluna, não a soma.** Os mesmos registros costumam ser atingidos por vários defeitos ao mesmo tempo, e somar contaria a mesma linha repetidas vezes. A primeira versão fazia isso e devolvia zero para qualquer base minimamente suja, o que é inútil como diagnóstico.

**Identificadores não viram número.** Código de cliente é rótulo, não quantidade, e converter transformava `1577` em `1577.0`.

**Nada é alterado em silêncio.** Cada correção fica registrada com a coluna afetada e a quantidade de registros, o que permite reconciliar a base tratada com o sistema de origem. É a primeira pergunta que o cliente faz.

---

## Como executar

Abra os notebooks no Google Colab pelos links abaixo e rode as células na ordem.

1. `01_gerar_base.ipynb` cria o arquivo de exemplo com os problemas
2. `02_diagnostico_limpeza.ipynb` diagnostica, limpa e gera as saídas

Para usar com seus próprios dados, pule o primeiro notebook e altere a linha `ARQUIVO` no segundo.

Único requisito é pandas, já disponível no Colab.

---

## Uso com dados reais

Ao apontar o notebook para uma base verdadeira, o processamento acontece na sessão do Colab e nada é enviado para outro lugar. Ainda assim, valem dois cuidados.

Nunca publique em repositório público a base tratada nem o relatório gerado a partir de dados reais, porque o relatório traz exemplos de valores encontrados e isso pode expor informação pessoal.

Se o objetivo for mostrar o resultado a terceiros, gere o relatório sobre uma amostra anonimizada ou remova os exemplos antes de compartilhar.

---

## Limitações

O diagnóstico aponta indícios, não conclusões. Um campo em branco pode ser erro de cadastro ou informação que legitimamente não se aplica àquele registro, e só quem opera o sistema sabe distinguir.

A detecção de valores equivalentes compara grafia, e não significado. Abreviações e apelidos internos não são reconhecidos automaticamente.

A validação de documentos verifica a quantidade de dígitos, e não o dígito verificador.

---

## Estrutura

```
diagnostico-qualidade-dados/
├── 01_gerar_base.ipynb          Gera a base de exemplo com problemas
├── 02_diagnostico_limpeza.ipynb Diagnostica, limpa e gera as saídas
├── docs/
│   └── index.html               Relatório publicado, acessível pelo link acima
├── exemplos/
│   └── dados_tratados.csv       Base corrigida
└── README.md
```
