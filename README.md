# 🛡️ Face Match 1:1 com Anti‑Spoofing e Detecção de Deepfake

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)

Um sistema de verificação de identidade que não apenas compara duas faces, mas também determina se cada imagem é **autêntica** (pessoa real, ao vivo) ou se trata de uma **falsificação** – foto de rede social, montagem, ataque de apresentação ou deepfake gerado por IA.

> Este repositório é um estudo prático sobre as três camadas de defesa necessárias para um `Face Match` confiável, utilizando apenas ferramentas **open source**.

---

## 📋 Tabela de Conteúdos

- [Visão Geral](#visão-geral)
- [Motivação](#motivação)
- [Arquitetura em Camadas](#arquitetura-em-camadas)
  - [Nível 1: Comparação Robusta de Faces](#nível-1-comparação-robusta-de-faces)
  - [Nível 2: Análise de Vivacidade e Anti‑Spoofing](#nível-2-análise-de-vivacidade-e-anti-spoofing)
  - [Nível 3: Detecção de Deepfakes e Montagens](#nível-3-detecção-de-deepfakes-e-montagens)
- [Tecnologias e Bibliotecas](#tecnologias-e-bibliotecas)
- [Instalação e Configuração](#instalação-e-configuração)
- [Uso Básico](#uso-básico)
- [Exemplos e Resultados](#exemplos-e-resultados)
- [Datasets e Treinamento](#datasets-e-treinamento)
- [Roadmap e Contribuições](#roadmap-e-contribuições)
- [Referências e Leitura Adicional](#referências-e-leitura-adicional)
- [Licença](#licença)

---

## 🔍 Visão Geral

O projeto implementa um **validador de `Face Match 1:1`** que responde a três perguntas fundamentais:

1. **As duas imagens pertencem à mesma pessoa?**
2. **Cada imagem mostra uma pessoa viva e presente, ou é um ataque (foto de foto, tela, vídeo)?**
3. **A imagem em si foi manipulada digitalmente (montagem, face swap, deepfake)?**

A resposta final combina os resultados dessas três análises, oferecendo uma pontuação de confiança e detalhando os motivos da decisão.

---

## 🎯 Motivação

Com o avanço da geração de imagens por IA e a facilidade de se obter fotos de redes sociais, a simples comparação biométrica não é suficiente. Um sistema de verificação de identidade moderno precisa de múltiplos níveis de defesa para ser resistente a fraudes.

Este repositório nasceu para **estudar e demonstrar** como construir esse tipo de sistema usando apenas código aberto, explorando o estado da arte em visão computacional.

---

## 🏰 Arquitetura em Camadas

A defesa é organizada em três níveis, cada um focado em um tipo específico de ameaça.

### Nível 1: Comparação Robusta de Faces
*Compara duas faces com alta precisão, mesmo sob variações de pose, iluminação ou expressão.*

- **Detecção facial:** `YOLOv8` (via `ultralytics`) ou `MTCNN`.
- **Extração de embeddings:** `ArcFace` (loss com margem angular aditiva) como modelo principal, orquestrado pela biblioteca `DeepFace` que também permite ensembles com `FaceNet`, `VGG‑Face`, etc.
- **Métrica de similaridade:** Distância de cosseno entre embeddings normalizados. Um limiar ajustável determina se as faces são da mesma pessoa.

```python
from deepface import DeepFace

result = DeepFace.verify(img1_path, img2_path,
                         model_name='ArcFace',
                         detector_backend='yolov8',
                         distance_metric='cosine')
print(result['verified'], result['distance'])

```
## Nível 2: Análise de Vivacidade e Anti‑Spoofing
Determina se o rosto é de uma pessoa real (viva) ou uma tentativa de burla usando foto impressa, tela, vídeo ou máscara 3D.

Abordagem passiva (sem ação do usuário): Utiliza modelos como Fasnet, AttackNetV2 ou as CNNs do MiniVision's Silent Face Anti‑Spoofing (integradas ao DeepFace).

Detector de objetos especializado: O projeto EasyShield v2.5 demonstra o uso de YOLOv12 nano treinado para classificar rostos em real ou spoof, transformando o problema de vivacidade em uma detecção de duas classes. Aqui usamos uma estratégia similar com YOLOv8 e um dataset customizado.

# Fluxo:

A face detectada pelo YOLO é recortada e passada para o modelo de anti‑spoofing.

O modelo retorna um score de vivacidade (0 a 1). Valores abaixo do limiar indicam ataque.

Nível 3: Detecção de Deepfakes e Montagens
Analisa a imagem como um todo em busca de artefatos de manipulação digital ou geração por IA (redes GAN, diffusion models).

Pacote dedicado: deepface-antispoofing – oferece uma API única para classificar imagens como real, spoof (ataque físico) ou deepfake (ataque digital).

Modelos treinados em datasets sociais: Redes neurais convolucionais (CNNs) treinadas em datasets como So‑Fake‑Set e SocialDF, que contêm imagens adulteradas típicas de redes sociais.


## 🧰 Tecnologias e Bibliotecas
Camada	Ferramentas / Modelos	Descrição
Detecção facial	YOLOv8 (via ultralytics), MTCNN, OpenCV	Localização do rosto na imagem
Comparação	DeepFace, ArcFace, FaceNet, VGG‑Face	Geração de embeddings e cálculo de similaridade
Anti‑Spoofing	Fasnet (DeepFace), AttackNetV2, EasyShield (YOLO‑based)	Detecção de vivacidade passiva
Deepfake / montagens	deepface-antispoofing, CNNs específicas	Detecção de manipulação digital e ataques de IA
Orquestração	Python 3.8+, OpenCV, NumPy	Pipeline completo


Datasets e Treinamento
Para reproduzir ou melhorar os modelos, utilizamos (ou recomendamos) os seguintes datasets públicos:

Anti‑Spoofing: CASIA‑SURF, OULU‑NPU, SiW, UniAttackData (ICCV 2025, com ataques físicos e digitais).

Deepfake: So‑Fake‑Set, SocialDF, Celeb‑DF, FaceForensics++.

Face Match: LFW, CPLFW, AgeDB (usados para benchmark, não para treino).

Os scripts de treinamento estão na pasta training/ (em construção).

🗺️ Roadmap e Contribuições
Pipeline básico de comparação com DeepFace e YOLO

Integração de anti‑spoofing passivo (Fasnet / EasyShield)

Detecção de deepfake via deepface-antispoofing

Suporte a vídeo (análise temporal)

API REST com FastAPI

Dashboard de visualização dos resultados

Fine‑tuning dos modelos com datasets brasileiros (RG, CNH)

Contribuições são muito bem‑vindas! Veja o arquivo CONTRIBUTING.md.

📖 Referências e Leitura Adicional
DeepFace: Face Recognition Library

ArcFace: Additive Angular Margin Loss

EasyShield: Lightweight Anti‑Spoofing with YOLO

Silent Face Anti‑Spoofing (MiniVision)

deepface-antispoofing package

So‑Fake‑Set: A Large Scale Dataset of Social Media Fake Images

📄 Licença
Este projeto está sob a licença MIT – veja o arquivo LICENSE para detalhes.

💡 Este repositório é um estudo acadêmico/pessoal. Para uso em produção, valide os modelos com conjuntos de dados adequados ao seu domínio e siga as regulamentações de proteção de dados (LGPD).
