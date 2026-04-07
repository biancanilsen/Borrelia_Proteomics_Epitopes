# 🧬 Análise Proteômica e de Epítopos: *Borrelia burgdorferi*

Este repositório contém o pipeline de bioinformática e os resultados da análise proteômica da bactéria *Borrelia burgdorferi* (agente etiológico da Doença de Lyme), desenvolvida para a disciplina de Bioinformática.

## 📌 Objetivo do Projeto
O objetivo principal foi identificar potenciais candidatos a vacinas e diagnósticos, focando em **proteínas de membrana** e mapeamento de **epítopos de células B** linearmente expostos.

## 🛠️ Ferramentas Utilizadas
A análise foi executada utilizando virtualização via Docker:

* **FastProtein:** Pipeline automatizado para caracterização físico-química e predição de localização subcelular.
* **EpiBuilder:** Orquestrador de predição de imunogenicidade (BepiPred 3.0, Vaxign, IEDB).
* **Docker Desktop:** Para execução de containers em arquitetura `linux/amd64` (via Rosetta 2).
* **UniProt:** Fonte dos dados proteômicos (Proteoma ID: `UP000001807`).


## 🚀 Fluxo de Trabalho

### 1. Obtenção dos Dados
O proteoma foi baixado diretamente da UniProt via terminal:
```bash
curl -s "https://rest.uniprot.org/uniprotkb/stream?format=fasta&query=proteome:UP000001807" -o input.fasta
```

### 2. Caracterização Proteômica
Execução do **FastProtein** para identificar proteínas de membrana:
```bash
docker run --rm --platform linux/amd64 -v "$(pwd)":/data bioinfoufsc/fastprotein:clean-latest fastprotein -i /data/input.fasta -o /data/results_fastprotein
```

### 3. Filtro de Candidatos
Seleção das 50 principais proteínas com evidência de membrana para análise de epítopos:
```bash
awk '/^>/{c++} c<=50' results_fastprotein/raw/membranes.fasta > top50.fasta
```

### 4. Predição de Epítopos
Uso do **EpiBuilder** para mapeamento imunogênico:
```bash
docker run -it --rm --platform linux/amd64 -v "$(pwd)":/data/ -v /var/run/docker.sock:/var/run/docker.sock -e EPIBUILDER_VOLUME=epibuilder-data bioinfoufsc/epibuilder-core epibuilder --input_file /data/top50.fasta --loc gram_neg --output /data/results_epibuilder
```

## 📊 Principais Resultados

* **Proteínas totais analisadas:** 1291
* **Proteínas com evidência de membrana:** 258
* **Proteínas com epítopos preditos (Top 50):** 28
* **Candidato Principal:** Proteína **P66 (H7C7N8)**, apresentando epítopos em alças extracelulares (Topologia `oooo`), ideais para reconhecimento por anticorpos.
