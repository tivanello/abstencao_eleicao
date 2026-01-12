# Projeto: Análise de Abstenção por Local de Votação / Seção (RO)
(Etapa 1 — Tratamento e preparação dos dados)

## Getting Started

### Pré-requisitos
- Python 3.x
- Git

### Configuração do Ambiente
1. Clone o repositório:
   ```
   git clone https://github.com/tivanello/abstencao_eleicao.git
   cd abstencao_eleicao
   ```

2. Crie e ative o ambiente virtual:
   ```
   py -m venv venv
   .\venv\Scripts\Activate.ps1  # No Windows PowerShell
   ```

3. Instale as dependências:
   ```
   py -m pip install --upgrade pip
   py -m pip install -r requirements.txt  # Se existir
   ```

### Estrutura do Projeto
- `data/`: Contém dados brutos, processados e finais.
  - `raw/`: Dados originais, como `Informações_urnas_abstenção.csv`.
  - `processed/`: Dados após processamento.
  - `final/`: Dados finais prontos para análise.
- `notebooks/`: Notebooks Jupyter para análise e processamento.
- `Comandos básicos – Ambiente Virtual e Git.md`: Comandos básicos para configuração do ambiente e Git.

## 2) Contexto
A abstenção vem aumentando ao longo dos pleitos e não cresce de forma linear. Ela varia bastante entre municípios, zonas, locais de votação e seções. O objetivo do projeto é entender esse comportamento de forma estruturada, identificando padrões persistentes (crônicos) e variações pontuais (choques por pleito).

## 3) Objetivo desta etapa (Tratamento dos dados)
Preparar uma base analítica consistente para medir e comparar a abstenção por:
- Município
- Zona
- Local de votação (e tipo de local)
- Seção
- Pleito (e 2º turno quando houver)

Entregável desta etapa: um conjunto de tabelas limpas e padronizadas, prontas para BI e análises estatísticas/ML interpretável.

## 4) Perguntas que esta etapa precisa permitir responder
- Quais locais/seções têm abstenção acima da média do município/zona?
- Essa abstenção alta é recorrente (muitos pleitos) ou foi pontual (um pleito)?
- A variação está concentrada em poucos locais ou espalhada?
- Existe diferença clara entre área urbana vs distrito vs rural vs comunidades?

(“Motivo” exige dados adicionais e entra na etapa 2/3; aqui a meta é deixar a base preparada para isso.)

## 5) Fontes de dados (primeira etapa)
### 4.1 Log/Resumo de urna por seção (já disponível)
Arquivo: `Informações_urnas_abstenção.csv`

Campos esperados (exemplos do seu arquivo):
- pleito
- municipio
- zona
- secao
- local_votacao (texto/código)
- bairro (do local, quando existir)
- area (urbano/distrito/rural/indígena/comunidade etc.)
- is_segundo_turno
- urna_aptos
- urna_comparecimentos
- urna_abstencoes

### 4.2 Cadastro do eleitor (Espelho do título) — em lote (para etapa 2)
Arquivo modelo: `Espelho titulo.pdf` (exemplo unitário)

Campos úteis (para extração em massa no futuro):
- nascimento (data)
- gênero
- instrução (escolaridade)
- ocupação
- PcD (sim/não; tipo se disponível)
- bairro e CEP do eleitor
- “no município desde” / “na UF desde”
- município/zona/seção/local de votação (no momento do snapshot)

Observação crítica:
- Para análise por seção/pleito, o ideal é um “snapshot do eleitorado por pleito”.
- Se o eleitor muda de seção, o “atual” pode distorcer pleitos antigos.

### 4.3 Eventos ASE (consulta separada) — para etapa 2/3 (comportamento)
- ASE 94: Ausência às urnas (data e situação)
- ASE 167: Justificativa de ausência (data e situação)

Observação crítica:
- Esses eventos têm múltiplas ocorrências por eleitor.
- Sem histórico de domicílio eleitoral (município/zona/seção no pleito), a atribuição por seção fica limitada.

## 6) Regras de padronização (o que será feito no tratamento)
### 5.1 Tipos e formatos
- Datas: converter para padrão ISO (YYYY-MM-DD) quando aplicável
- Números: garantir `urna_aptos`, `urna_comparecimentos`, `urna_abstencoes` como inteiros
- Textos: padronizar (trim, caixa, remover duplicidades simples)

### 5.2 Chaves e integridade
Chave recomendada para “seção no pleito”:
- pleito + municipio + zona + secao
(Se “local_votacao” tiver código, ele entra como apoio, não como chave principal.)

### 5.3 Validações de consistência (qualidade)
Checagens obrigatórias:
- urna_abstencoes >= 0
- urna_comparecimentos >= 0
- urna_aptos > 0
- urna_abstencoes + urna_comparecimentos = urna_aptos
  - se não bater: registrar em relatório de inconsistência e tratar caso a caso

### 5.4 Duplicidades
- Se houver mais de uma linha para a mesma chave (pleito+município+zona+seção):
  - regra: consolidar (somar) apenas se fizer sentido e as linhas forem partes do mesmo registro
  - caso contrário: manter a linha mais confiável e registrar o conflito no log de qualidade

## 7) Métricas e colunas derivadas (features iniciais)
Criar no dataset final:
- taxa_abstencao = urna_abstencoes / urna_aptos
- taxa_comparecimento = urna_comparecimentos / urna_aptos
- tamanho_secao = urna_aptos
- indicador_2o_turno = is_segundo_turno (0/1)

Derivações úteis do texto do local (se aplicável):
- tipo_local (extraído do nome: escola municipal, escola estadual, etc.)
- padronização de local_votacao (evitar “Escola X” escrito de 3 jeitos)

## 8) Entregáveis desta etapa (saída do tratamento)
### 7.1 Dataset principal (fato)
`fato_abstencao_secao_pleito.csv/parquet`
- uma linha por seção por pleito (e por turno quando aplicável)
- com taxas calculadas e campos padronizados

### 7.2 Dimensão de locais (opcional já nesta etapa)
`dim_local_votacao.csv/parquet`
- municipio, zona, local_votacao (código/texto padronizado), bairro, area, tipo_local

### 7.3 Relatório de qualidade
`relatorio_qualidade_dados.md`
- total de linhas
- quantidade de chaves duplicadas
- quantidade de linhas com inconsistência (aptos != comparecimentos+abstenções)
- campos com valores nulos relevantes
- decisões de tratamento aplicadas

## 9) Ferramentas recomendadas
- Python (pandas) para limpeza, validação, padronização e geração das bases finais
- Power BI (ou outro BI) para conferência visual e validação rápida:
  - ranking de abstenção
  - distribuição por área
  - evolução por pleito e turno
- (Opcional) armazenamento em Parquet para performance e reprodutibilidade

## 10) Governança e LGPD (regras do projeto)
- Nesta etapa (dados de urna), não há dados pessoais sensíveis.
- Quando entrar “eleitorado” e ASE:
  - evitar CPF, nome, endereço completo
  - usar ID interno pseudonimizado
  - trabalhar e publicar sempre em formato agregado (por seção/local)
  - controlar acesso às bases brutas

## 11) Próximos passos (depois do tratamento)
Etapa 2 — Enriquecimento e explicação:
- adicionar perfil do eleitorado por seção (idade, escolaridade, PcD, estabilidade)
- opcional: distância estimada eleitor -> local (CEP/bairro + endereço do local)
- adicionar comportamento ASE (94/167) agregando por pleito

Etapa 3 — Modelagem e interpretação:
- modelos interpretáveis (regressão/árvore + SHAP)
- análise multinível (efeitos por município/zona/local)
- clusterização de seções (padrões de abstenção)

